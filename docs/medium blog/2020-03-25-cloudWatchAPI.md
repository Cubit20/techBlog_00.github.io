---
layout: post
title : CloudWatch Events로 특정 AWS API 모니터링하기
category : medium blog
order : 5
date: 2020-03-25
---

![그림1](/assets/images/mediumBlog/20.03.25-cloudWatchAPI/cloudWatchAPI_01.jpg)

AWS CloudTrail는 AWS 계정 내에서 수행된 작업들의 API 로그를 기록하고 저장해서 “누가”, “언제", “어떤" 작업을 했는지 확인하고 수집된 로그들을 토대로 보안 분석, 운영 감사, 규정 위반 감사, 비정상적 활동 탐지등을 손쉽게 할수 있는 서비스입니다.

뿐만 아니라 CloudWatch Events와의 연동을 통해서 Trail에 저장되는 API 이벤트 로그를 실시간으로 탐지후 특정 API가 호출될 경우에 AWS SNS를 통해서 알람을 보낸다거나 AWS Lambda를 통해서 미리 짜여진 작업을 실행시킬수도 있습니다.

이번 포스팅에서는 몇가지 사례를 통해서 AWS API를 실시간으로 모니터링 하는 방법에 대해서 알아보겠습니다.

우선 CloudTrail을 생성합니다.

1. AWS Management Console에서 좌측 상단에 있는 [Services] 를 선택하고 검색창에서 CloudTrail를 검색하거나 [Management & Governance] 밑에 있는 [CloudTrail] 를 선택
2. [Create trail] → Trail name = 지정할 이름 입력, S3 Bucket = Trail를 저정한 S3 Bucket 지정→ [Create]

***

첫번째로 적용할 내용은 CloudTrail Logging이 정지될 경우 다시 자동으로 Logging 활성화하는 로직입니다. Logging이 비활성화될 경우 Trail이 S3로 저장되지 않습니다. 제가 해커라면 우선 Trail Logging을 끄겠죠? 흔적을 남기지 않기 위해서..

Lambda Function 생성

1. AWS Management Console에서 좌측 상단에 있는 [Services] 를 선택하고 검색창에서 Lambda를 검색하거나 [Compute] 밑에 있는 [Lambda] 를 선택
2. Lambda Dashboard에서 [Create function] 클릭후, Function name = enable_trail, Runtime = Python 3.8, [Create function] 클릭
3. 좌측 하단에 있는 [Execution role] 에서 View the enable_trail-role-xxxx on the IAM console 를 선택
4. [Permissions] 섹션 오른쪽에 있는 [➕ Add inline policy] 클릭후, Service = CloudTrail, Actions = StartLogging, Resources 탭에 있는 [Add ARN] 클릭 → Region = ap-northeast-2, Trail name = 위에서 생성한 Trail 이름 입력 → [Add]
5. [Review Policy] → Name = allow-lambda-start-trail-logging → [Create Policy]
6. 아래 코드블록을 Lambda에 복사 후, [Save] 클릭
        import json
        import boto3
        def lambda_handler(event, context):
            trail_name = event['detail']['requestParameters']['name']
            
            client = boto3.client('cloudtrail')
            
            response = client.start_logging(
                Name=trail_name
            )

CloudWatch Events Rule 생성

1. AWS Management Console에서 좌측 상단에 있는 [Services] 를 선택하고 검색창에서 CloudWatch를 검색하거나 [Management & Governance] 밑에 있는 [CloudWatch] 를 선택
2. CloudWatch Dashboard에서 [Events] 섹션 아래에 있는 [Rules] → [Create rule]
3. 왼쪽 Event Source에서 🔘 Event Pattern 선택 , Service Name = CloudTrail, Event Type = AWS API Call via CloudTrail, 🔘 Specific operation(s)= StopLogging
4. 오른쪽 Targets에서 [➕ Add target] → Lambda function → Function = enable_trail → [Configure details]
5. Name = enable_trail, State = ✅ Enabled → [Create rule]

CloudTrail dashboard로 이동해서 Trail Logging은 비활성화 합니다. 몇초후에 페이지를 다시 불러오면 Trail Logging이 활성화 되 있는것을 확인할수 있습니다.

***

다음으로는 EC2 인스턴스가 시작, 정지, 종료될때 SNS를 통해서 알림을 받아보겠습니다. AWS를 사용하다 보면 종종 갑자기 못 보던 인스턴스가 생성되어 있거나 실행중이던 인스턴스가 정지되어 있어서 흠칫할때가 있는데요. 실시간으로 알림을 받아보면 좋겠죠?

SNS 토픽생성

1. AWS Management Console에서 좌측 상단에 있는 [Services] 를 선택하고 검색창에서 SNS를 검색하거나 [Application Integration] 밑에 있는 [Simple Notification Service] 를 선택
2. 왼쪽 패널에서 Topics 선택 → [Create topic] → Name = 토픽 이름 입력 → [Create topic]
3. [Create subscription] → Protocol = Email, Endpoint = 알림 받을 이메일 주소 입력 → [Create subscription]
4. 위에서 입력한 이메일로 전송된 인증메일에서 Confirm subscription 클릭   

![그림2](/assets/images/mediumBlog/20.03.25-cloudWatchAPI/cloudWatchAPI_02.png)

CloudWatch Events Rule 생성

1. Service Name = EC2, Event Type = EC2 Instance State-change Notification, 🔘 Specific state(s)= running, stopped, terminated
2. 오른쪽 Targets에서 [➕ Add target] → SNS topic → Topic = 위에서 생성한 토픽 이름 → [Configure details]
3. Name = ec2_state_change, State = ✅ Enabled → [Create rule]

이제 EC2 인스턴스를 한개 생성해보겠습니다. 생성 완료후에 SNS topic에 등록한 이메일로 아래와 같은 내용의 알림이 수신됬음을 확인할 수 있습니다.

    {
        "version":"0",
        "id":"6da1e8c2-bbae-8877-f185-6c7bc6d9dcad",
        "detail-type":"EC2 Instance State-change Notification",
        "source":"aws.ec2",
        "account":"1234567890",
        "time":"2020-03-25T05:42:48Z",
        "region":"ap-northeast-2",
        "resources":[
            "arn:aws:ec2:ap-northeast-2:1234567890:instance/i-05e8fb4ee552d3511"
        ],
        "detail":{
            "instance-id":"i-05e8fb4ee552d3511",
            "state":"running"
        }
    }

이메일 말고도 Slack과 연동해서 특정 이벤트에 대한 내용을 실시간으로 받아 볼 수도 있습니다. 관련 내용은 해당 [링크](https://github.com/youngwjung/aws-root-account-best-practice#root-%EC%9C%A0%EC%A0%80-%EB%A1%9C%EA%B7%B8%EC%9D%B8-slack-%EC%95%8C%EB%9E%8C-%EC%84%A4%EC%A0%95)에서 확인할 수 있습니다.
