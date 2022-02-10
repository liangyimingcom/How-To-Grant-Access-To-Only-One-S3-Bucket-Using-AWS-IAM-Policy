# 如何生成S3存储桶的专属访问权限给不同的IAM用户，打造专属企业共享网盘

How To Grant Access To Only One S3 Bucket Using AWS IAM Policy



## 一、业务需求：

1）企业不同的部门，使用不同的S3 Bucket（自己专属）；
2）不同的部门只能访问自己专属的S3 Bucket，且有完整的读写权限；
3）不允许访问其他部门S3 Bucket，看一眼也不行；
4）配置需支持 S3 Browser, s3 cyberduck等，三方S3客户端工具；
5）后期运维要有简单的运维办法，每次添加部门或者S3 bucket时候，不想配置一堆代码才能实现，最好一步操作完成；



## 二、解决方案：

1）采用IAM User方式，来管理全部的S3 Bucket 访问权限；
2）新增IAM User后，需要同步增加相同名称的S3 Bucket；
3）通过添加IAM User到IAM Group，实现Policy赋值 - 从而完成对应权限管理；
4）Policy赋值只给了S3资源的使用权限，没有其他资源访问能力；
5）以上流程可以提供简便有效的IT运维方式；



## 三、方案特点：

1）S3高数据安全，没有任何数据在公网publish，数据传输通过加密进行；
2）S3业务高灵活度，快速实现多个部门的文件共享；
3）S3数据为三副本保存。对无法避免的员工误删除操作，可以通过开启S3 Version可以保留无限个版本实现CDP备份与恢复；
4）低成本：S3自动分层和生命周期实现低成本；



## 四、IT运维操作指南：

### 举例业务需求：计划对以下五个部门分别开启S3 Bucket（存储桶），完成数据共享与数据安全传输，获得稳定的传输速度； 五个部门名称分别为：

- **平面设计小组01共享目录  —> graphicdesign-team01-s3share**  

- 平面设计小组02共享目录 —> graphicdesign-team02-s3share

- 3D设计小组共享目录 —> dddesign-team02-s3share

- 财务部门共享目录 —>  financedepartment-team02-s3share
- 采购小组共享目录 —> purchasingdepartment-team02-s3share

<u>注意：请务必使用英文小写以符合S3bucket的创建规范</u>



### 1. 创建IAM User

1）打开https://console.amazonaws.cn/iamv2/home#/users后点击创建新用户，选择“Access key - Programmatic access”确保iam user只能通过AK/SK访问，如下图：

![image-20220210170150443](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20220210170150443.png)

2）添加这个新用户到 IAM Group “s3bucket-accessonly-byawsusername-iamgroups”里，如下图：

![image-20220210154652515](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20220210154652515.png)

如果没有找到IAM Group，请参考章节《后台工作原理与IAM User/Policy的配置方法》完成基本配置工作。



3）创建用户成功后，请单独记录下你的AK/SK（很重要）。关闭界面后SK将无法获取只能重新创建；

![image-20220210160725587](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20220210160725587.png)



### 2. 创建S3 Bucket

打开S3后，创建新的S3 Bucket，名字与 IAM User 完全一致，如“GraphicDesign-Team01-S3share”；

使用缺省配置创建成功；

![image-20220210165821995](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20220210165821995.png)





### 3. 配置S3 Browser,三方S3客户端

S3 Browser 相对于s3命令行是一个很方便的操作S3的图形化界面工具，下载网址：http://s3browser.com/ ，配置步骤如下：

1）新建用户

![image-20220210171612202](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20220210171612202.png)



2）填入相同的用户名“graphicdesign-team01-s3share”，配置注意选择 AccountType: Amazon S3 in China ；

![image-20220210170929788](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20220210170929788.png)



3）填入IAM USER部分的AK/SK信息后，点击完成；

![image-20220210171332464](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20220210171332464.png)



4）测试能否正常工作后完成。

![image-20220210171709360](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20220210171709360.png)



### 4. 创建完成

（平面设计小组01共享目录  —> graphicdesign-team01-s3share ）创建完成，你可以相同步骤完成其他目录的创建工作。





## 五、后台工作原理与IAM User/Policy的配置方法：



### 1. 工作原理：允许所需的 Amazon S3 权限

首先，您需要指定访问 Amazon S3 所需的权限 - ListAllMyBuckets 和 GetBucketLocation。 如果未指定这两个权限，则用户在每次尝试访问存储桶内的任何对象时都将面临“拒绝访问”错误。

```json
{
  "Sid": "AllowUserToSeeBucketListInTheConsole",
  "Action": ["s3:GetBucketLocation", "s3:ListAllMyBuckets"],
  "Effect": "Allow",
  "Resource": ["arn:aws:s3:::*"]
}
```



其次，我们的用户应该只能访问一个与IAM User 用户名相同的S3存储桶（如“graphicdesign-team01-s3share”），存储桶中的所有其他文件夹都应该可见。 

为了能够在 S3 存储桶中导航对象，我们需要指定额外的权限：

```json
{
  "Sid": "AllowAllS3ActionsInUserFolder",
  "Effect": "Allow",
  "Action": [
    "s3:*"
  ],
  "Resource": [
    "arn:aws-cn:s3:::${aws:username}",
    "arn:aws-cn:s3:::${aws:username}/*"
  ]
},
```

现在我们的用户可以查看存储桶根目录下的文件和文件夹。 我们授予了他们访问文件夹中所有对象以及将来可能创建的任何子文件夹的权限。



### 2. IAM Policies 完整的配置代码

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowAllS3ActionsInUserFolder",
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws-cn:s3:::${aws:username}",
                "arn:aws-cn:s3:::${aws:username}/*"
            ]
        },
        {
            "Sid": "AllowUserToSeeBucketListInTheConsole",
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets",
                "s3:GetBucketLocation"
            ],
            "Resource": "*"
        },
        {
            "Sid": "NOTAllowDeleteBucketInTheConsole",
            "Effect": "Deny",
            "Action": "s3:DeleteBucket",
            "Resource": "*"
        }
    ]
}
```

![image-20220210173941055](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20220210173941055.png)



### 3. IAM Policies 完整的配置

![image-20220210174347071](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20220210174347071.png)

![image-20220210174420560](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20220210174420560.png)



### 六、总结：

随着时间发展，各类网盘的使用限制越来越多。为了降低影子IT的影响，大多数企业都对常见网盘进行了网络限速，甚至是IP封锁。 

使用AWS S3+S3客户端打造企业专属共享网盘，在使用效率、数据安全、成本方面都具备各方面的优势。

本文的教程协助您打通最复杂的权限配置部分，更方便的使用AWS S3打造企业专属共享网盘。



## 七、进阶（备份、连续数据保护与存储成本降低）：

### 1. 在AWS云端可以保留多少个备份副本？

[![image-20210511105709858](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210511105709858.png)](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210511105709858.png)

### 2. 在AWS云端如何降低存储成本？

[![image-20210511105727578](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210511105727578.png)](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210511105727578.png)



## 引用：

1） AWS S3 上传下载工具 - 配置与使用指南(**适用于MAC版本的S3 cyberduck配置指南**]) https://github.com/liangyimingcom/AWS-S3-upload-and-download-tool-configuration-and-usage-guide_available-for-AWS-China/blob/readme-edits/README.md

2）世界上最简单的S3命令行说明(文件备份与恢复) The simplest S3 command line guide in the world for Files Backup/Restore https://github.com/liangyimingcom/The-simplest-S3-command-line-guide-in-the-world-for-Files-Backup-Restore




