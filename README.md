# AWS_CLI
## Task 1
<img src="https://github.com/osonoi/AWS_CLI/blob/main/fig1.png">

ターミナルに"envsubst" パッケージをインストール。シェルフォーマットの文字列をテキストの環境変数で置き換えるもの
```
sudo yum install gettext -y
```
インストールされている場合
```
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
Package gettext-0.19.8.1-3.amzn2.x86_64 already installed and latest version
Nothing to do
```
AWS アカウント番号を環境変数として設定します。
```
export awsAccount=`aws sts get-caller-identity --query "Account" --output text` && echo awsAccount=$awsAccount >> ~/.bashrc
```
インスタンスメタデータをクエリし、Amazon EC2 インスタンスの AWS リージョンを環境変数として設定します。
```
export awsRegion=`curl -s http://169.254.169.254/latest/meta-data/placement/region` && echo awsRegion=$awsRegion >> ~/.bashrc
```
仮想プライベートクラウド (VPC) ID を環境変数として設定します。
```
export VPC=`aws ec2 describe-vpcs --filters Name=tag:Name,Values=wa-lab-vpc --query 'Vpcs[*].VpcId' --output text --region $awsRegion` && echo VPC=$VPC >> ~/.bashrc
```
AWS リージョンの最初のアベイラビリティーゾーンのIDを環境変数を設定します。
```
export awsAZ1=`aws ec2 describe-availability-zones --region $awsRegion --query 'AvailabilityZones[].ZoneName[]|[0]' --output text` && echo awsAZ1=$awsAZ1 >> ~/.bashrc
```
AWS リージョンの 2 つ目のアベイラビリティーゾーンの環境変数を設定します。
```
export awsAZ2=`aws ec2 describe-availability-zones --region $awsRegion --query 'AvailabilityZones[].ZoneName[]|[1]' --output text` && echo awsAZ2=$awsAZ2 >> ~/.bashrc
```
2 つ目のアベイラビリティーゾーンにパブリックサブネットを作成します。
```
aws ec2 create-subnet --vpc-id $VPC --cidr-block "10.100.2.0/24" --availability-zone $awsAZ2 --tag-specifications 'ResourceType=subnet, Tags=[{Key=Name,Value=wa-public-subnet-2}]' --region $awsRegion
```
出力
```
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "use1-az6",
        "Tags": [
            {
                "Value": "wa-public-subnet-2",
                "Key": "Name"
            }
        ],
        "AvailableIpAddressCount": 251,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:us-east-1:238827011620:subnet/subnet-0f191ca93f95b8082",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0a296354b51520c5d",
        "State": "available",
        "AvailabilityZone": "us-east-1b",
        "SubnetId": "subnet-0f191ca93f95b8082",
        "OwnerId": "238827011620",
        "CidrBlock": "10.100.2.0/24",
        "AssignIpv6AddressOnCreation": false
    }
}
```
2 つ目のアベイラビリティーゾーンにプライベートサブネットを作成します。
```
aws ec2 create-subnet --vpc-id $VPC --cidr-block "10.100.3.0/24" --availability-zone $awsAZ2 --tag-specifications 'ResourceType=subnet, Tags=[{Key=Name,Value=wa-private-subnet-2}]' --region $awsRegion
```
出力
```
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "use1-az6",
        "Tags": [
            {
                "Value": "wa-private-subnet-2",
                "Key": "Name"
            }
        ],
        "AvailableIpAddressCount": 251,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:us-east-1:238827011620:subnet/subnet-05dced5e4b32261dd",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0a296354b51520c5d",
        "State": "available",
        "AvailabilityZone": "us-east-1b",
        "SubnetId": "subnet-05dced5e4b32261dd",
        "OwnerId": "238827011620",
        "CidrBlock": "10.100.3.0/24",
        "AssignIpv6AddressOnCreation": false
    }
}
```
新しいパブリックサブネットを環境変数として追加します。
```
export publicSubnetId=`aws ec2 describe-subnets --filters Name=tag:Name,Values=wa-public-subnet-2 --query 'Subnets[*].SubnetId' --output text --region $awsRegion` && echo publicSubnetId=$publicSubnetId >> ~/.bashrc
```
新しいプライベートサブネットを環境変数として追加します。
```
export privateSubnetId=`aws ec2 describe-subnets --filters Name=tag:Name,Values=wa-private-subnet-2 --query 'Subnets[*].SubnetId' --output text --region $awsRegion` && echo privateSubnetId=$privateSubnetId >> ~/.bashrc
```
既存のパブリックルートテーブルを環境変数として追加します。
```
export publicRt=`aws ec2 describe-route-tables --filters Name=tag:Name,Values=wa-public-rt --query 'RouteTables[*].RouteTableId' --output text --region $awsRegion` && echo publicRt=$publicRt >> ~/.bashrc
```
既存のプライベートルートテーブルを環境変数として追加します。
```
export privateRt=`aws ec2 describe-route-tables --filters Name=tag:Name,Values=wa-private-rt --query 'RouteTables[*].RouteTableId' --output text --region $awsRegion` && echo privateRt=$privateRt >> ~/.bashrc
```
2 つ目のアベイラビリティーゾーンで作成されたパブリックサブネットを含む既存のパブリックルートテーブルに関連付ける。
```
aws ec2 associate-route-table --subnet-id $publicSubnetId --route-table-id $publicRt --region $awsRegion
```
出力
```
{
    "AssociationState": {
        "State": "associated"
    },
    "AssociationId": "rtbassoc-0754972f40549eba1"
}
```
2 つ目のアベイラビリティーゾーンで作成されたプライベートサブネットを含む既存のプライベートルートテーブルに関連付ける。
```
aws ec2 associate-route-table --subnet-id $privateSubnetId --route-table-id $privateRt --region $awsRegion
```
出力
```
{
    "AssociationState": {
        "State": "associated"
    },
    "AssociationId": "rtbassoc-0ef7937d44b5f94ee"
}
```

## Task 2
<img src="https://github.com/osonoi/AWS_CLI/blob/main/fig1.jpg">

１つ目のアベイラビリティーゾーンに RDS サブネットを作成します。
```
aws ec2 create-subnet --vpc-id $VPC --cidr-block "10.100.4.0/24" --availability-zone $awsAZ1 --tag-specifications 'ResourceType=subnet, Tags=[{Key=Name,Value=wa-rds-subnet-1}]' --region $awsRegion
```
出力
```
{
    "AssociationState": {
        "State": "associated"
    },
    "AssociationId": "rtbassoc-0ef7937d44b5f94ee"
}
sh-4.2$ aws ec2 create-subnet --vpc-id $VPC --cidr-block "10.100.4.0/24" --availability-zone $awsAZ1 --tag-specifications 'ResourceType=subnet, Tags=[{Key=Name,Value=wa-rds-subnet-1}]' --region $awsRegion
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "use1-az4",
        "Tags": [
            {
                "Value": "wa-rds-subnet-1",
                "Key": "Name"
            }
        ],
        "AvailableIpAddressCount": 251,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:us-east-1:238827011620:subnet/subnet-03d390f2fdfb8e35b",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0a296354b51520c5d",
        "State": "available",
        "AvailabilityZone": "us-east-1a",
        "SubnetId": "subnet-03d390f2fdfb8e35b",
        "OwnerId": "238827011620",
        "CidrBlock": "10.100.4.0/24",
        "AssignIpv6AddressOnCreation": false
    }
}
```
2 つ目のアベイラビリティーゾーンに Amazon RDS サブネットを作成します。
```
aws ec2 create-subnet --vpc-id $VPC --cidr-block "10.100.5.0/24" --availability-zone $awsAZ2 --tag-specifications 'ResourceType=subnet, Tags=[{Key=Name,Value=wa-rds-subnet-2}]' --region $awsRegion
```
出力
```
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "use1-az6",
        "Tags": [
            {
                "Value": "wa-rds-subnet-2",
                "Key": "Name"
            }
        ],
        "AvailableIpAddressCount": 251,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:us-east-1:238827011620:subnet/subnet-0d651df68f4a6b9a7",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0a296354b51520c5d",
        "State": "available",
        "AvailabilityZone": "us-east-1b",
        "SubnetId": "subnet-0d651df68f4a6b9a7",
        "OwnerId": "238827011620",
        "CidrBlock": "10.100.5.0/24",
        "AssignIpv6AddressOnCreation": false
    }
}
```
新しい Amazon RDS サブネットの最初のものを環境変数として追加します。
```
export rdsSubnet1Id=`aws ec2 describe-subnets --filters Name=tag:Name,Values=wa-rds-subnet-1 --query 'Subnets[*].SubnetId' --output text --region $awsRegion` && echo rdsSubnet1Id=$rdsSubnet1Id >> ~/.bashrc
```
新しい Amazon RDS サブネットの 2 つ目を環境変数として追加します。
```
export rdsSubnet2Id=`aws ec2 describe-subnets --filters Name=tag:Name,Values=wa-rds-subnet-2 --query 'Subnets[*].SubnetId' --output text --region $awsRegion` && echo rdsSubnet2Id=$rdsSubnet2Id >> ~/.bashrc
```
プライベートルートテーブルを最初の Amazon RDS サブネットにアソシエイト。
```
aws ec2 associate-route-table --subnet-id $rdsSubnet1Id --route-table-id $privateRt --region $awsRegion
```
出力
```
{
    "AssociationState": {
        "State": "associated"
    },
    "AssociationId": "rtbassoc-0720c4747473f5566"
}
```
プライベートルートテーブルを 2 つ目の Amazon RDS サブネットに関連付けます。
```
aws ec2 associate-route-table --subnet-id $rdsSubnet2Id --route-table-id $privateRt --region $awsRegion
```
出力
```
{
    "AssociationState": {
        "State": "associated"
    },
    "AssociationId": "rtbassoc-05a3d042dbbcf8d39"
}
```
新しい Amazon RDS サブネットの両方を使用して Amazon RDS サブネットグループを作成します。
```
aws rds create-db-subnet-group --db-subnet-group-name "wa-rds-subnet-group" --db-subnet-group-description "WA RDS Subnet Group" --subnet-ids $rdsSubnet1Id $rdsSubnet2Id --region $awsRegion
```
出力
```
{
    "DBSubnetGroup": {
        "Subnets": [
            {
                "SubnetStatus": "Active",
                "SubnetIdentifier": "subnet-03d390f2fdfb8e35b",
                "SubnetOutpost": {},
                "SubnetAvailabilityZone": {
                    "Name": "us-east-1a"
                }
            },
            {
                "SubnetStatus": "Active",
                "SubnetIdentifier": "subnet-0d651df68f4a6b9a7",
                "SubnetOutpost": {},
                "SubnetAvailabilityZone": {
                    "Name": "us-east-1b"
                }
            }
        ],
        "VpcId": "vpc-0a296354b51520c5d",
        "DBSubnetGroupDescription": "WA RDS Subnet Group",
        "SubnetGroupStatus": "Complete",
        "DBSubnetGroupArn": "arn:aws:rds:us-east-1:238827011620:subgrp:wa-rds-subnet-group",
        "DBSubnetGroupName": "wa-rds-subnet-group"
    }
}
```
Amazon RDS が使用するセキュリティグループを作成します。
```
aws ec2 create-security-group --description "RDS Security group" --group-name "wa-rds-sg" --vpc-id $VPC --region $awsRegion
```
出力
```
{
    "GroupId": "sg-08fe48f5a098d6e3f"
}
```
Amazon RDS セキュリティグループ ID を環境変数として設定します。
```
export rdsSg=`aws ec2 describe-security-groups --filters Name=group-name,Values=wa-rds-sg --query 'SecurityGroups[*].GroupId' --output text --region $awsRegion` && echo rdsSg=$rdsSg >> ~/.bashrc
```
Amazon EC2 データベースインスタンスが使用する 2 つ目のセキュリティグループ ID の環境変数を設定します。
```
export ec2DbSg=`aws ec2 describe-security-groups --filters Name=group-name,Values=wa-database-sg --query 'SecurityGroups[*].GroupId' --output text --region $awsRegion` && echo ec2DbSg=$ec2DbSg >> ~/.bashrc
```
Amazon RDS セキュリティグループが Amazon EC2 データベースインスタンスが使用するセキュリティグループと通信できるようにします。
```
aws ec2 authorize-security-group-ingress --group-id $rdsSg --source-group $ec2DbSg --protocol "tcp" --port "3306" --region $awsRegion
```
マルチ AZ Amazon RDS インスタンスを作成します。
```
aws rds create-db-instance --db-name "WaRdsDb" --db-instance-identifier "waDbInstance" --allocated-storage 20 --db-instance-class db.t3.micro --engine "mariadb" --master-username "mainuser" --master-user-password "WaStr0ngP4ssw0rd" --vpc-security-group-ids $rdsSg --db-subnet-group-name "wa-rds-subnet-group" --multi-az --no-publicly-accessible --backup-retention-period 0 --region $awsRegion
```
出力
```
{
    "DBInstance": {
        "PubliclyAccessible": false,
        "MasterUsername": "mainuser",
        "MonitoringInterval": 0,
        "LicenseModel": "general-public-license",
        "VpcSecurityGroups": [
            {
                "Status": "active",
                "VpcSecurityGroupId": "sg-08fe48f5a098d6e3f"
            }
        ],
        "CopyTagsToSnapshot": false,
        "OptionGroupMemberships": [
            {
                "Status": "in-sync",
                "OptionGroupName": "default:mariadb-10-11"
            }
        ],
        "PendingModifiedValues": {
            "MasterUserPassword": "****"
        },
        "Engine": "mariadb",
        "MultiAZ": true,
        "DBSecurityGroups": [],
        "DBParameterGroups": [
            {
                "DBParameterGroupName": "default.mariadb10.11",
                "ParameterApplyStatus": "in-sync"
            }
        ],
        "PerformanceInsightsEnabled": false,
        "AutoMinorVersionUpgrade": true,
        "PreferredBackupWindow": "05:23-05:53",
        "DBSubnetGroup": {
            "Subnets": [
                {
                    "SubnetStatus": "Active",
                    "SubnetIdentifier": "subnet-03d390f2fdfb8e35b",
                    "SubnetOutpost": {},
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1a"
                    }
                },
                {
                    "SubnetStatus": "Active",
                    "SubnetIdentifier": "subnet-0d651df68f4a6b9a7",
                    "SubnetOutpost": {},
                    "SubnetAvailabilityZone": {
                        "Name": "us-east-1b"
                    }
                }
            ],
            "DBSubnetGroupName": "wa-rds-subnet-group",
            "VpcId": "vpc-0a296354b51520c5d",
            "DBSubnetGroupDescription": "WA RDS Subnet Group",
            "SubnetGroupStatus": "Complete"
        },
        "ReadReplicaDBInstanceIdentifiers": [],
        "AllocatedStorage": 20,
        "DBInstanceArn": "arn:aws:rds:us-east-1:238827011620:db:wadbinstance",
        "BackupRetentionPeriod": 0,
        "DBName": "WaRdsDb",
        "PreferredMaintenanceWindow": "fri:03:40-fri:04:10",
        "DBInstanceStatus": "creating",
        "IAMDatabaseAuthenticationEnabled": false,
        "EngineVersion": "10.11.8",
        "DeletionProtection": false,
        "DomainMemberships": [],
        "StorageType": "gp2",
        "DbiResourceId": "db-EWAO2B4QVDWNI5GTDD4RBHTH24",
        "CACertificateIdentifier": "rds-ca-rsa2048-g1",
        "StorageEncrypted": false,
        "AssociatedRoles": [],
        "DBInstanceClass": "db.t3.micro",
        "DbInstancePort": 0,
        "DBInstanceIdentifier": "wadbinstance"
    }
}
```
## Task 3
## AWS Systems Manager Parameter Store データベースを更新する

以下のコマンドを実行して、RDS エンドポイントの可用性を確認します。
```
aws rds describe-db-instances --db-instance-identifier "waDbInstance" --query 'DBInstances[*].Endpoint.Address' --output text --region $awsRegion
```
出力
```
wadbinstance.cot2b1yqmec9.us-east-1.rds.amazonaws.com
```
AWS Systems Manager Parameter Store を更新する
```

