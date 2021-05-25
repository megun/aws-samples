# version
```
$ copilot -v
copilot version: v1.7.1
```

# 認証
- aws-cliの情報を使う

# 操作
## サンプル１
- 下記のように実行して応答に答えてくと
  ```
  $ cd demo
  $ copilot init
  ```
- 下記の色々なリソースを作ってくれる(裏でCFnが作ってる)
  - IAM
  - ECRリポジトリ作成とビルドしてプッシュ
  - KMS
  - S3
  - ECS
  - VPC
  - ServiceDiscovery
  - ELB
- 状況を確認
  ```
  $ copilot app ls
  $ copilot app show
  $ copilot env ls
  $ copilot svc show
  $ copilot svc ls
  $ copilot svc logs
  $ copilot svc status
  ```
- 削除
  ```
  $ copilot app delete
  ```

# その他
## 所感
- まるっと色々やってくれるので便利だけど、融通はきかなさそう？
