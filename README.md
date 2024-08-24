# AWS_CLI

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
