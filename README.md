# Laravel/nginxを使ったfargate構築

## ローカルでのDocker動作方法

#### DockerファイルからBuild
```
docker build -t ローカルでのイメージタグ:latest -f Dockerfileまでのパス/Dockerfile .
```

#### Dockerコンテナに入らずコマンド実行例
```
docker run -v $(pwd)/src:/application -it ecs-hands-on/laravel:latest php artisan make:command HelloWorld
```
HelloWorld.phpが作成されるコマンド
#### コンテナの中に入る
```
docker run -v $(pwd)/src:/application -it ecs-hands-on/laravel:latest bash
```
もちろんコンテナの中にも入れる。

## ECRへPush
#### AWSコンソール上で適当にリポジトリ作成

#### 認証トークンを取得し、レジストリに対して Docker クライアントを認証
```
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin アカウントID.dkr.ecr.ap-northeast-1.amazonaws.com
```

#### Dockerイメージを構築
```
docker build -t 使用したいイメージタグ -f Dockerファイルのパス .
```

#### ローカルのイメージにAWSリポジトリ除法をタグ付け
```
docker tag タグ付けしたいイメージ:latest アカウントID.dkr.ecr.ap-northeast-1.amazonaws.com/ECR上のリポジトリ名:latest
```

#### ECRへPush
```
docker push アカウントID.dkr.ecr.ap-northeast-1.amazonaws.com/ECRのリポジトリ名:latest
```

## ECS/fargateとの連携

#### タスクを作成
```
aws ecs register-task-definition --cli-input-json file://ecs-task-definition-for-command.json
```

#### タスクを実行
```
aws ecs run-task \\n--cluster クラスター名 \\n--task-definition タスク名 \\n--overrides '{"containerOverrides": [{"name":"laravel","command": ["sh","-c","走らせたいartisanコマンド"]}]}' \\n--launch-type FARGATE \\n--network-configuration "awsvpcConfiguration={subnets=[サブネットID],securityGroups=[セキュリティーグループID],assignPublicIp=ENABLED}"
```

#### タスクを作成
```
aws ecs register-task-definition --cli-input-json file://ecs-task-definition-for-web.json
```

#### サービスを実行
```
aws ecs create-service \\n --cluster クラスター名 \\n --service-name サービス名 \\n --task-definition タスク名 \\n --launch-type FARGATE \\n--desired-count 1 \\n--network-configuration "awsvpcConfiguration={subnets=[サブネットID],securityGroups=[セキュリティーグループID],assignPublicIp=ENABLED}"
```


