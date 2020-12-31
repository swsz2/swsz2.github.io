---
layout: post 
title: '나의 프로젝트 패키지 구조 in Spring Boot'
subtitle: 'swsz2는 어떻게 패키지 구조를 세울까'
categories: devlog 
tags: java
comments: true
---
## 패키지 구조에 대한 고찰
이런 대화를 나눈 적이 있습니다.
<pre>
 ???  : swsz2님! 패키지 구조는 어떻게 잡아야 하나요?   
swsz2 : 프로젝트 성격 등 여러 요소를 봐야겠죠?
 ???  : 어느 정도 고정된 구조는 없나요?  
swsz2 : 전 세계 공용 구조는 없는데 오픈 소스에서 자주 쓰는 구조는 있죠.
 ???  : 그럼 swsz2님은 본인만의 구조가 있나요?
swsz2 : 있긴 한데...
 ???  : 보여주세요!
swsz2 : 네...
</pre>

대답하면서 드는 생각은 이러했습니다.
<pre>
내가 프로젝트 개발 리더가 된다면 어떻게 패키지 구조를 짜야 할까?
나의 패키지 구조를 다른 사람에게 이해시킬 수 있을까? 
</pre>

그래서 쓰게 된 이번 포스팅은 **나의 프로젝트 패키지 구조 In Spring Boot** 입니다.

## 나의 패키지 구조 소개
![my-package-structure-in-spring-boot](/assets/img/post/my-package-structure-in-spring-boot.PNG)

**녹차의 처음부터 끝까지**라는 주제로 패키지를 구성해봤습니다.   
→ Spring Data JPA를 사용한다고 가정
<br>

### 모듈별로 구분한다.
**녹차의 처음부터 끝까지** 프로젝트는 아래 모듈의 조합으로 이루어집니다.
<pre>
- field : 밭에서 녹찻잎을 수확하기 위한 패키지
- factory : 수확한 녹찻잎을 가공하기 위한 패키지
- shop : 가공된 녹찻잎을 이용해 음료를 판매하기 위한 패키지
</pre>
[TMI] 하나의 시스템은 여러 기능적 구성 요소인 모듈의 조합으로 이루어집니다.

#### 왜?
먼저 아래와 같은 장점이 있습니다.
<pre>
- 전체 프로젝트의 구조를 파악하기 쉽다
- 팀 단위 작업을 하기 쉽다
- 유지 보수가 용이하다
</pre>
물론 단점도 있습니다.
<pre>
- 패키지가 모듈 수의 배로 커진다.
</pre>
단점에서 오는 마이너스보다 장점에서 오는 플러스가 크다고 생각해서 모듈별로 구분하는 것을 선호합니다.  

### 하위 패키지부터는 기능과 역할을 기준으로 분리한다
#### Common
<pre>
- 어느 위치에서나 호출할 수 있는 유틸, 헬퍼 클래스를 정의하는 패키지
</pre>
#### Component
<pre>
- 독립적인 기능을 수행하는 작은 단위의 모듈을 정의하는 패키지
  - Bar : 마실 것을 만드는 곳
  - Hall : 만들어진 차를 마시는 곳
  - Kitchen : 디저트를 만드는 곳
</pre>
#### Constants
<pre>
- Enum 클래스(상수)를 정의하는 패키지
  - GreenTea : 녹차의 종류 (반야차, 설록차, 죽로차 등)
  - Latte : 라테의 종류 (카페 라테, 바닐라 라테, 녹차 라테 등) 
</pre>
#### Controller
<pre>
- 사용자의 요청을 받는 클래스를 정의하는 패키지
  - Counter : 손님의 요청을 판단하여 Bar, Kitchen, Hall에 처리 요청
</pre>
#### Entity
<pre>
- 데이터베이스의 테이블에 매핑되는 클래스를 정의하는 패키지
  - Ingredient : 식재료 정보와 매핑되는 클래스
  - Order : 주문 정보와 매핑되는 클래스
  - Revenue : 이윤 기록과 매핑되는 클래스
</pre>
#### Model
<pre>
- 비즈니스 로직에 사용되는 데이터를 운반 또는 저장하기 위한 클래스를 정의하는 패키지
  - Bill : Order 후 처리 결과를 손님에게 전달하기 위한 클래스
  - Cake : Kitchen 에서 만들어진 조리 결과를 손님에게 전달하기 위한 클래스
</pre>
#### Repository
<pre>
- 데이터베이스의 테이블에 접근하기 위한 인터페이스를 정의하는 패키지
  - IngredientRepository : 식재료 정보 테이블에 접근
  - OrderRepository : 주문 정보 테이블에 접근
  - RevenueRepository : 이윤 기록 테이블에 접근
</pre>
#### Service
<pre>
- 비즈니스 로직을 수행하는 클래스, 인터페이스를 정의하는 패키지
  - CleaningService : 청소 로직 관리
- 동작하는 행위가 동일하지만 대상이 다른 경우 인터페이스를 생성한다.
  - OrderService : 주문을 위한 로직, 인터페이스
  - BeverageOrderService : 음료 주문을 위한 로직, 클래스
  - DessertOrderService : 디저트 주문을 위한 로직, 클래스
</pre>
#### Worker
<pre>
- 비즈니스 로직에서 특정 행위를 하는 클래스, 인터페이스를 정의하는 패키지
  - Waiter : 주문을 받고 완료된 음식을 운반하는 클래스
</pre>

## 마무리
저는 위와 같은 규칙으로 패키지 구조를 세우지만 사실 패키지 구조에 정답은 없습니다.  
왜냐면 동료들과 소통이 가능하고 협업하는 데 큰 문제만 없으면 된다고 생각하기 때문입니다.  

관련된 재미있는 스택오버플로우 주제가 있으니 구경해보시면 좋을 듯 합니다.  
[are-there-best-practices-for-java-package-organization?](https://stackoverflow.com/questions/3226282/are-there-best-practices-for-java-package-organization){: target="_blank"}
<br>
P.S  
패키지 구조에 대한 의견이나 질문 등을 댓글로 남겨주시면 답변해드리겠습니다.  
두서없는 글을 읽어주셔서 감사합니다.  
