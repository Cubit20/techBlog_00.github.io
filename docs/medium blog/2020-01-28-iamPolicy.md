---
layout: post
title : Principle of Least Privilege with IAM Policy
category : medium blog
order : 2
date: 2020-01-28
---

![iamPolicy](./assets/images/mediumBlog/../../../../../assets/images/mediumBlog/20.01.28-iamPolicy/iamPolicy_01.jpg)

Principle of Least Privilege, 굳이 직역하자면 최소 권한의 원칙(?)일거 같고, 풀어서 말하자면 특정 임무를 수새항하는 객체에 그 특정 임무를 수행하는데 꼭 필요한 최소한의 권한만 부여 및 허용하는 방법론입니다.

AWS에서는 IAM Policy에 권한을 명시하고 IAM User나 IAM Policy를 부여하는 방식으로 AWS Resources에 대한 접근 제어를 할 수 있습니다. 고로 IAM Policy를 어떻게 사용하느냐에 따라서 AWS 클라우드 상에서의 보안을 더욱 견고하게 할 수 있는데요. 몇가지 예를 통해서 올바른 IAM Policy를 만드는 법을 알아보겠습니다.

* * *

> Scenario 1 - 특정 IAM User에게 least_privilege_principle S3 Bucket에 있는 객체들을 보고, 읽고, 지우는 권한을 부여하십시오.

이 경우 AWS 초심자의 경우에는 잘 몰라서, AWS를 잘 사용한다고 하는 분들은 귀찮아서 AWS Managed Policy인 AmazonS3FullAccess 권한을 부여하곤 하는데요. (사실 저도 테스트용이라는 변명과 함께 종종 사용하는 방법입니다. 😆) 이 정책에 부여된 권한을 JSON으로 본다면 아래와 같습니다.

    {
        "Version" : "2012-10-17"
        "Statement" : [
            {
                "Effect" : "Allow",
                "Action" : "s3:*",
                "Resource" : "*"
            }
        ]
    }

해당 AWS 계정에 있는 모든 S3 Bucket에 S3 관련 모든 Action들을 취할 수 있는 권한입니다. _`least_private_principle`_ Bucket에 대한 권한만 필요하므로 약간의 수정을 해보겠습니다.

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "s3:*",
                "Resource": [
                    "arn:aws:s3:::least_privilege_principle",
                    "arn:aws:s3:::least_privilege_principle/*"
                ]
            }
        ]
    }

조금 더 나아지기는 했지만 Action 부분에 대한 수정이 필요할듯 하네요. ListBucket, GetObject, DeleteObject Action만 실행 할수있게 수정해 보겠습니다.

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "s3:ListBucket",
                "Resource": "arn:aws:s3:::least_privilege_principle"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:DeleteObject"
                ],
                "Resource": "arn:aws:s3:::least_privilege_principle/*"
            }
        ]
    }

> Scenario 2 - 특정 IAM User에게 _`least_privilege_principle`_ S3 Bucket에 있는 객체들을 보고, 읽고, 지우는 권한을 부여하십시오. 단 GetObject와 DeleteObject Action은 해당 IAM 유저가 회사 네트워크에 접속한 상태에서만 실행할 수 있습니다.

이 경우 IAM Policy 요소 중에서 Condition을 이용해서 특정 IP 대역대에서 오는 요청에 대해서만 허가하는 조건을 넣을 수 있습니다. 회사 네트워크 IP 대역대가 123.123.123.123/32라고 가정을 한다면 아래와 같은 조건문을 넣어서 IP주소 기반 접근제어를 할 수 있습니다.

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "s3:ListBucket",
                "Resource": "arn:aws:s3:::least_privilege_principle"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:DeleteObject"
                ],
                "Resource": "arn:aws:s3:::least_privilege_principle/*",
                "Condition": {
                    "IpAddress": {
                        "aws:SourceIp": "123.123.123.123/32"
                    }
                }    
            }
        ]
    }

> Scenario 3 - 특정 IAM User에게 _`least_privilege_principle`_ S3 Bucket에 있은 객체들을 보고, 읽고 지우는 권한을 부여하십시오. 단 GetObject와 DeleteObject Action은 해당 IAM 유저가 회사 네트워크에 접속한 상태에서만 실행할 수 있어야 합니다. 해당 IAM 유저에 MFA가 설정되어 있지 않을 경우 DeleteObject를 실행할 수 없어야 합니다.

아무리 완벽한 IAM Policy를 생성했다고 해도 해당 정책을 가진 IAM User의 Credential이 유출되었다면... 상상만 해도 끔찍한 일이네요. MFA 역시 Condition을 통해서 제어할 수 있습니다.

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "s3:ListBucket",
                "Resource": "arn:aws:s3:::least_privilege_principle"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject"
                ],
                "Resource": "arn:aws:s3:::least_privilege_principle/*",
                "Condition": {
                    "IpAddress": {
                        "aws:SourceIp": "123.123.123.123/32"
                    }
                }
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:DeleteObject"
                ],
                "Resource": "arn:aws:s3:::least_privilege_principle/*",
                "Condition": {
                    "Bool": {
                        "aws:MultiFactorAuthPresent": "true"
                    },
                    "IpAddress": {
                        "aws:SourceIp": "123.123.123.123/32"
                    }
                }
            }
        ]
    }

