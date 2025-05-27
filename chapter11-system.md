# 11장 시스템
> _"복잡성은 죽음이다. 개발자에게서 생기를 앗아가며, 제품을 걔획하고 제작하고 테스트하기 어렵게 만든다"_ - 레이 오지, 마이크로소프트 CTO

## 도시를 세운다면?
만약 나 혼자서 도시를 세운다고 생각해보자. 도시에서 돌아가는 모든 세세한 일을 다 관리할 수는 없을 것이다. 

흔히 소프트웨어 팀도 도시처럼 구성한다. 도시(소프트웨어 팀)는 수도 관리팀, 전력관리팀, 교통 관리 팀 등 각 분야를 관리하는 팀으로 돌아간다.

또한, 도시가 돌아가는 또다른 이유는 적절한 추상화와 모듈화 때문이다. 그래서 큰 그림을 이해하지 못할지라도 개인개인이 관리하는 '구성요소'는 효율적으로 돌아간다.

깨끗한 코드를 구현하면 낮은 추상화 수준에서 관심사를 분리하기 쉬워진다. 이 장에서는 **높은 추상화 수준, 즉 시스템 수준에서도 깨끗함을 유지하는 방법**을 살펴본다.


## 시스템 제작과 시스템 사용을 분리하라
우선 제작(construction)과 사용(use)은 아주 다르다는 사실을 명심한다. 

```text
예를 들어, 호텔을 새로 짓는다고 가정하자. 콘크리트 상자에 기중기와 승강기가 부착되어있고, 안전모에 작업복을 착용한 사람들이 바쁘게 움직인다. 1년 정도면 호텔은 완공된다. 

호텔이 완공되면 기중기와 승강기는 사라진다. 건물은 유리벽과 예쁜 색상의 벽으로 말끔히 꾸며지고, 호텔에 근무하고 채류하는 사람들은 호텔을 지을때 근무하던 사람들과 아주 다른 차림새를 하고 있을 것이다.
```

**소프트웨이 시스템은 (애플리케이션 객체를 제작하고 의존성을 서로 '연결'하는) 준비 과정과 (준비 과정 이후에 이이지는) 런타임 로직을 분리해야 한다.**

시작 단계는 모든 애플리케이션이 풀어야 할 **관심사**다. 이것이 이 장에서 우리가 맨 처음 살펴볼 **관심사**다. **관심사 분리는 우리 분야에서 가장 오래되고 가장 중요한 설계 기법 중 하나**다.

다음은 시작 단계 코드를 주먹구구식으로 구현할 뿐만 아니라 린타임 로직과 마구 뒤섞어 놓은 전형적인 예다.
```java
public Service getService() {
    if (service = null)
        service = new MyServiceImpl(... ) ; // 모든 상황에 적합한 기본값일까?
    return service;
}
```

이것은 초기화 지연(Lazy initializaton) 혹은 계산 지연(Lazy Evaluaton)이라는 기법이다.


> 초기화 지연은 객체나 데이터, 또는 계산 비용이 큰 리소스의 초기화를 그것이 실제로 필요할 때까지 미루는 프로그래밍 기법이다. 즉, 객체나 데이터를 미리 생성하지 않고, 실제로 접근하거나 사용할 때까지 초기화를 연기한다

우선, 실제로 필요할 때까지 객체를 생성하지 않으므로 불필요한 부하가 걸리지 않아, 애플리케이션을 시작하는 시간이 그만큼 빨라진다. 둘째, 어떤 경우에도 Null 포인터를 반환하지 않는다. 이러한 장점이 있다.

하지만 `getService` 메서드가 `MyServicelmpl`과 (위에서는 생략한) 생성자 인수에 명시적으로 의존한다. 따라서, **런타임 로직에서 `MyServicelmpl`객체를 전혀 사용 하지 않더라도 의존성을 해결하지 않으면 컴파일이 안 된다.**

테스트도 문제인데, `MyServicelmpl`이 무거운 객체라면 단위 테스트에서 `getService` 메서드를 호출하기 전에 **적절한 테스트 전용 객체**를 `service` 필드에 할당해야 한다. 

또한 `Service`가 null인 경로/null이 아닌 경로 등 모든 실행 경로도 테스트해야 한다. 즉, 메서드가 두 가지 이상 일을 수행하며, **작게나마 단일 책임 원칙(SRP)을 깬다는 말이다.**

하지만 무엇보다 우려인점은 **`MyServicelmpl`이 모든 상황에 적합한 객체인지 모든다는 사실이다.** 현실적으로 한 객체 유형이 모든 문맥에 적합할 가능성은 희박하다.