```
出力
```
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
|                                                                           GetParameters                                                                           |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+
||                                                                           Parameters                                                                            ||
|+------------------------------------------------------------+-----------+-------------------+---------------+---------+------------------------------+-----------+|
||                             ARN                            | DataType  | LastModifiedDate  |     Name      |  Type   |            Value             |  Version  ||
|+------------------------------------------------------------+-----------+-------------------+---------------+---------+------------------------------+-----------+|
||  arn:aws:ssm:us-east-1:238827011620:parameter/DbPrivateDns |  text     |  1724489463.87    |  DbPrivateDns |  String |  ip-10-100-1-97.ec2.internal |  1        ||
|+------------------------------------------------------------+-----------+-------------------+---------------+---------+------------------------------+-----------+|
```
Amazon RDS エンドポイントを環境変数として設定します。
```
export rdsEndPoint=`aws rds describe-db-instances --db-instance-identifier "waDbInstance" --query 'DBInstances[*].Endpoint.Address' --output text --region $awsRegion` && echo rdsEndPoint=$rdsEndPoint >> ~/.bashrc
```
Amazon RDS エンドポイントを指すようにパラメータストアの値を更新します。
```
aws ssm put-parameter --name "DbPrivateDns" --value $rdsEndPoint --overwrite --region $awsRegion
```
出力
```
{
    "Tier": "Standard",
    "Version": 2
}
```
次のコマンドを実行して、パラメータストアが Amazon RDS エンドポイントで更新されていることを確認します。
```
aws ssm get-parameters --names "DbPrivateDns" --output table --region $awsRegion
```
出力
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
|                                                                                        GetParameters                                                                                        |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
||                                                                                        Parameters                                                                                         ||
|+------------------------------------------------------------+-----------+-------------------+---------------+---------+---------------------------------------------------------+----------+|
||                             ARN                            | DataType  | LastModifiedDate  |     Name      |  Type   |                          Value                          | Version  ||
|+------------------------------------------------------------+-----------+-------------------+---------------+---------+---------------------------------------------------------+----------+|
||  arn:aws:ssm:us-east-1:238827011620:parameter/DbPrivateDns |  text     |  1724497618.98    |  DbPrivateDns |  String |  wadbinstance.cot2b1yqmec9.us-east-1.rds.amazonaws.com  |  2       ||
|+------------------------------------------------------------+-----------+-------------------+---------------+---------+---------------------------------------------------------+----------+|
```

