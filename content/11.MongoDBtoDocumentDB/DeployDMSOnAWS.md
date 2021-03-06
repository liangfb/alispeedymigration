﻿---
title: "使用 AWS DMS 进行数据迁移"
chapter: false
weight: 112
---

## 部署 使用 AWS DMS 进行数据迁移

#### 创建复制实例

1.下载CloudFormation脚本到本地电脑。
{{%attachments title="下载链接:" /%}}

2.进入印度region的Cloudformation界面：https://ap-south-1.console.aws.amazon.com/cloudformation/home?region=ap-south-1

3.点击【创建堆栈】按钮，选择"Upload a template file"，并把第一步下载的CloudFormation脚本上传，点击【Next】按钮。

4.在指定堆栈的界面上，按照如下内容输入以后，点击【Next】按钮。

* Stack name：aliworkshop-dms-<你的姓名拼音>

* DMSInstanceName：bcdrdms-<你的姓名拼音>

* DMSSubnetGroupName：dmssubnet-<你的姓名拼音>

* BastionInstanceTag：BastionInstance-<你的姓名拼音>

* EC2InstancePrivateIP：把把缺省值中的第二个0换成对应你的编号（10.x.0.5）

* KeyPairName：选择一个已经存在的key pair，如果没有则创建一个新的并下载即可。

* VpcCidr：把缺省值中的第二个0换成对应你的编号（10.x.0.0/16）

* SubnetCidr1：把缺省值中的第二个0换成对应你的编号（10.x.0.0/24）

* SubnetCidr2：把缺省值中的第二个0换成对应你的编号（10.x.32.0/24）

* SubnetCidr3：把缺省值中的第二个0换成对应你的编号（10.x.64.0/24）

* SubnetCidr4：把缺省值中的第二个0换成对应你的编号（10.x.96.0/24）

5.点击【Next】，并点击【Create Stack】按钮，从而开始创建DMS复制实例。

#### 获取阿里云上的MongoDB的相关信息