> Scenario 4 - 한국 시간 기준 업무시간 (09:00 am ~ 05:00 pm) 이외에은 AWS 상에서 어떤 작업도 할 수 없어야 합니다.

이 경우에도 역시 Condition을 이용해서 특정 날짜와 시간대에 따라서 접근 제어를 할 수 있지만 여기서 문제점은 **날짜**에 있습니다. ISO 8061 포맷에 맞춰진, 예를 들자면 **2020-01-28T08:00:00+00:00**, 시간대만 지원하므로 매일 매일 수동 혹은 자동으로 YYYY-MM-DD 부분을 업데이트 해줘야 합니다. CloudWatch Event와 AWS Lambda를 사용해서 자동화 해보도록 하겠습니다.

1. 아래와 같은 IAM Policy를 생성합니다. 현재 시간이 한국시간 기준 2020년 1월 28알 오전 9시라고 가정하고 아래와 같은 IAM Policy를 생성하고 부여한다면, 2020년 1월 28일 오후 5시 (UTC기준 오전 8시) 이후에 오는 모든 요청을 거부하게 되겠죠.

        {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Deny",
                    "Action": "*",
                    "Resource": "*",
                    "Condition": {
                        "DateGreaterThanEquals": {
                            "aws:CurrentTime": "2020-01-28T08:00:00+00:00"
                        }
                    }
                }
            ]
        }

2. 아래와 같은 Lambda function을 생성하고, Lambdma 환경 변수에 키 = POLICY_ARN, 값 = 위에서 생성한 IAM Policy ARN을 넣어서 지정합니다. 해당 Lambda fucntion이 실행되는 시점을 기준으로 위에서 생성한 IAM Policy에서의 날짜조건을 오늘 기준으로 업데이트 합니다.

        import json
        import boto3
        import datetime
        import os
        def lambda_handler(event, context):
            
            policyArn = os.environ['POLICY_ARN']
            
            client = boto3.client('iam')
            day = datetime.date.today()
            to_date = datetime.datetime(day.year, day.month, day.day, 8, 00, 00).replace(tzinfo=datetime.timezone.utc).isoformat()
            
            iam_policy = {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Effect": "Deny",
                        "Action": "*",
                        "Resource": "*",
                        "Condition": {
                            "DateGreaterThanEquals": {
                                "aws:CurrentTime": to_date
                            }
                        }
                    }
                ]
            }
            
            lpv_response = client.list_policy_versions(
                PolicyArn=policyArn
            )
            
            if len(lpv_response['Versions']) > 4:
                dpv_response = client.delete_policy_version(
                    PolicyArn=policyArn,
                    VersionId=lpv_response['Versions'][4]['VersionId']
                )
                
            cpv_reponse = client.create_policy_version(
                PolicyArn=policyArn,
                PolicyDocument=json.dumps(iam_policy),
                SetAsDefault=True
            )

3. CloudWatch Event에 Schedule Rule을 생성합니다. UTC 기준 월요일부터 금요일까지 매일 00시에(한국시간 기준 오전 9시) 위에서 생성한 Lambda Function이 실행되도록 설정합니다.

자 그럼 현재 시간이 한국시간 기준 2020년 1월 28일 오후 5시 (UTC 기준 오전 8시)라고 가정한다면, 이 시간 이후로 오는 모든 요청은 Deny 될것입니다. 하루가 지나고 2020년 1월 29일 오전 9시(UTC 기준 00시) 위의 Lambda function이 실행되면 IAM Policy의 날짜 조건은 2020–01–29T08:00:00+00:00로 업데이트가 되어서 당일 기준 오후 5시까지 모든 Action에 대한 Explicy Deny 가 적용되지 않을겁니다. 이후 오후 5시부터 다음날 오전 8시59분까지는 모든 요청이 거부되겠죠.

## My Two Cents

위에서 간단단 예를 들어서 IAM Policy를 어떻게 하면 잘 생성할 수 있는지에 대해서 소개를 했는데요. IAM Policy에 권한을 부여하는 것이 가장 좋지만 가장 귀찮은 방법은 아무 권한도 부여하지 않고 시작하는 것입니다. AWS Console, SDK, CLI를 이용할 때나 권한문제가 있을 때 AWS는 친절하게도 Error Message를 보여줍니다. 예를 들자면 아래와 같은

    A client error (AccessDenied) occurred when calling the ListObjects operation: Access Denied

이 에러인 즉슨 **ListObject**에 대한 권한이 없어서 명령을 수행할 수 없다는 뜻이죠. 자 그럼 이제 뭘 해야 될까요? IAM Policy에 해당 권한을 추가하면 되겠죠. 無에서 시작해서 Access Denied를 볼때마다 해당 권한을 추가한다면 나에게 꼭 맞은 IAM 정책을 수립할 수 있을 겁니다.

[AWS](#elasticip.md)