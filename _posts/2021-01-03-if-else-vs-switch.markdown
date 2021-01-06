---
layout: post
title: 'if-else vs switch' 
subtitle: '[주의] 당신이 원하는 답이 아닐 수 있습니다.' 
categories: devlog 
tags: java 
comments: true
---
## 효율적?
<pre>
 ???  : swsz2님 여기선 switch보다 if-else가 더 효율적이지 않나요?   
swsz2 : 효율적이라면... 수행 속도가 빠르다는 건가요?
 ???  : 네!  
swsz2 : if-else가 더 빠를 수도 있겠네요. 
 ???  : 그럼 if-else문으로 바꿔도 되나요?
swsz2 : 그건...
 ???  : 왜요? 빠르면 좋은 거 아닌가요?
swsz2 : 그게 말이죠... 우선 테스트부터 해볼까요?
</pre>


## 빠르면 좋은 거 아닌가요?

저는 코드에 대한 질문과 의견을 받았을 때 그것들에 대한 존중이 필요하다고 생각합니다.  
그래서 지난 포스팅에서 다뤘던 **녹차의 처음부터 끝까지** 프로젝트를 활용해서 검증과 비교를 해보기로 했습니다.

<pre>
1. 간단한 코드를 통한 if-else vs switch 퍼포먼스 테스트
2. 가독성
</pre>

### 간단한 코드를 통한 if-else vs switch 퍼포먼스 테스트

카페에서 커피 또는 디저트를 제공하기 위한 과정은 다음과 같습니다.
<pre>
1. 손님으로 부터 주문을 받는다.
2. 주문에 대한 내용을 Bar 또는 Kitchen에 전달한다.
3. 주문에 맞는 음료 또는 디저트를 만든다.
4. 손님에게 제공한다.
</pre>
*java 11에서 테스트*

```java
for (int orderNumber = 1; orderNumber < 1 * 10; orderNumber++) { // 주문 번호 생성
    Order order = new Order();                                   // 주문 생성
    order.setMenu(orderNumber);                                  // 주문 번호에 따른 메뉴 설정
    bar.brew(order);                                             // bar에서 주문 전달
    waiter.serve(order);                                         // 주문에 따른 서빙
}
```
```java
@ToString
public class Coffee extends Beverage {
    @Getter
    @Setter
    private CoffeeType type;
    @Getter
    @Setter
    private Size size;
    @Getter
    @Setter
    private boolean served;

    public Coffee(CoffeeType coffeeType) {
        type = coffeeType;
        size = Size.TALL;
    }
}
```

```java
@ToString
public enum CoffeeType {

    ESPRESSO("espresso", 2000),
    AMERICANO("americano", 2000),
    CAPPUCCINO("capuccino", 2500),
    CAFE_LATTE("cafe latte", 2500),
    CAFE_MOCHA("cafe mocha", 2500),
    CON_PANNA("con panna", 3000),
    AFFOGATO("affogato", 3000),
    CAFE_ROYAL("cafe royal", 3000),
    DUTCH("dutch", 3000),
    EMPTY("empty",0);

    private String name;
    @Getter
    private int price;

    CoffeeType(String name, int price) {
        this.name = name;
        this.price = price;
    }

    public static CoffeeType findByOrder(Order order) {
        int menu = order.getMenu() % 9;
        if (menu == 0) {
            return ESPRESSO;
        } else if (menu == 1) {
            return AMERICANO;
        } else if (menu == 2) {
            return CAPPUCCINO;
        } else if (menu == 3) {
            return CAFE_LATTE;
        } else if (menu == 4) {
            return CAFE_MOCHA;
        } else if (menu == 5) {
            return CON_PANNA;
        } else if (menu == 6) {
            return AFFOGATO;
        } else if (menu == 7) {
            return CAFE_ROYAL;
        } else if (menu == 8) {
            return DUTCH;
        } else {
            return EMPTY;
        }
    }

    public static CoffeeType findByOrder(Order order) {
        switch (order.getMenu() % 9) {
            case 0:
                return ESPRESSO;
            case 1:
                return AMERICANO;
            case 2:
                return CAPPUCCINO;
            case 3:
                return CAFE_LATTE;
            case 4:
                return CAFE_MOCHA;
            case 5:
                return CON_PANNA;
            case 6:
                return AFFOGATO;
            case 7:
                return CAFE_ROYAL;
            case 8:
                return DUTCH;
            default:
                return EMPTY;
        }
    }
}
```
*그대로 복붙하시면 컴파일 에러가 발생합니다. 하나의 findByOrder 함수명을 변경해 주시거나 제거해야 합니다.*  