```bash
MONGO=`aliyun dds DescribeDBInstances | jq .DBInstances | jq .DBInstance | jq .[0] | jq .DBInstanceId | sed 's/\"//g'`
aliyun dds ModifySecurityIps --DBInstanceId $MONGO --SecurityIps <DMS实例的公网ip地址>
aliyun dds AllocatePublicNetworkAddress --DBInstanceId $MONGO
PUBHRI=`aliyun dds DescribeShardingNetworkAddress \
--DBInstanceId $MONGO \
| jq .NetworkAddresses | jq .NetworkAddress | jq .[2] | jq .NetworkAddress | sed 's/\"//g'
echo $PUBHRI
```

#### 创建终端节点

按照以下步骤创建源终端节点和目标终端节点：

1. 打开印度 Region 的 DMS 控制台：[https://ap-south-1.console.aws.amazon.com/dms/v2/home?region=ap-south-1](https://ap-south-1.console.aws.amazon.com/dms/v2/home?region=ap-south-1)

2. 创建源终端节点
    
    * 在左边菜单上选择【终端节点】菜单，然后在右边的界面上选择【创建终端节点】按钮
        ![](/images/MongoDB2DocDB/CreateEndpoint.png)
    
    * 在创建终端节点的界面上：
        
        * “终端节点类型”选择：源终端节点
        
        * “终端节点标识符”输入：`ali-endpoint`
        
        * “源引擎”选择：mongodb
        
        * “服务器名称”输入: 上一步所获得的阿里云MongoDB的外网连接地址里的域名
        
        * “端口”输入：`3717`
        
        * “身份验证模式”选择：password
        
        * “用户名”输入：`root`
        
        * “密码”输入：`Initial-1`
                
        * “数据库名称”输入：`admin`

        * 其他保留缺省值
        ![](/images/MongoDB2DocDB/SourceEndpoint-1.png)
        ![](/images/MongoDB2DocDB/SourceEndpoint-2.png)
        
        * 点击【创建终端节点】

3. 创建目标终端节点
    
    * 在左边菜单上选择【终端节点】菜单，然后在右边的界面上选择【创建终端节点】按钮
        ![](/images/MongoDB2DocDB/CreateEndpoint2.png)
    
    * 在创建终端节点的界面上：
        
        * “终端节点类型”选择：目标终端节点
        
        * “终端节点标识符”输入：`aws-endpoint`
        
        * “源引擎”选择：docdb
        
        * “服务器名称”输入DocumentDB连接信息里的域名
        ![](/images/MongoDB2DocDB/DocdbCname.png)
        
        * “端口”输入：`27017`
        
        * “安全套接字层 (SSL) 模式”选择：verify-full
        
        * 点击 “添加新的CA证书”，把DocumentDB连接信息里获取到的pem证书下载地址通过浏览器或者wget命令下载到本地，然后点击【Choose File】上传，“证书标识符”输入：`LabCertificate`，点击【导入证书】
            
        ![](/images/MongoDB2DocDB/DocdbCAURL.png)                        
        ![](/images/MongoDB2DocDB/DocdbCAUpload.png)
        
        * “用户名”输入：`root`
        
        * “密码”输入：`Initial-1`
        
        * “数据库名称”输入：`admin`

        * 其他保留缺省值
        ![](/images/MongoDB2DocDB/DestEndpoint.png)

        * 点击【创建终端节点】

#### 测试终端节点连接

按照以下步骤测试终端节点连接：

1. 打开印度Region的DMS控制台：[https://ap-south-1.console.aws.amazon.com/dms/v2/home?region=ap-south-1#endpointList](https://ap-south-1.console.aws.amazon.com/dms/v2/home?region=ap-south-1#endpointList)

2. 选择ali-endpoint，在【操作】下拉框里选择【测试连接】
    ![](/images/MongoDB2DocDB/AliEndpoint.png)
    
3. 在界面上选择【运行测试】按钮，看到状态为successful以后，点击【返回】按钮
    ![](/images/MongoDB2DocDB/AliEndpointTest.png)
    
4. 选择ali-endpoint，在【操作】下拉框里选择【测试连接】
    ![](/images/MongoDB2DocDB/AWSEndpoint.png)
    
5. 在界面上选择【运行测试】按钮，看到状态为successful以后，点击【返回】按钮
    ![](/images/MongoDB2DocDB/AWSEndpointTest.png)

#### 创建迁移任务

按照以下步骤创建迁移任务：

1. 打开DMS控制台： [https://ap-south-1.console.aws.amazon.com/dms/v2/home?region=ap-south-1#tasks](https://ap-south-1.console.aws.amazon.com/dms/v2/home?region=ap-south-1#tasks)

2. 在左边菜单上选择【数据库迁移任务】菜单，然后在右边的界面上选择【创建任务】按钮
![](/images/MongoDB2DocDB/CreateMigrationJob.png)

3. 在创建数据库迁移任务界面上：
    
    * “任务标识符”输入：`lab-migration`
    
    * “复制实例”选择：bcdrdms-<你的姓名拼音>- vpc-xxxxxxx
    
    * “源数据库终端节点”选择：ali-endpoint
    
    * “目标数据库终端节点”选择：aws-endpoint
    
    * “迁移类型”选择：迁移现有数据并复制正在进行的更改
    
    * “表映像”选择：引导式UI，点击【添加新选择规则】，“架构”选择：输入架构
    
    * 其他保留缺省值
    ![](/images/MongoDB2DocDB/CreateMigrationJobInfo-1.png)
    ![](/images/MongoDB2DocDB/CreateMigrationJobInfo-2.png)
        
    * 点击【创建任务】按钮

4. 等迁移任务进度变成“100%”后，进行下一步的实验
![](/images/MongoDB2DocDB/MigrationDone.png)
