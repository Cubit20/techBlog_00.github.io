---
layout: post
title: AWS CaseStudy-코인네스트
category : Git Blog
order : 7
date: 2018-03-30
---

![coinnest_logo](./assets/images/gitBlog/../../../../../assets/images/gitBlog/2018-03-30-CaseStudy-coinnest-en/CaseStudy_coinnest-en_01.png)

#### Customer : 코인네스트 ( Coinnest )


### 고객 요구사항
1. On-premise 대비 TCO 절감 위한 클라우드 서비스 이용
2. 전사적 Digital Transformation 전략에 적합한 Cloud 기반 서비스 필요
3. 대규모 Traffic에 대한 유연한 대처가 가능한 구성 필요
4. 암호화폐거래소 서비스이기 때문에 24시간 Downtime 없이 운영 되어야 함
5. DDoS 공격에 대한 방어와 장애 포인트 없이 서비스가 가능해야 함
6. 실제 운영 서버들에 대한 논리적인 망분리 가 되어야 함
7. 보안강화를 위하여 WAF, IPS 구성 필요
8. 빠른 응답속도 및 웹 서버 부하 분산을 위한 CDN 구성


### 아키텍처
- 암호 화폐 거래소 웹사이트 구축
- 각 product 서버 별 이중화 및 Multi-Az 구성
- CloudFront 구성으로 DDoS 방어 및 웹 가속 효과 이용(Global 이용 시)
- 서비스들은 모두 Private Subnet으로 구성하였고, 실제 Operation을 위한 Bation Host등은 Public Subnet으로 분기하여 구성
- 운영 서버 부하 분산을 위한 이중화 및 Az FailOver를 위한 Multi-AZ 구성 및 AutoScale Group 구성

### 아키텍처 준수사항
- AWS의 Architecting for the Cloud(AWS Best Practice) 의 Design Principles 가이드라인을 준수 하여 고객에게  구성방안 제안 하였습니다.


### 모니터링

![quickview](./assets/images/gitBlog/../../../../../assets/images/gitBlog/2018-03-30-CaseStudy-coinnest-en/CaseStudy_coinnest-en_02.png)

- 솔트웨어(as fitCloud)에서 제공하는 AWS 모니터링 서비스
- CPU, Memory, Disk, Traffic 등 서버 성능 모니터링
- CloudWatch 연동을 통한 다차원적인 리소스 분석
- Multi-Region 리소스 지원
- 각 성능 지표가 설정한 임계치에 도달하면 이메일 알림


### 온프레미스 vs AWS TCO 비교

![coinnest_tco](./assets/images/gitBlog/../../../../../assets/images/gitBlog/2018-03-30-CaseStudy-coinnest-en/CaseStudy_coinnest-en_03.png)

- TCO 분석을 통해 약 32%의 비용 절감이 가능하다는 AWS TCO Calculator 결과를 얻을 수 있었습니다.
- Onpremiss 기준 10대 서버로 비교 하였으며, 실제 AWS구축 시 5대를 기본 AutoScaling을 통해 최대 10대까지 증설되게 구축 진행 하여 더많은 할인 율을 기대 할 수 있음.
- TCO분석에 Seoul Region이 가장 근접한 Region인 Tokyo Region에서 분석하였습니다.
- 3년에 대한 TCO분석 결과 이며, 실제 서울 Region의 On-premiss의 서버 가격에는 차이가 있으나, 실제 데이터센터 운영인력 및 전기료 기타 시설료를 합산하면 해당 결과 값 만큼의 할인율을 도출할 수 있었습니다.
- 특정 시점의 Peak가 해당 물리 서버의 한계를 넘어 서는 시점의 경우 On-premiss는 서버 추가 구매해야 하나 AWS의 경우 AutoScaling을 통해 비용 부담은 덜 수 있습니다.

