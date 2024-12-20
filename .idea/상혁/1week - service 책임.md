### 비즈니스 로직 vs 도메인 로직

https://enterprisecraftsmanship.com/posts/what-is-domain-logic/

다음 2가지 코드를 살펴보자.

1. 
```java
// service code
public class Shop {
  public void sell(Account account, Product product) {
    if (account.canAfford(product.getPrice()) {
      account.withdraw(product.getPrice());
      System.out.println(product.getName() + "를 구매했습니다.");
    } else {
      System.out.println("잔액이 부족합니다.");
    }
  }
}
// domain
class Account {

  private long money;

  public boolean canAfford(long amount) {
    return money >= amount;
  }

  public void withdraw(long amount) {
    money -= amount;
  }
}
```
2.
```java
public class Shop {
  public void sell(Account account, Product product) {
    account.withdraw(product.getPrice());
  }
}

class Account {

  private long money;

  private boolean canAfford(long amount) {
    return money >= amount;
  }

  public void withdraw(long amount) {
    if (canAfford(amount)) {
      money -= amount;
      System.out.println(product.getName() + "를 구매했습니다.");
    } else {
      System.out.println("잔액이 부족합니다.");
    }
  }
}
```
`withdraw()` 로직 내부에 `canAfford()` 검증 로직을 넣어서, Account에 행동 책임을 전가했다.

둘 중 어느 코드가 더 좋을까?

이는 추구하는 방향에 따라 다를 것 같다고 생각했다.
- Service 코드는 하나의 명세처럼 읽혀야 하는가
- 행동으로만 보여줘야 하는가

개인적인 경험으로는, 2번처럼 구현할 때 Service 코드가 간단하게 되고 그로인해 가독성이 좋아지는 코드를 만들었다.
하지만 너무 캡슐화 한 나머지, Service 테스트 코드를 작성할 때 해당 메서드가 어떤 flow를 가지는지 파악하기 힘들어서 테스트 코드도 같이 작성하기 힘든 기억이 있다.

다음과 같은 간단한 경우는 domain 비즈니스 로직으로 판단했다.
- 값 검증
- 값 변경

다음과 같은 경우는 Service 코드로 분류했다.
- 예외 처리
- 다른 모듈 사용

결국 1번처럼 구현하는 방법이
- 객체에게 책임을 위임
하는 적절한 방법이지 않을까 생각한다.

---

서비스가 가지는 책임은 다음과 같다.
- 트랜잭션 관리
- 데이터 조립
- 비즈니스 로직
- 데이터 검증
- ...

결국 서비스의 책임이 많아져서 이를 분배하는 방법을 찾던 중 도메인 주도 개발 방법론이 등장함을 알게 됨

- 테스트 코드 작성: 서비스에 몰려있는 역할과 반대로, 도메인 단위테스트에서 
- 응집도 -> 도메인 코드를 보면 관련된 비즈니스를 바로 알 수 있음

비즈니스 로직에 검증 로직이 들어감
결국 2번 과정에서 테스트를 진행할 때, 도메인의 

메서드가 여러 책임을 가질 수 있다는 단점
이것은 어느 정도의 감수가 필요

'정책'과 메서드의 어느정도의 트레이드 오프가 필요.
캡슐화 과정 -> 비즈니스 로직은 처리함과 동시에 그 속에서 캡슐화가 일어남.

https://curiousjinan.tistory.com/entry/spring-architecture-domain

> 코드의 구조를 이끄는 것은 도메인 안에 정립된 개념의 분류 체계가 아니라 객체들의 협력이다.

https://eunjin3786.tistory.com/326

항상 역할, 책임, 협력을 고려하자.

# 책임, 모호한 녀석..

'책임'이라는 단어는 굉장히 추상적이라 생각한다.

사람들마다 보는 기준이 다르고, 그에 따라 다양한 책임이 등장하고, 설계가 등장한다.

흔히 말하는 객체지향 원칙 5개 중 한가지인 SRP는 "단 하나의 책임만 있어야 한다"라고 익히 들었을 것. 하지만 그 책임을 나누는 것이 너무 자유도가 높은 느낌이었다. 

> "SRP는 굉장히 단순한 원칙처럼 보이지만 협업하는 실무 레벨에서 이 원칙을 적용하는 것은 꽤나 어렵습니다.
> 왜냐하면 책임은 문맥을 포함하는 개념이기 때문이며, 그로 인해 '책임'이라는 개념은 그것을 바라보는 개인이나 상황마다 다르게 해석될 여
> 지가 있기 때문입니다.
>
> 자바/스프링 개발자를 위한 실용주의 프로그래밍 - 김우근

그러면 책임이라는 것을 어떻게 이해할 수 있을까?

클린 아키텍처에서는 '액터'라는 개념이 등장했다.

- 액터(actor): 메세지를 전달하는 주체


> SRP는 액터에 대한 책임. 
> - 어떤 모듈이나 클래스가 담당하는 액터가 혼자라면 단일 책임 원칙을 지키고 있는 것.
> - 어떤 모듈이나 크랠스가 담당하는 액터가 여럿이라면 단일 책임 원칙에 위배.
> 시스템에 존재하는 액터를 먼저 이해해야 합니다. 그리고 그러기 위한 문맥과 상황이 필요합니다. 단일 책임 원칙에서 말하는 책임은 결국 액터에 대한 책임이기 때문입니다.
> 
> 자바/스프링 개발자를 위한 실용주의 프로그래밍 - 김우근

나는 이렇게 해석했다.

'일정 시간 간격을 수정한다'라는 로직이 있을 때, 일반적으로 service 코드에 `updateDateTime`이라는 메서드로 구현할 것이다.

하지만 일정은 팀 일정과 공지 일정으로 2개가 나눠서 존재했다.

이를 사용하는 액터를 떠올려보면, controller 즉, 사용자가 접근하는 flow를 생각했을 때, 팀 일정을 수정하라고 시키는 액터, 공지 일정을 수정하라고 시키는 액터 2명이 떠올랐다. 그러면 책임이 2가지가 존재하는 것.

액터가 둘 이상일 때 책임도 둘 이상이 되므로 변경할 때 힘들 수 있다는 의미로 받아들였다.

'액터' 명심하자.

---

[자바/스프링 개발자를 위한 실용주의 프로그래밍 - 김우근]()