초기화 지연 기법을 한 번 정도 사용한다면 별로 심각한 문제가 아니다. 하지만 이러한 설정 방식이 애플리케이션 전반에 흩어져있다면, 모듈성은 저조하며 대개 중복이 심각해진다.

체계적이고 탄탄한 시스템을 만들고 싶다면 모듈성을 깨서는 절대로 안 된다. 객체를 생성하거나 의존성을 연결할 때도 마찬가지다. **설정 논리는 일반 실행 논리와 분리해야 모듈성이 높아진다.**

또한 **주요 의존성을 해소하기 위한 방식, 즉 전반적이며 일관적인 방식도 필요하다.**

### _Main 분리_

시스템 생성과 시스템 사용을 분리하는 한 가지 방법으로, **생성과 관련한 코드는 모두 `main`이나 `main`이 호출하는 모듈로 옮기고**, 나머지 시스템은 모든 객체가 생성되었고 모든 의존성이 연결되었다고 가정한다.

![그림 11.1 main()에서 생성 분리](/images/chapter11/main-example1.png)

흐름은 다음과 같다. 

1. `main`에서 시스템에 필요한 객체들을 모두 생성한 후 이를 애플리케이션에 넘긴다. 
2. 애플리케이션은 객체를 사용한다.

main과 애플리케이션 사이에 표시된 의존성 화살표를 보면, 모든 화살표는 main에서 애플리케이션 방향으로 향한다. 

**즉, 애플리케이션은 main이나 객체가 생성되는 과정을 모른다. 하지만, 모든 객체가 적절히 생성되었다고 가정하며 객체들을 사용할 수 있다.**

### _팩토리_

물론 때로는 **객체가 생성**되는 **시점**을 애플리케이션이 결정할 필요도 생긴다.

예를 들어, 주문처리 시스템에서 애플리케이션은 `LineItem` 인스턴스를 생성해 `Order`에 추가한다. 

이런 경우에는 추상 팩토리 패턴을 사용한다. 그러면 `LineItem`을 생성하는 시점은 애플리케이션이 결정하지만, `LineItem`을 생성하는 코드는 애플리케이션이 모른다. (그림 11-2 참조)

![그림 11.1 팩토리로 생성 분리](/images/chapter11/main-example2.png)

여기서도 마찬가지로 모든 의존성이 `main`에서 `OrderProcessing` (애플리케이션)으로 향한다. 즉, `OrderProcessing`은 `LineItem`이 생성되는 구체적인 방법을 모른다. 

그 방법은 `main` 의 `LineItemFactoryImplementation`이 안다. 
그럼에도 `OrderProcessing` 애플리케이션은 `LineItem` 인스턴스가 생성되는 시점을 완벽하게 통제할 수 있다.

### _의존성 주입_

사용과 제작을 분리하는 강력한 메커니즘 하나가 **의존성 주입(Dependency Injection, DI)** 이다. 의존성 주입은 **제어 역전(Inversion of Control, IoC) 기법을 의존성 관리에 적용한 메커니즘**이다. 

> 제어 역전은 프로그램의 제어 흐름을 개발자가 아닌 외부 환경(e.g. 프레임워크, 컨테이너)에 위임하는 소프트웨어 설계 원칙이다.

제어 역전에서는 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 떠넘긴다. 새로운 객체는 넘겨받은 책임만 맡으므로 단일 책임 원칙을 따른다. 

의존성 관리 맥락에서, 의존성 자체를 인스턴스로 만드는 책임은 지지않고, 이러한 책임을 다른 **전담 메커니즘에 넘겨야만** 한다. 그렇게 함으로써 제어를 역전한다. **초기 설정은 시스템 전체에서 필요하므로 대개 '책임질' 메커니즘으로 `main` 루틴이나 특수 컨테이너를 사용한다.**

다음 예제 코드를 살펴보자. JNDI 검색은 의존성 주입을 '부분적으로' 구현한 기능이다. 객체를 직접 생성하지 않고 외부 저장소(JNDI 디렉터리)에서 가져온다. 

```java
MyService myService = (MyService)(jndiContext.lookup("NameOfMyService"));
```

`myService`를 호출하는 클래스는 실제로 반환되는 객체의 유형을 제어하지 않는다. 대신 이 클래스는 의존성을 능동적으로 해결한다. (e.g. 수동적은 프레임워크가 해결)

이 코드는 의존성 주입을 부분적으로 구현했다. 진정한 의존성 주입은 여기서 한 걸음 더 나간다. 