### 수행 결과
<pre>
Order(id=602f42b8-aaa2-41a9-8727-0465715891b3, menu=1, price=2000, beverage=Coffee(type=CoffeeType.AMERICANO(name=americano, price=2000), size=TALL, served=true), dessert=null, served=true)
Order(id=88b308fd-7e75-4eae-b3fd-ae4ed8d1f206, menu=2, price=2500, beverage=Coffee(type=CoffeeType.CAPPUCCINO(name=capuccino, price=2500), size=TALL, served=true), dessert=null, served=true)
Order(id=7d051a12-445b-43fc-8470-3530c3b41dcc, menu=3, price=2500, beverage=Coffee(type=CoffeeType.CAFE_LATTE(name=cafe latte, price=2500), size=TALL, served=true), dessert=null, served=true)
Order(id=d1964aed-7c8e-4f0b-9f28-3edf229d7c0e, menu=4, price=2500, beverage=Coffee(type=CoffeeType.CAFE_MOCHA(name=cafe mocha, price=2500), size=TALL, served=true), dessert=null, served=true)
Order(id=798a4160-d1d7-4027-9086-0ae96c84e523, menu=5, price=3000, beverage=Coffee(type=CoffeeType.CON_PANNA(name=con panna, price=3000), size=TALL, served=true), dessert=null, served=true)
Order(id=d8bd8de7-f27b-4595-950e-b6731ff2eea8, menu=6, price=3000, beverage=Coffee(type=CoffeeType.AFFOGATO(name=affogato, price=3000), size=TALL, served=true), dessert=null, served=true)
Order(id=ee46afca-d840-4039-a728-c07718187246, menu=7, price=3000, beverage=Coffee(type=CoffeeType.CAFE_ROYAL(name=cafe royal, price=3000), size=TALL, served=true), dessert=null, served=true)
Order(id=99ed9c40-1fad-4651-bcf3-3a6d0cf6ecfb, menu=8, price=3000, beverage=Coffee(type=CoffeeType.DUTCH(name=dutch, price=3000), size=TALL, served=true), dessert=null, served=true)
Order(id=ecaff310-c115-4cd2-b9fa-d9416091ae80, menu=9, price=2000, beverage=Coffee(type=CoffeeType.ESPRESSO(name=espresso, price=2000), size=TALL, served=true), dessert=null, served=true) 
</pre>

### 반복에 따른 테스트 결과
![detail-for-if-switch-statistics](/assets/img/post/detail-for-if-switch-statistics.PNG)
<pre>
1억 번의 주문을 switch는 약 300초, if-else는 약 330초 동안 처리함
-> int에 대한 처리를 1억 번 반복했을 때 switch가 if-else보다 약 10% 정도 더 빨리 처리함
-> 다른 경우에는 if-else가 빠를 수도 있음 
</pre>

## 그러나 여기서 중요한 건 속도가 아니라 가독성입니다.
물론 **속도**도 중요합니다.  
그러나 더 중요한 것은 **가독성**입니다.

### 왜?
저는 **if-else, switch에 따른 속도 차이로 퍼포먼스를 챙기는 것**보다     
**코드에 대한 가독성 그리고 유지 보수하기 편한 이점을 챙기는 것**이 더 좋다고 생각합니다.      
즉 **상황에 따른 if-else, switch 선택이 중요하다**고 생각합니다.  

#### if-else를 사용하는 경우
<pre>
1. true, false와 같이 두 가지 경우로만 처리가 가능한 경우
2. 범위에 대한 처리를 하는 경우 
-> if-else는 범위에 대한 처리에 매우 유용합니다.
</pre>

```java
public void printStatus (boolean status) {
    if (status) {
        log.info("enable");
    } else {
        log.info("disable");
    }
}

// 굳이 switch를 사용하지 않아도 될 거 같지 않나요?
public void printStatus (boolean status) {
    switch(status){
        case true:
            log.info("enable");
            break;
        default :
            log.info("disable");
            break;
    }
}
```
```java
// 이걸 swtich로 처리한다고 생각하면... 
public void printGrade (int point) {
    if (point >= 0 && point < 40) {
        log.info("C");
    } else if (point >= 40 && point < 80) {
        log.info("B");
    } else if (point >= 80) {
        log.info("A");
    } else {
        log.info("D");
    }
}
```


#### switch를 사용하는 경우
<pre>
1. 언제든 조건이 추가될 가능성이 있는 경우
-> swtich는 조건의 추가, 제거에 매우 유용합니다.
2. 위 if-else를 사용하는 경우가 아닌 경우
</pre>
```java
public static CoffeeType findByOrder(Order order) {
    switch (order.getMenu() % 9) {
        case 0:
            return ESPRESSO;
        case 1:
            return AMERICANO;
        case 2:
            return CAPPUCCINO;
        case 3:
            return CAFE_LATTE;
        case 4:
            return CAFE_MOCHA;
        case 5:
            return CON_PANNA;
        case 6:
            return AFFOGATO;
        case 7:
            return CAFE_ROYAL;
        case 8:
            return DUTCH;
        default:
            return EMPTY;
    }
}
```

## 마무리 
결론은 저는 if-else가 더 좋다! 또는 switch가 더 좋다! 가 아닌 **상황에 맞는 코드를 작성하는 것이 중요하다**고 생각합니다.

오늘도 이와 관련된 재미있는 주제가 있으니 구경해보시면 좋을 듯합니다.  
[If statements vs switch cases? in a JavaScript game and if to use a function](https://softwareengineering.stackexchange.com/questions/222145/if-statements-vs-switch-cases-in-a-javascript-game-and-if-to-use-a-function){:
target="_blank"}  
<br>
<br>
P.S  
혹시나 설명이 필요한 부분이 더 있다면 댓글로 남겨주세요.  
설명을 추가하고 다음 포스팅부터는 설명까지 넣을 수 있도록 하겠습니다.
 
두서없는 글을 읽어주셔서 감사합니다.  
