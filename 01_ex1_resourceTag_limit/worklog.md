# このディレクトリでやりたいこと

[例1](https://docs.aws.amazon.com/ja_jp/apigateway/latest/developerguide/apigateway-tagging-iam-policy.html#apigateway-tagging-iam-policy-example-1)の内容を実際に動かしてみて、何が出来るのかを理解する

# 環境

* IAMで以下のポリシーをもつexample1ユーザが作成されている

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "apigateway:GET",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "apigateway:*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/iamrole": "readWrite"
        }
      }
    }
  ]
}
```

* API Gatewayでexample1というAPI名のAPIサンプルが作成されている
    * APIサンプルの作成方法：https://gyazo.com/750ce49fae90abd21ef904d22d8c8d3a
        * APIを作成 > REST API
        * 「APIの例」を選択
        * インポート

| key | value |
| --- | --- |
| API名 | example1 |
| ID | {rest-api-id} |
| リソースID（/） | {resource-id} |


# やったこと

* まずはiamの権限をもつユーザでポリシーの適用を確認

```
$ aws iam list-attached-user-policies --user-name example-1
{
    "AttachedPolicies": [
        {
            "PolicyName": "example1-policy",
            "PolicyArn": "arn:aws:iam::xxxxxxxxxxxx:policy/example1-policy"
        }
    ]
}
```
```
$ aws iam get-policy-version --version-id v1 --policy-arn arn:aws:iam::xxxxxxxxxxxx:policy/example1-policy
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": "apigateway:GET",
                    "Resource": "*"
                },
                {
                    "Effect": "Allow",
                    "Action": "apigateway:*",
                    "Resource": "*",
                    "Condition": {
                        "StringEquals": {
                            "aws:ResourceTag/iamrole": "readWrite"
                        }
                    }
                }
            ]
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2020-02-28T01:52:07Z"
    }
}
```

* example1ユーザに切り替え、切り替わったことを確認

Arnの末尾が「example-1」になっていることを確認します。

```
$ aws sts get-caller-identity
{
    "UserId": "xxxx",
    "Account": "xxxxxxxxxxxx",
    "Arn": "arn:aws:iam::xxxxxxxxxxxx:user/example-1"
}
```

* 「example1」APIにタグが付いていないことを確認

```
$ aws apigateway get-tags --resource-arn arn:aws:apigateway:ap-northeast-1::/restapis/{rest-api-id}
{
    "tags": {}
}
```

* put-method出来ないことを確認

```
$ aws apigateway put-method \
  --rest-api-id {rest-api-id} \
  --resource-id {resource-id} \
  --http-method POST \
  --authorization-type "NONE" \
  --region ap-northeast-1
An error occurred (AccessDeniedException) when calling the PutMethod operation: User: arn:aws:iam::xxxxxxxxxxxx:user/example-1 is not authorized to perform: apigateway:PUT on resource: arn:aws:apigateway:ap-northeast-1::/restapis/{rest-api-id}/resources/{resource-id}/methods/POST  
```

* タグ付け

```
$ aws apigateway tag-resource \
--resource-arn arn:aws:apigateway:ap-northeast-1::/restapis/{rest-api-id} \
--tags iamrole=readWrite
```

* put-method出来ることを確認

```
$ aws apigateway put-method \
  --rest-api-id {rest-api-id} \
  --resource-id {resource-id} \
  --http-method POST \
  --authorization-type "NONE" \
  --region ap-northeast-1

{
    "httpMethod": "POST",
    "authorizationType": "NONE",
    "apiKeyRequired": false
}
```

# まとめ

* リソースのタグで操作を制御できる