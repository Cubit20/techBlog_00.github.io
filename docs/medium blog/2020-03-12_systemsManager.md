---
layout: post
title : AWS Systems Manager와 KMS로 쉽고 안전하게 안전하게 환경변수 관리하기
category : medium blog
order : 3
date: 2020-03-12
---

![systemsManager](./assets/images/mediumBLog/../../../../../assets/images/mediumBlog/20.03.12-systemsManager/systemsManager_01.jpeg)

개발을 하다 보면 Database Credentails, API Key, Secret Key 같이 외부에 노출될 경우 보안적 문제가 발생할 수 있는 요소들을 다루게 되는 경우가 다반사입니다.

Private Git Repository를 사용하니까 평문으로 소스코드 안에 저장해도 괜찮이 않냐라고 생각할수 있지만, 요즘처럼 다수의 개발자들이 Shared Repository에서 작업하는 환경에서는 꼭 그렇다고 볼 수는 없습니다. 통계적으로 보안사고 90% 이상이 Human error에서 시작되기 때문에 개발자 컴퓨터에 저장된 Local Repository가 유출될 경우를 배제할 수 없습니다.

이 문제를 해결하기 위해서 어떤 방법들이 있을까요? 비밀정보를 암호화해서 소스코드안에 저장하고 불러올때 복호화해서 사용하는 방법이나 Secrets Management System을 구축해서 중요 정보들을 불러오게 할수 있습니다. 하지만 이 역시 암호화 키 관리 및 Secrets Management System으로 접속을 위한 정보에 대한 보안이 필요하므로 보안시스템 이용을 위한 보안이, 또 그 보안을 위한 보안.. 끝이없네요

이번 포스팅에서는 AWS System Manager Parameter Store에 MySQL 접속정보를 AWS Key Management Service (KMS)에서 생성한 Key로 암호화 해서 저장하고 애플리케이션에서 환경변수로 불러오는 방법에 대해서 알아보겠습니다. 접속 테스트할 MySQL 서버가 없으실 경우에는 RDS를 통해서 MySQL 서버를 생성해주세요.

우선 암호화에 사용할 Key를 생성하겠습니다.

1. AWS Management Console 에서 좌측 상단에 있는 [Services]를 선택하고 검색창에서 KMS를 검색하거나 [Security, Identity, & Compliance] 밑에 있는 [Key Management Service] 를 선택
2. Custom managed kyes → [Create Key]
3. Key type에는 *Symmetric*(암호화와 복호화에 동일한 Key를 이용)과 *Asymmetric*(저희가 흔히 알고 있는 RSA Public/Private key pair)이 있습니다. Symmetric 으로 선택하고 [NEXT]
4. Alias에 key 이름을, Description에 키에 대한 정보를 입력하고 [Next]
5. Key administrators에 해당 Key의 관리자 권한을 부여할 IAM User나 IAM Role을 지정할 수 있습니다. 만약 현재 IAM User로 로그인되어 있을 경우에는 해당 IAM User를 선택하고 [NEXT]
6. 해당 Key의 사용 권한을 부여할 IAM User나 IAM Role을 지정하거나 만약 다른 AWS 계정에 해당 Key의 사용권하을 부여할 경우에는 타 계정 정보를 입력할 수 있습니다. 여기서는 아무것도 선택하지 않고 [NEXT]
7. 리뷰화면에서 아래와 같은 Key Policy를 확인할 수 있습니다.

        {
            "Id": "key-consolepolicy-3",
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Sid": "Enable IAM User Permissions",
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::ACCOUNT_ID:root"
                    },
                    "Action": "kms:*",
                    "Resource": "*"
                },
                {
                    "Sid": "Allow access for Key Administrators",
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::ACCOUNT_ID:user/John"
                    },
                    "Action": [
                        "kms:Create*",
                        "kms:Describe*",
                        "kms:Enable*",
                        "kms:List*",
                        "kms:Put*",
                        "kms:Update*",
                        "kms:Revoke*",
                        "kms:Disable*",
                        "kms:Get*",
                        "kms:Delete*",
                        "kms:TagResource",
                        "kms:UntagResource",
                        "kms:ScheduleKeyDeletion",
                        "kms:CancelKeyDeletion"
                    ],
                    "Resource": "*"
                }
            ]
        }

    우선 첫번째 블록은 Default로 생성되는 구문으로 IAM Policy를 통해서 해당 Key에 대한 권한을 부여하고 접근할수 있도록 하는 구문입니다. 두번째 구문은 John 이란 IAM 유저에게 KMS 대한 관리 권한을 주는 구문입니다. 자세히 보시면 실제로 Key를 사용하는 Encrypt, Decrypt은 빠져있니다. John 에게 부여된 IAM 정책에 두번째 구문의 권한이 부여되어 있지 않더라도 Key Policy를 통해서 권한을 부여받습니다.

    정리하자면 Key Policy 또는 IAM Policy 혹은 두개 다를 이용해서 KMS Key에 대한 접근관리를 할수 있습니다. 자 그럼 [Finish]을 클릭해서 Key를 생성합니다.

***

이제 그럼 Systems Manager Parameter Store에 Paramter 들을 생성하겠습니다.

