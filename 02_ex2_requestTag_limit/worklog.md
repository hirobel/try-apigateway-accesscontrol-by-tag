# このディレクトリでやりたいこと

[例2](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/apigateway-tagging-iam-policy.html#apigateway-tagging-iam-policy-example-2)の内容を実際に動かしてみて、何が出来るのかを理解する

# 環境

* IAMで以下のポリシーをもつexample2ユーザが作成されている

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "apigateway:*"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Deny",
            "Action": "apigateway:POST",
            "Resource": "arn:aws:apigateway:*::/restapis/*/stages",
            "Condition": {
                "Null": {
                    "aws:RequestTag/stage": "true"
                }
            }
        },
        {
            "Effect": "Deny",
            "Action": "apigateway:POST",
            "Resource": "arn:aws:apigateway:*::/restapis/*/stages",
            "Condition": {
                "ForAnyValue:StringNotEquals": {
                    "aws:RequestTag/stage": [
                        "beta",
                        "gamma",
                        "prod"
                    ]
                }
            }
        }
    ]
}
```

* API Gatewayでexample2というAPI名のAPIサンプルが作成されている

# やったこと

* 一旦デプロイメントをダミーのステージに対して作成

```
$ aws apigateway create-deployment --rest-api-id {restapi-id} --stage-name dummy
{
    "id": "{deployment-id}",
    "createdDate": 1585986393
}
```

* hogeステージの作成→失敗

```
$ aws apigateway create-stage \
--rest-api-id {restapi-id} \
--stage-name prodstage \
--deployment-id {deployment-id} \
--tags stage=hoge
An error occurred (AccessDeniedException) when calling the CreateStage operation: User: arn:aws:iam::{aws-account-id}:user/example-2 is not authorized to perform: apigateway:POST on resource: arn:aws:apigateway:ap-northeast-1::/restapis/{restapi-id}/stages with an explicit deny
```

* betaステージの作成→成功

```
$ aws apigateway create-stage \
--rest-api-id {restapi-id} \
--stage-name prodstage \
--deployment-id {deployment-id} \
--tags stage=beta
{
    "deploymentId": "{deployment-id}",
    "stageName": "prodstage",
    "cacheClusterEnabled": false,
    "cacheClusterStatus": "NOT_AVAILABLE",
    "methodSettings": {},
    "tracingEnabled": false,
    "tags": {
        "stage": "beta"
    },
    "createdDate": 1585986562,
    "lastUpdatedDate": 1585986563
}
```