**클래스가 의존성을 해결하려 시도하지 않는다.** 대신에 의존성을 주입하는 방법으로 설정자(Setter) 메서드나 생성자 인수를 (혹은 둘 다를) 제공한다. 

예를 들어, DI 컨테이너는 요청이 들어오면 필요한 객체의 인스턴스를 만든 후 생성자 인수나 설정자 메서드를 사용해 의존성을 설정한다. 실제로 생성되는 객체 유형은 설정 파일에서 지정하거나 특수 생성 모듈에서 코드로 명시한다. _스프링 프레임워크_ 는 가장 널리 알려진 자바 DI 컨테이너를 제공한다. 객체 사이 의존성은 XML 파일에 정의한다. 그리고 자바 코드에서는 이름으로 특정한 객체를 요청한다. 

## 확장

'처음부터 올바르게' 시스템을 만들 수 있다는 믿음은 미신이다. 

대신에 우리는 오늘 주어진 사용자 스토리에 맞춰 시스템을 구현해야 한다. 내일은 새로운 스토리에 맞춰 시스템을 조정하고 확장하면 된다. TDD, 리팩토터링, 깨끗한 코드는 코드 수준에서 시스템을 조정하고 확장하기 쉽게 만든다.

**하지만 시스템 수준에서는 어떨까?** 현실적으로 단순한 아키텍처를 복잡한 아키텍처로 조금씩 키울 수 없다.

_소프트웨어 시스템은 물리적인 시스템과 다르다. 관심사를 적절히 분리해 관리한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있다._

다음은 관심사를 적절히 분리하지 못해 유기적인 성장이 어려웠던 아키텍쳐, `EJB1`, `EJB2` 이다.

목룍 11-1은 클라이언트가 사용할 프로세스 내 지역 인터페이스이다. 목록 11-1 에서 열거하는 속성은 Bank 주소, 은행이 소유하는 계좌다. 

```java
// 목록 11-1 Bank EJB용 EJB2 지역 서비스
package com.example.banking;

import java.util.Collections;
import javax.ejb.*;

public interface BankLocal extends java.ejb.EJBLocalObject {
    String getStreetAddr1() throws EJBException;
    String getStreetAddr2() throws EJBException;
    String getCity() throws EJBException;
    String getState() throws EJBException;
    String getZipCode() throws EJBException;
    void setStreetAddr1(String street1) throws EJBException;
    void setStreetAddr2(String street2) throws EJBException;
    void setCity(String city) throws EJBException;
    void setState(String state) throws EJBException;
    void setZipCode(String zip) throws EJBException;
    Collection getAccounts() throws EJBException;
    void setAccounts(Collection accounts) throws EJBException;
    void addAccount(AccountDTO accountDTO) throws EJBException;
}

```

목록 11-2는 목록 11-1 인터페이스를 구현한 `Bank` 빈에 대한 구현 클래스다.  엔티티 빈은 관계형 자료, 즉 테이블 행을 표현하는 객체로, 메모리에 상주한다.

```java
// 목록 11-2 상응한느 EJB2 엔티티 빈 구현
import java.util.Collections;
import javax.ejb.*;

public abstract class Bank implements javax.ejb.EntityBean {
    // 비즈니스 논리...
    public abstract String getStreetAddr1();
    public abstract String getStreetAddr2();
    public abstract String getCity();
    public abstract String getState();
    public abstract String getZipCode();
    
    public abstract void setStreetAddr1(String street1);
    public abstract void setStreetAddr2(String street2);
    public abstract void setCity(String city);
    public abstract void setState(String state);
    public abstract void setZipCode(String zip);
    public abstract Collection getAccounts();
    public abstract void setAccounts(Collection accounts);
    
    public void addAccount(AccountDTO accountDTO) {
        InitialContext context = new InitialContext();
        AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
        AccountLocal account = accountHome.create(accountDTO);
        Collection accounts = getAccounts();
        accounts.add(account);
    }
    
    public abstract void setId(Integer id);
    public abstract Integer getId();
    
    public Integer ejbCreate(Integer id) { ... }
    public void ejbPostCreate(Integer id) { ... }
    
    // 나머지도 구현해야 하지만 일반적으로 비어있다.
    public void setEntityContext(EntityContext ctx) {}
    public void unsetEntityContext() {}
    public void ejbActivate() {}
    public void ejbPassivate() {}
    public void ejbLoad() {}
    public void ejbStore() {}
    public void ejbRemove() {}
}

```

**비즈니스 논리**는 `EJB2` 애플리케이션 **'컨테이너'에 강하게 결합된다**. 클래스를 생성할 때는 