1. AWS Management Console 에서 좌측 상단에 있는 [Services] 를 선택하고 검색창에서 Systems Manager를 검색하거나 [Management & Governance] 밑에 있는 [Systems Manager] 를 선택
2. 좌측 패널 Application Management 탭 아래 Parameter Store 클릭 → [Create parameter]
3. 한번에 동일한 Path에 있는 모든 파라미터를 불러올거기 때문에 Name은 Path 형식으로 작성합니다. Fakebook 이라는 서비스에 Test 환경의 DB Credentials을 저장한다고 가정하고 Name을 /FAKEBOOK/TEST/DB_URL 로 지정하겠습니다.
4. Type은 위에서 생성한 KMS Key로 암호화 할거기 때문에 SecureString 으로 선택하고 KMS Key ID를 위에서 생성한 KMS Key를 선택합니다.
5. Value에 접속할 Database endpoint 값을 넣고 [Create parameter]
6. 동일한 방법으로 /FAKEBOOK/TEST/DB_USER, /FAKEBOOK/TEST/DB_PASSWORD 파라미터를 생성합니다.
7. 생성한 파라미터를 클릭해서 상세내용을 보면 Value 가 ******** 로 보이지 않습니다. 옆에 Show 를 클릭하면 볼수있지만 로그인되어 있는 유저가 (1) 해당 파라미터를 볼수 있는 GetParameter 권한과 (2) 해당 파라미터를 암호화한 KMS Key의 대한 Decrypt 권한이 있어야지만 복호화를 통해서 파라미터 값을 볼수 있습니다.

***

이제 EC2를 생성해서 위에서 만든 파라미터를 불러와 보도록 하겠습니다.

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ssm:GetParametersByPath",
                    "ssm:GetParameter"
                ],
                "Resource": "arn:aws:ssm:AWS_REGION:ACCOUNT_ID:parameter/FAKEBOOK/TEST/*"
            }
        ]
    }

1. EC2에 부여할 IAM Role을 생성하고 위와 같은 IAM 정책을 생성하여 부여합니다.
2. KMS Dashboard로 이동해서 앞에서 생성한 KMS Key users에 방금 만든 IAM Role을 추가합니다.
3. EC2를 생성하고 위에서 만든 IAM Role을 부여합니다.
4. MySQL 서버 방화벽 설정에서 해당 EC2로의 접근을 열어줍니다.
5. EC2에 접속해서 데모 어플리케이션을 설치합니다.

        # Python3 및 Git 설치
        $ sudo yum install python3 git -y

        # Demo application repository clone
        $ git clone https://github.com/youngwjung/ssm_ps.git

        # Python library 설치
        $ sudo pip3 install -r ssm_ps/requirements.txt

        # 환경변수 설정 (AWS_REGION를 파라미터가 저장된 리전코드로 변경하세요)
        $ export AWS_DEFAULT_REGION=AWS_REGION SSM_PATH=/FAKEBOOK/TEST/

        # Application 실행
        $ cd ssm_ps && python3 app.py

성공적으로 구성됬을 경우 아래와 같은 메시지를 볼 수 있습니다.

    Connected to xxxxxx.xxxxxx.ap-northeast-2.rds.amazonaws.com

데모 어플리케이션에 대해서 살펴보면 간단히 살펴보면

    # ssm_ps.py

    import boto3

    # 한개의 Parameter 불러오기
    def get_ssmparameter(param_name):
        
        ssm = boto3.client('ssm')

        # Parameter 불러올때 복호화를 통해서 평문으로 불러오기
        response = ssm.get_parameters(
            Names=[
                param_name,
            ],
            WithDecryption=True
        )

    # Response 에서 Parameter 값 리턴
    if not response['InvalidParameters']:
        param_value = response['Parameters'][0]['Value']
    else:
        param_value = 'InvalidParameter'

    return param_value

    # 특정 Path에 있는 모든 파라미터 불러오기
    def get_ssmparameters_by_path(path, next_token=''):

    ssm = boto3.client('ssm')

    # 한번에 10개의 파라미터를 불러올수 있어서 이전 Response에서 NextToken이 있을경우 다음 아이템 불러오기
    if next_token:
        response = ssm.get_parameters_by_path(
            Path=path,
            Recursive=True,
            WithDecryption=True,
            NextToken=next_token
        )
    else:
        response = ssm.get_parameters_by_path(
            Path=path,
            Recursive=True,
            WithDecryption=True
        )

    return response

    # env.py
    import os
    from ssm_ps import get_ssmparameters_by_path

    token = ""

    # NextToken이 없을때까지 파라미터값을 가져오기
    while True:
        response = get_ssmparameters_by_path(os.environ['SSM_PATH'], token)

        for param in response['Parameters']:
            
            # 불러온 파라미터 이름에서 Prefix 제거해서 환경변수로 지정
            os.environ[param['Name'].replace(os.environ['SSM_PATH'], '')] = param['Value']
            
        if 'NextToken' in response:
            token=response['NextToken']

        else:
            break

    # app.py

    import mysql.connector
    import os

    # env.py 모듈을 통해서 Parameter Store에서 불러온 파라미터를 환경변수로 지정
    from env import *

    try:
        db = mysql.connector.connect(
            host = os.environ['DB_URL'],
            user = os.environ['DB_USER'],
            passwd = os.environ['DB_PASSWORD'],
            connection_timeout = 5
        )
        print (f"Connected to {os.environ['DB_URL']}")
    except Exception as e: print(e)

만약 STAG 환경의 DB 접속정보를 불러오고 싶다면 /FAKEBOOK/STAG/ Prefix 아래로 DB_URL, DB_USER, DB_PASSWORD를 저장하고, SSM_PATH 환경변수만 /FAKEBOOK/STAG로 지정하고 애플리케이션 실행시키면 소스코드의 변경없이 다양한 환경의 파라미터를 불러올수 있습니다.

이와같은 방식의 장점은 운영자의 입장에서는 소스코드의 변경없이 Secrets을 추가, 변경, 삭제할수 있고, 개발자의 입장에서는 간단히 환경변수에서 Secrets들을 불러와서 사용할 수 있습니다. Application에는 어떤 암호화 Key도 평문화된 비밀정보도 저장할 필요 없이요.