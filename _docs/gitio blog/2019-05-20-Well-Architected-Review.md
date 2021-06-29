---
layout: post
title: Saltware's Well Architected Review
category : Git Blog
order : 14
date: 2019-05-20
---


## Saltware Well Architected Program

AWS의 Solution Architector(이하 SA) 들이 워크로드 검토를 수행 하는 것을 이제는 Saltware의 전문 SA 들이 함께 수행을 합니다.
저희 Saltware SA들이 Customer들의 AS-IS 시스템에 대한 Architecture 및 Workload를 분석하여 AWS Best Practice 및 common Guide를 전달하고 AWS를 보다더 잘 활용 할 수 있게 도움을 드리는 프로그램입니다. 솔트웨어는 AWS의 공인된 Well Architected Partner 입니다.

![AWS Well-Architected Partner Program Certificate](/assets/images/gitBlog/2019-05-20-Well-Architected-Review/well-Architected-Review_01.png)

Well Architected 프로그램은 아래 5개 Pillar를 중심으로 아키텍처를 검토 및 개선 합니다.

### Well-Architected 5 pillar
![WAR_02](/assets/images/gitBlog/2019-05-20-Well-Architected-Review/well-Architected-Review_02.jpg) 

## WAR Review 진행 절차

- Well Architected Tool 의 5개 Pillar의 모든 항목들을 Customer 와 대화 형식으로 현재 AS-IS에 대한 상태를 약 2~4시간동안 리뷰를 진행 합니다.
- Customer과 Review 진행 후 나온 결과 Report를 가지고 Saltware SA가 개선점등을 찾고 가장 알맞은 아키텍처로 재설계 하여 Improvement Report를 작성합니다.
- 작성 된 Improvement Report를 가지고 Customer에게 프리젠테이션을 진행 하여 AS-IS의 문제점과 TO-BE로 개선 해야할 내용들을 설명드리고, Customer과 함께 개선 해야할 부분에 대한 우선 순위등에 대해 Plan을 설정 합니다
- 개선에 대한 Plan 확립되면, 프리젠 테이션 이후 1개월 내로 개선 작업을 시작하며 Customer  개선에 필요한 부분에 대한 기술 지원도 함께 진행 됩니다.

## AWS Well Architected Tool 소개

AWS Well-Architected Tool은 워크로드 상태를 검토하고 AWS 아키텍처 모범 사례와 비교하는 데 도움이 됩니다. 이 도구는 클라우드 아키텍트가 안전하고, 성능이 뛰어나며, 복원력을 갖춘, 효율적인 애플리케이션 아키텍처를 구축할 수 있도록 개발된 AWS Well-Architected Framework를 기반으로 합니다. AWS 솔루션 아키텍처 팀이 수십만 건의 워크로드 검토를 수행하는 데 사용해 온 이 프레임워크는 Customer 및 파트너가 아키텍처를 평가할 수 있는 일관된 접근 방식을 제공하며, 시간이 지나면서 애플리케이션 요구 사항에 따라 확장되는 디자인을 구현하는 데 도움이 되는 지침을 제공합니다.

## Well Architected Tool 사용 방법

- I open the AWS Well-Architected Tool Console and click Define workload to get started:
![WAR03](/assets/images/gitBlog/2019-05-20-Well-Architected-Review/well-Architected-Review_03.jpg)

- begin by naming and defining my workload. I choose an industry type and an industry, list the regions where I operate, indicate if this is a pre-production or production workload, and optionally enter a list of AWS account IDs to define the span of the workload. Then I click Define workload to move ahead:
![WAR04](/assets/images/gitBlog/2019-05-20-Well-Architected-Review/well-Architected-Review_04.jpg)

- I am ready to get started, so I click Start review:
![WAR05](/assets/images/gitBlog/2019-05-20-Well-Architected-Review/well-Architected-Review_05.jpg)

- The first pillar is Operational Excellence. There are nine questions, each with multiple-choice answers. Helpful resources are displayed on the side:
![WAR06](/assets/images/gitBlog/2019-05-20-Well-Architected-Review/well-Architected-Review_06.jpg)

- I can go through the pillars and questions in order, save and exit, and so forth. After I complete my review, I can consult the improvement plan for my workload:
![WAR07](/assets/images/gitBlog/2019-05-20-Well-Architected-Review/well-Architected-Review_07.jpg)

- I can generate a detailed PDF report that summarizes my answers:
![WAR08](/assets/images/gitBlog/2019-05-20-Well-Architected-Review/well-Architected-Review_08.jpg)

- I can review my list of workloads: <br/>
![WAR09](/assets/images/gitBlog/2019-05-20-Well-Architected-Review/well-Architected-Review_09.jpg)

- And I can see the overall status in the dashboard:
![WAR10](/assets/images/gitBlog/2019-05-20-Well-Architected-Review/well-Architected-Review_10.jpg)

Well Architected Tool 사용법은 AWS Blog에서 발췌 하였으며, 참고 원문 URL은 아래와 같습니다.
https://aws.amazon.com/ko/blogs/aws/new-aws-well-architected-tool-review-workloads-against-best-practices/

솔트웨어의 Well Architected Review 프로그램에 대하여 궁금하신 점이나 의견이 있으시다면 언제든지 솔트웨어 Cloud Conciege 에게 문의 주시기 바랍니다.

고객 여러분의 의견은 저희에게 소중한 자산입니다.

#### AWS Well Architected Review Program 관련 문의 : 

>정태경 팀장 / aws.sales@saltware.co.kr / 02-2025-4933

