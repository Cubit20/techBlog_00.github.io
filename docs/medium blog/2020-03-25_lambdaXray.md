---
layout: post
title : AWS Lambda에 X-Ray 적용하기
category : medium blog
order : 6
date: 2020-03-25
---

![그림1](/assets/images/mediumBlog/20.03.25-lambdaXray/lambdaXray_01.jpg)

AWS Lambda는 서버없이(Serverless) 애플리케이션 코드를 실행할 수 있어서 Microservice Architecture을 구성하거나, AWS의 여러 서비스들과 연계해서 Event-driven Architecture를 구현할 때 많이 사용되는데요.

아주 간단한 코드라면 Lambda 로그를 통해서 에러를 디버깅하거나 Performance를 확인할 수 있지만 코드가 복잡해지거나 Step Function을 통해서 여러개의 Lambda function이 동시다발적으로 실행된다면 디버깅이 쉽지 않을 수도 있습니다. 이때 AWS X-Ray를 활용하면 좀 더 쉽게 Lambda에서 실행되는 코드를 Tracing 할 수 있습니다.

API Gateway + Lambda + DynamoDB로 DynamoDB 테이블을 스캔하는 API를 만들고 각 구간별 실행시간을 X-Ray로 확인하는 방법에 대해서 알아 보겠습니다.

***

우선 DyanmDB 테이블을 생성합니다.

1. AWS Management Console에서 좌측 상단에 있는 [Services]를 선택하고 검색창에서 DynamoDB를 검색하거나 [Database] 밑에 있는 [DynamoDB]를 선택
2. DynamoDB Dashboard 에서 [Create table] 클릭 → Table name = x-ray-demo, Primary key = id, Use default settings = ✅ → [Create]
3. [Create item]으로 몇개의 아이템을 생성합니다.

![그림2](/assets/images/mediumBlog/20.03.25-lambdaXray/lambdaXray-02.png)

***

다음으로는 Lambda function을 생성하겠습니다.

1. AWS Management Console에서 좌측 상단에 있는 [Services]를 선택하고 검색창에서 Lambda를 검색하거나 [Compute] 밑에 있는 [Lambda]를 선택
2. Lambda Dashboard에서 [Create function] 클릭 → Function name = x-ray-demo, Runtime = Node.js 12.x → [Create function]
3. 좌측 하단에 있는 [Execution role] 에서 **View the x-ray-demo-role-xxx on the IAM console** 를 선택하고 위에서 생성한 DynamoDB에 대한 읽기 권한을 부여
4. 아래 코드블록을 Function Code에 복사 후 [Save] 클릭

        const AWS = require('aws-sdk')

        exports.handler =  function(event, context, callback) {

        const dynamodb = new AWS.DynamoDB()
            var params = {
                TableName: 'x-ray-demo'
            }    
            var body
            dynamodb.scan(params, function(err, data) {
                
                if (err) {
                    console.log("error")
                    console.log(err, err.stack)
                    callback(err)
                } else {
                    const response = {
                        statusCode: 200,
                        body: JSON.stringify(data.Items)
                    }
                    callback(null, response)
                }
            })
        }

생성된 Lambda Function을 API Gateway와 연동하겠습니다.

[Designer] 탭에서 [➕ Add Trigger] 클릭 → Select a trigger = API Gateway, API = Create an API, API type = REST API, Security = Open → [Add]

생성된 API endpoint를 클릭하면 아래와 같이 DynamoDB 테이블의 레코드가 표시됩니다.

        // 20200325203114
        // https://dy2iuwz3xl.execute-api.ap-northeast-2.amazonaws.com/default/x-ray-lab

        [
            {
                "id":{
                    "S": "2"
                },
                "name":{
                    "S": "Jane"
                }
            },
            {
                "id":{
                    "S": "1"
                },
                "name":{
                    "S": "Jhon"
                }
            }
        ]

이제 X-Ray tracing을 켜보겠습니다. Lambda 설정창에서 Active tracing을 활성화하고 [Save]

![그림3](/assets/images/mediumBlog/20.03.25-lambdaXray/lambdaXray-03.png)

API Endpoint로 여러번 접속한 다음에 [View traces in X-Ray]를 통해서 X-Ray Dashboard로 이동해서 Service Map하고 Traces를 확인합니다.
   
![그림4](/assets/images/mediumblog/20.03.25-lambdaXray/lambdaXray-04.png)

![그림5](/assets/images/mediumBlog/20.03.25-lambdaXray/lambdaXray-05.png)

음... DynamoDB에 대한 정보는 따로 보여지지 않습니다. Lambda에서 기본적으로 제공하는 X-Ray Tracing은 해당 Lambda에 대한 정보만 보여지므로 소스코드에 X-Ray Tracing 부분을 설정해줘야 합니다.

***

AWS X-Ray SDK for Node.js가 필요한데요. Lambda Layer를 통해서 구현하겠습니다.

1. Lambda Dashboard에서 Layers 클릭 → [Create layer] → Name = nodejs-xray-sdk, 🔘 Upload a file from Amazon S3, Amazon S3 link URL = `https://saltware-aws-lab.s3.ap-northeast-2.amazonaws.com/msa/node-xray-sdk.zip` Compatible runtimes = Node.js 10.x, Node.js 12.x → [Create]
2. 위에서 생성한 Lambda function으로 가서 Layers 선택 → [Add a layer] → Name = nodejs-xray-sdk, Version =1 → [Add]
3. Lambda 소스 코드를 아래와 같이 수정하고 [Save]

        const AWSXRay = require ('aws-xray-sdk');
        const AWS = AWSXRay.capture(require('aws-sdk'));

        exports.handler = function(event, context, callback) {
        ...
        ...

    node-xray-sdk.zip 파일은 아래와 같은 방식으로 생성할수 있습니다.

        mkdir nodejs
        cd nodejs
        npm install aws-xray-sdk
        zip node-xray-sdk.zip ../nodejs -r

    이제 API Endpoint를 몇번 호출하고 X-Ray Dashboard로 가서 변경된 사항을 확인해봅니다.

    ![그림6](/assets/images/mediumBlog/20.03.25-lambdaXray/lambdaXray-06.png)

    이번에는 DynamoDB 부분에 대한 내용도 상세하게 확인할 수 있습니다. 만약에 MySQL 데이터베이스를 사용한다면 아래와 같이 mysql 클라이언트를 정의하고 DB에 연결할 경우에 Query에 대한 Tracing도 가능합니다.

        const mysql = AWSXRay.captureMySQL(require('mysql'))
    
***

해당 포스팅은 Node.js 기준으로 생성됬지만 AWS X-Ray는 Java, Python등 다른 언어도 지원합니다. Lambda 뿐만 아니라 일반적인 어플리케이션에서 적용 가능하니 AWS를 이용해서 개발하신다면 한번씩 사용해 보세요.