## Task 4
<img src="https://github.com/osonoi/AWS_CLI/blob/main/fig1.jpg">

Application Load Balancer (ALB) のセキュリティグループを作成します。
```
aws ec2 create-security-group --description "ALB Security group" --group-name "wa-alb-sg" --vpc-id $VPC --region $awsRegion
```
出力
```
{
    "GroupId": "sg-09ff981c9cf8df36b"
}
```
Application Load Balancer セキュリティグループを環境変数として設定します
```
export albSg=`aws ec2 describe-security-groups --filters Name=group-name,Values=wa-alb-sg --query 'SecurityGroups[*].GroupId' --output text --region $awsRegion` && echo albSg=$albSg >> ~/.bashrc
```
Application Load Balancer セキュリティグループを介した HTTP アクセスを許可します。
```
aws ec2 authorize-security-group-ingress --group-id $albSg --protocol "tcp" --port "80" --cidr "0.0.0.0/0" --region $awsRegion
```
Application Load Balancer が使用する最初のパブリックサブネットを環境変数として設定します。
```
export albSubnet1Id=`aws ec2 describe-subnets --filters Name=tag:Name,Values=wa-public-subnet-1 --query 'Subnets[*].SubnetId' --output text --region $awsRegion` && echo albSubnet1Id=$albSubnet1Id >> ~/.bashrc
```
Application Load Balancer が使用する 2 つ目のパブリックサブネットを環境変数として設定します。
```
export albSubnet2Id=`aws ec2 describe-subnets --filters Name=tag:Name,Values=wa-public-subnet-2 --query 'Subnets[*].SubnetId' --output text --region $awsRegion` && echo albSubnet2Id=$albSubnet2Id >> ~/.bashrc
```
Application Load Balancer を作成します。
```
aws elbv2 create-load-balancer --name "waAlb" --subnets $albSubnet1Id $albSubnet2Id --security-groups $albSg --type "application" --region $awsRegion
```
出力
```
{
    "LoadBalancers": [
        {
            "IpAddressType": "ipv4",
            "VpcId": "vpc-0a296354b51520c5d",
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-1:238827011620:loadbalancer/app/waAlb/a6c9b0c0db02c959",
            "State": {
                "Code": "provisioning"
            },
            "DNSName": "waAlb-503572362.us-east-1.elb.amazonaws.com",
            "SecurityGroups": [
                "sg-09ff981c9cf8df36b"
            ],
            "LoadBalancerName": "waAlb",
            "CreatedTime": "2024-08-24T11:15:05.740Z",
            "Scheme": "internet-facing",
            "Type": "application",
            "CanonicalHostedZoneId": "Z35SXDOTRQ7X7K",
            "AvailabilityZones": [
                {
                    "SubnetId": "subnet-035dfb7bd05f76724",
                    "LoadBalancerAddresses": [],
                    "ZoneName": "us-east-1a"
                },
                {
                    "SubnetId": "subnet-0f191ca93f95b8082",
                    "LoadBalancerAddresses": [],
                    "ZoneName": "us-east-1b"
                }
            ]
        }
    ]
}
```
ターゲットグループを作成します。
```
aws elbv2 create-target-group --name "waAutoscale-tg" --protocol "HTTP" --port 80 --vpc-id $VPC --target-type "instance" --region $awsRegion
```
出力
```
{
    "TargetGroups": [
        {
            "HealthCheckPath": "/",
            "HealthCheckIntervalSeconds": 30,
            "VpcId": "vpc-0a296354b51520c5d",
            "Protocol": "HTTP",
            "HealthCheckTimeoutSeconds": 5,
            "TargetType": "instance",
            "HealthCheckProtocol": "HTTP",
            "Matcher": {
                "HttpCode": "200"
            },
            "UnhealthyThresholdCount": 2,
            "HealthyThresholdCount": 5,
            "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:238827011620:targetgroup/waAutoscale-tg/21f859df0a64cb1a",
            "HealthCheckEnabled": true,
            "HealthCheckPort": "traffic-port",
            "Port": 80,
            "TargetGroupName": "waAutoscale-tg"
        }
    ]
}
```
ターゲットグループを環境変数として設定します。
```
export waTg=`aws elbv2 describe-target-groups --names waAutoscale-tg --query 'TargetGroups[*].TargetGroupArn' --output text --region $awsRegion` && echo waTg=$waTg >> ~/.bashrc
```
Application Load Balancer の Amazon Resource Name (ARN) を環境変数として設定します。
```
export albArn=`aws elbv2 describe-load-balancers --names waAlb --query 'LoadBalancers[*].LoadBalancerArn' --output text --region $awsRegion` && echo albArn=$albArn >> ~/.bashrc
```
テンプレート listener.json ファイルをシェルインスタンスにダウンロードします。
```
cd ~
wget https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/ILT-TF-200-CSAWAF-10-EN/listener-source.json
```
出力
```
--2024-08-24 11:19:05--  https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/ILT-TF-200-CSAWAF-10-EN/listener-source.json
Resolving aws-tc-largeobjects.s3-us-west-2.amazonaws.com (aws-tc-largeobjects.s3-us-west-2.amazonaws.com)... 52.92.176.170, 52.218.170.1, 3.5.87.114, ...
Connecting to aws-tc-largeobjects.s3-us-west-2.amazonaws.com (aws-tc-largeobjects.s3-us-west-2.amazonaws.com)|52.92.176.170|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 96 [application/json]
Saving to: ‘listener-source.json’

100%[===================================================================================================================================================================>] 96          --.-K/s   in 0s

2024-08-24 11:19:06 (1.37 MB/s) - ‘listener-source.json’ saved [96/96]
```
テンプレート内の環境変数をターゲットグループに置き換え、新しいテンプレートファイルを作成します。
```
envsubst < "listener-source.json" > "listener.json"
```
テンプレートファイルを参照して Application Load Balancer のリスナーを作成します。
```
aws elbv2 create-listener --load-balancer-arn $albArn --protocol "HTTP" --port 80 --default-actions file://listener.json --region $awsRegion
```
出力
```
{
    "Listeners": [
        {
            "Protocol": "HTTP",
            "DefaultActions": [
                {
                    "ForwardConfig": {
                        "TargetGroupStickinessConfig": {
                            "Enabled": false
                        },
                        "TargetGroups": [
                            {
                                "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:238827011620:targetgroup/waAutoscale-tg/21f859df0a64cb1a",
                                "Weight": 1
                            }
                        ]
                    },
                    "TargetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:238827011620:targetgroup/waAutoscale-tg/21f859df0a64cb1a",
                    "Type": "forward",
                    "Order": 1
                }
            ],
            "LoadBalancerArn": "arn:aws:elasticloadbalancing:us-east-1:238827011620:loadbalancer/app/waAlb/a6c9b0c0db02c959",
            "Port": 80,
            "ListenerArn": "arn:aws:elasticloadbalancing:us-east-1:238827011620:listener/app/waAlb/a6c9b0c0db02c959/54a123f9e846f811"
        }
    ]
}
```

