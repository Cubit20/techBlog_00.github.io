---
layout: post
title: AWS CaseStudy-미래에셋자산운용
category : Git Blog
order : 9
date: 2017-07-20
---

![miraeasset](/assets/images/gitBlog/2018-03-30-CaseStudy-miraeasset-ko/CaseStudy-miraeasset-ko_01.jpg)https://investments.miraeasset.com

#### Customer : 미래에셋자산운용 ( Mirae Asset Global Investment ) 

AWS 고객사례 : https://aws.amazon.com/ko/solutions/case-studies/miraeasset/

### 고객 요구사항
1. 마케팅을 위해 글로벌 웹사이트에 의존하고 있어 글로벌 클라우드 서비스 고려
2. 해외 접속자들의 속도 개선 필요
3. Onpremise 대비 TCO 절감 위한 클라우드 서비스 (기존 노후화되고 불안정한 시스템 대체)
4. 전사적 Digital Transformation 전략에 적합한 Cloud 기반 서비스 필요


### 아키텍처
- 웹 서비스는 일반적인 3 tier 구조
- 각 티어별, 서비스별 이중화 및 Multi-Az 구성
- ELB, AutoScaling을 통해 유연한 인프라 구성  
- CloudFront를 통해 일반적인 DDoS 방어 및 웹 가속 효과
- CloudFront에 ACM 서비스를 연동함으로써 SSL 인증서 무상 사용
- 글로벌 서비스 및 GSLB를 위해 Route53 구성


### 아키텍처 준수사항
- AWS의 Architecting for the Cloud(AWS Best Practice) 의 Design Principles 가이드라인을 준수 하여 고객에게  구성방안 제안 하였습니다.


### 모니터링
![mirae_monitoring](/assets/images/gitBlog/2018-03-30-CaseStudy-miraeasset-ko/CaseStudy-miraeasset-ko_02.jpg)
- 솔트웨어(as fitCloud)에서 제공하는 AWS 모니터링 서비스
- CPU, Memory, Disk, Traffic 등 서버 성능 모니터링
- CloudWatch 연동을 통한 다차원적인 리소스 분석
- Multi-Region 리소스 지원
- 각 성능 지표가 설정한 임계치에 도달하면 이메일 알림


### 온프레미스 vs AWS TCO 비교
![mirae_tco](/assets/images/gitBlog/2018-03-30-CaseStudy-miraeasset-ko/CaseStudy-miraeasset-ko_03.jpg)
- TCO 분석을 통해 약 91%의 비용 절감이 가능하다는 AWS TCO Calculator 결과를 얻을수 있었습니다.
- Tokyo Region과 Singapore Region에서 분석
- 3년에 대한 TCO분석 결과 이며, 실제 서울 Region의 On-premiss의 서버 가격에는 차이가 있으나, 실제 데이터센터 운영인력 및 전기료 기타 시설료를 합산하면 해당 결과 값 만큼의 할인율을 도출할 수 있었습니다.
- 특정 시점의 Peak가 해당 물리 서버의 한계를 넘어 서는 시점의 경우 On-premiss는 서버 추가 구매, AWS의 경우 AutoScaling을 통해 추가 증설된 Instance의 초단위 과금의 차이도 있어 더 많은 할인율을 기대 할 수 있습니다.


