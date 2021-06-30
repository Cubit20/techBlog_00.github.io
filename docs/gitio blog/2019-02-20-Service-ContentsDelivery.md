---
layout: post
title: 빠르고 안전한 콘텐츠 전송
category : Git Blog
order : 13
date: 2019-02-20
---

글로벌 CDN 서비스인 Amazon CloudFront 를 클라우드의 전면에 배치하여 빠르고 안전하게 콘텐츠를 전송하고 외부로 부터의 침해를 방어 할 수 있습니다.

### 콘텐츠 전송 네트워크 서비스 ( CDN )
CDN 이라고 불리우는 콘텐트 전송 네트워크 ( Contents Delivery Network )는 서비스 하고자 하는 콘텐츠를 빠르게 전송하기 위하여 사용자와 가까운 지역에 콘텐츠를 캐싱하여 서비스 하도록 하여 주어 서버의 트래픽 부하를 줄여 주고, 엔드유저의 웹 경험을 빠르게 제공하여 주는 서비스 입니다.

### Amazon CloudFront - 빠르고 안전한 AWS의 CDN 서비스

> Layer 3,4 DDoS 에 대한 무료 방어 서비스
><br> HTTP, HTTPS 구분 없이 동일 가격
><br> AWS 오리진 데이터 전송 비용 무료

Amazon CloudFront는 전세계의 AWS 백본망과 전용 고속 엣지 로케이션을 통하여 콘텐츠를 빠르고 안전하게 전송하는 CDN 서비스입니다.

![AmazonCF](./assets/images/gitBlog/../../../../../assets/images/gitBlog/2019-02-20-Service-ContentsDelivery/Service-ContentsDelivery_01.png)

- 빠른 속도와 글로벌한 규모
- 엣지보안
- 고도로 프로그래밍 가능 ( Lambda@edge )
- AWS와 긴밀한 통합

일반적인 CDN 이 정적인 콘텐츠를 캐싱하여 전송하는 서비스임에 반하여 AWS CloudFront는 동적 콘텐츠에 대한 글로벌 가속 서비스, DDoS 에 대한 완화서비스를 별도의 추가 비용 없이 함께 제공합니다. 또한 Lamda@edge 와 같은 프로그래밍을 CDN에 적용하여 요청헤더에 보안 적용을 추가로 구성할 수 있으며, 특정 지역에 대한 트래픽 허용 및 차단기능을 콘솔을 통해 쉽게 구성할 수 도 있습니다.

### Amazon CloudFront 의 장점
Amazon CloudFront를 사용한다면 빠르고 안전한 콘텐츠 전송 뿐만 아니라, 높아지는 클라우드상의 데이터 전송 비용도 절감 할 수 있습니다.

![image](./assets/images/gitBlog/../../../../../assets/images/gitBlog/2019-02-20-Service-ContentsDelivery/Service-ContentsDelivery_02.png)

- Super POP Architecture : AWS 클라우드 구축 / 운영 Know-How가 담긴 고성능/대용량 아키텍처
- 한국내 최대 Capacity / 가장 빠르게 성장하는 글로벌 CDN 서비스
- Single-Service : (캐싱, 다이나믹 가속, HTTPS, AWS Shield Standard 등) 동일 가격 체계로 제공
- AWS Backbone 전용망 : Edge <-> Origin 가속 서비스로 활용
- 인라인 DDoS 방어 : CloudFront이용만으로 Shield Standard 무료 적용으로 Layer3&4 DDoS 방어
- 최상의 Routing : 지연시간 기준으로 최상의 라우팅 룰 적용.


### AWS의 EC2와 S3 사용시 오리진 전송비용 무료

![cf-cost](./assets/images/gitBlog/../../../../../assets/images/gitBlog/2019-02-20-Service-ContentsDelivery/Service-ContentsDelivery_03.png)


1. CloudFront를 사용하실 경우 Default로 AWS 오리진(EC2, ELB, S3) Data-Out 비용이 면제 됩니다.
2. CloudFront는 캐싱은 물론 미디어 전송, API와 같은 동적 콘텐츠 가속, HTTPS 등을 동일 가격 체계로 제공하여 기존 CDN 대비 월등히 가성비 높은 서비스를 제공합니다.
3. AWS Shield와 AWS WAF를 통한 서비스 연계로, DDoS 공격 및 웹해킹 위협으로부터 안전한 서비스 운영이 가능합니다.
4. 100Gbps 다중회선의 AWS 오리진 to 에지 전용 백본망을 이용하여 첫번째 다운로드 및 캐싱이 되지 않는 동적 콘텐츠에 대해 높은 성능 및 안정성을 제공합니다.
5. EC2, S3, ELB는 물론 Route53, ACM, CloudWatch 등과 같이 AWS의 다양한 서비스들과 연동 및 통합 관리됩니다.

AWS의 대표적인 서비스인 Amazon EC2 와 Amazon S3 를 Amazon CloudFront 의 Origin 으로 활용하는 경우에는 전송비용이 발생하지 않습니다. 
또한 Amazon CloudFront 의 경우 DDoS 를 방어하기 위한 Amazon Standard Shield 서비스가 기본으로 활성화 되어 있어 빠르고 안전한 전송을 함께 서비스 받게 됩니다.

## Saltware의 FitCloud Contents Delivery 서비스
>솔트웨어는 AWS의 인증된 Amazon CloudFront Service 파트너사입니다.  
><br>https://aws.amazon.com/ko/partners/find/partnerdetails/?n=Saltware&id=001E000000xHZ4MIAW

솔트웨어를 통하여 Amazon CloufFront 를 사용하시면 다음과 같은 잇점이 있습니다.
- 대량의 콘텐츠 전송에 대한 할인 ( 전송량 약정시 )
- 데이터 전송비용 절감을 위한 아키텍처 진단
- 콘텐츠 전송 서비스에 대한 전문적인 기술지원
- DDoS , 네트워크 침해 방지를 위한 각종 보안 구성 및 솔루션 제언

솔트웨어의 Amazon CloudFront 서비스에 대하여 궁금하신 점이나 의견이 있으시다면 언제든지 솔트웨어 Cloud Conciege 에게 문의 주시기 바랍니다.

고객 여러분의 의견은 저희에게 소중한 자산입니다.

#### AWS CloudFront 서비스 관련 문의 : 

>정태경 팀장 / aws.sales@saltware.co.kr / 02-2025-4933