# version
```
$ ecs-cli --version
ecs-cli version 1.21.0 (bb0b8f0)
```

# 初期設定
### Profile設定
- ecs-cliのprofile設定から操作する場合は、作成必要
```
$ ecs-cli configure profile --profile-name sample01 --access-key xxxx --secret-key xxxx

$ cat ~/.ecs/credentials
version: v1
default: sample01
ecs_profiles:
  sample01:
    aws_access_key_id: xxxx
    aws_secret_access_key: xxxx
```

- aws-cliのprofile設定で操作する場合は、ecs-cliのprofile作成は不要(`--aws-profile`で使用するaws-cliのprofileを指定する)

### Cluster設定
- cluster情報の設定
```
$ ecs-cli configure --cluster sample01 --default-launch-type FARGATE --config-name sample01 --region ap-northeast-1

$ cat ~/.ecs/config
version: v1
default: sample01
clusters:
  sample01:
    cluster: sample01
    region: ap-northeast-1
    default_launch_type: FARGATE
```

### ロール作成
- タスク実行ロール(ecs-agent用のロール)
```
$ aws iam create-role --role-name ecsTaskExecutionRole --assume-role-policy-document file://task-execution-assume-role.json --profile megun002
$ aws iam attach-role-policy --role-name ecsTaskExecutionRole --policy-arn arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy --profile megun002
```

- タスクロール(コンテナ用のロール)
```
$ aws iam create-role --role-name ecs-exec-task-role --assume-role-policy-document file://task-execution-assume-role.json --profile megun002
$ aws iam put-role-policy --role-name ecs-exec-task-role --policy-name ecs-exec-task-role-policy --policy-document file://ecs-exec-task-role-policy.json --profile megun002
```

# 操作
## Cluster
- VPC+ECSClusterを作成
```
$ ecs-cli up --cluster-config sample01 --aws-profile megun002
```

- VPC+ECSCluster+ECSInstanceを作成
```
$ ecs-cli up --capability-iam --size 2 --instance-type t3.medium --launch-type EC2 --cluster-config sample01 --aws-profile megun002
```

- `ecs-cli up` で作成されたリソース削除
```
$ ecs-cli down --force --cluster-config sample01 --aws-profile megun002
```

## Task
- docker-composeからタスク定義の作成
```
$ ecs-cli compose --project-name hoge --file docker-compose.yml create --launch-type FARGATE --cluster-config sample01 --aws-profile megun002
```

## Service
- service作成とタスク実行
```
$ ecs-cli compose --project-name hoge --file docker-compose.yml service up --cluster-config sample01 --aws-profile megun002
```

- タスク数の調整
```
$ ecs-cli compose --project-name hoge service scale 2 --cluster-config sample01 --aws-profile megun002
```

- service削除
```
$ ecs-cli compose --project-name hoge service down --cluster-config sample01 --aws-profile megun002
```

- ServiceDiscovery
```
裏でCFnがPrivateDNSZoneとCloudMapを作成する
$ ecs-cli compose --project-name backend service up --private-dns-namespace sample-zone --enable-service-discovery --vpc vpc-06b32e8e592788654 --cluster-config sample01 --aws-profile megun002
$ ecs-cli compose --project-name frontend service up --private-dns-namespace sample-zone --enable-service-discovery --vpc vpc-06b32e8e592788654 --cluster-config sample01 --aws-profile megun002


削除するとき
$ ecs-cli compose --project-name frontend service down --delete-namespace --cluster-config sample01 --aws-profile megun002
$ ecs-cli compose --project-name backend service down --delete-namespace --cluster-config sample01 --aws-profile megun002
```

# その他
## `ecs exec` でコンテナへアクセス
- [参考](https://aws.amazon.com/jp/blogs/news/new-using-amazon-ecs-exec-access-your-containers-fargate-ec2/)
- [AWS CLI 用の Session Manager pluginのインストール](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)
- IAMポリシー作成
  - タスクロールに下記のSSMアクセス許可追加する
    ```
    {
       "Version": "2012-10-17",
       "Statement": [
           {
           "Effect": "Allow",
           "Action": [
                "ssmmessages:CreateControlChannel",
                "ssmmessages:CreateDataChannel",
                "ssmmessages:OpenControlChannel",
                "ssmmessages:OpenDataChannel"
           ],
          "Resource": "*"
          }
       ]
    }
    ```
- serviceの設定でenable-execute-commandを有効にしてタスク起動しなおす(ecs-cliではできなさそう)
  ```
  aws ecs update-service \
      --cluster sample01 \
      --service hoge \
      --enable-execute-command
  
  aws ecs describe-services \
      --cluster sample01 \
      --services hoge
  ```
- コンテナに対してコマンド実行
  ```
  aws ecs execute-command --cluster sample01 \
      --task task-id \
      --container container-name \
      --interactive \
      --command "/bin/sh"
  ```

## terraformとかとのすみわけ
- terraformなどでやったほうがいいと思われるもの
  - VPC
  - Cluster
  - Loadbalancerとか
- ecs-cliでやったほうがいいと思われるもの
  - task定義
  - service
