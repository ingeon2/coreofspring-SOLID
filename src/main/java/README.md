가장 처음 할 일은 예제 만들기임.
discount, member, order 패키지만들어서 인터페이스와 클래스까지 만들고 비즈니스 로직도 만들어서 자바 안에서만 잘 작동되는지 확인한것,  
여기서는 서로가 필요한 객체들을 생성자 주입이 아니라  
전부 다 각각 MemberService memberService = new MemberServiceImpl(); 이런식으로 사용함  
이 내용은 SOLID중에 D, 즉 추상화에 의존해야지, 구체화에 의존하면 안된다 의 원칙을 벗어남  
(memberService는 인터페이스(추상화)고 MemberServiceImpl는 클래스(구체화)니까.)  
//  
하지만(좋은점)  
package hello.core.order; 의 OrderServiceLmpl 클래스에서는  
매서드 안에서 int discountPrice = discountPolicy.discount(member, itemPrice); 때문에  
S의 원칙(단일책임) 은 잘 지켰음(int discountPrice)  
여기까지가 스프링 핵심 원리 이해1 - 예제 만들기 의 내용임. 오직 순수 자바를 사용.  
//  
//  
//  
그다음 할일은 예제의 기획이 변경되었을때 대처하는 방식을 보며, 문제점을 찾으며 하나하나 좋은 코드(DI,Ioc, 등)로 고쳐나가는것.  
먼저 package hello.core.order; 의 OrderServiceLmpl 클래스에서 할인정책 선택 방식이  
private final DiscountPolicy discountPolicy = new FixDiscountPolicy();  
private final DiscountPolicy discountPolicy = new RateDiscountPolicy();  
에서 무엇을 사용할지에 따라 주석처리해주어야함.  
즉, 위의 코드는  
★OCP, DIP 같은 객체지향 설계 원칙을 위배한것.   
1. (위에서 DIP는 클래스 의존관계를 분석해 보자. 추상(인터페이스) 뿐만 아니라 구체(구현) 클래스에도 의존.)  
추상(인터페이스) 의존: DiscountPolicy  
구체(구현) 클래스: FixDiscountPolicy , RateDiscountPolicy  
(discountPolicy 객체는 자기 역할만 하면 되는데 하위 클래스를 설정해주는 역할까지 하고 있다)  
2. (위에서 OCP는 지금 코드는 기능을 확장해서 변경하면, 클라이언트 코드에 영향을 준다! 따라서 OCP를 위반.)  
// 어려우면 pdf 다어이그램 볼것  
//  
그러면 위의 문제를 어떻게 해결할까나..?  
간단하게 추상화에만 의존하는 코드를 짜보자  
private final DiscountPolicy discountPolicy;  
짰다.   
근데 위에처럼만 코드를 짜면 DIP는 해결했지만,  Test 실행하면 NPE(null point exception 오류).  
당연하다. 할인정책이 추상매서드인데 매서드 내용이 없으니(인터페이스니까) 실행할 수 없다.  
//  
또 이문제는 어떻게 해결할까나..?  
클라이언트인 OrderServiceImpl 에 DiscountPolicy 의 구현 객체를 대신 생성하고 주입(DI)  
//  
//  
위의 문제를 종합하면,  
관심사의 분리  
애플리케이션을 하나의 공연이라 생각해보자. 각각의 인터페이스를 배역(배우 역할)이라 생각하자. 그런데!  
실제 배역 맞는 배우를 선택하는 것은 누가 하는가?  
로미오와 줄리엣 공연을 하면 로미오 역할을 누가 할지 줄리엣 역할을 누가 할지는 배우들이 정하는게  
아니다.   
이전 코드는 마치 로미오 역할(인터페이스)을 하는 레오나르도 디카프리오(구현체, 배우)가 줄리엣  
역할(인터페이스)을 하는 여자 주인공(구현체, 배우)을 직접 초빙하는 것과 같다.   
디카프리오는 공연도 해야하고 동시에 여자 주인공도 공연에 직접 초빙해야 하는 다양한 책임을 가지고 있다.  
//  
관심사를 분리하자  
배우는 본인의 역할인 배역을 수행하는 것에만 집중해야 한다.   
//  
디카프리오는 어떤 여자 주인공이 선택되더라도 똑같이 공연을 할 수 있어야 한다.  
공연을 구성하고, 담당 배우를 섭외하고, 역할에 맞는 배우를 지정하는 책임을 담당하는 별도의 공연 기획자가 나올시점이다.  
공연 기획자를 만들고, 배우와 공연 기획자의 책임을 확실히 분리하자.  
★그러한 공연 기획자가 AppConfig 이다.  
(애플리케이션의 전체 동작 방식을 구성(config)하기 위해, 구현 객체를 생성하고, 연결하는 책임을 가지는 별도의 설정 클래스)  
//  
정리  
AppConfig를 통해서 관심사를 확실하게 분리했다.  
배역, 배우를 생각해보자.  
AppConfig는 공연 기획자다.  
AppConfig는 구체 클래스를 선택한다. 배역에 맞는 담당 배우를 선택한다. 애플리케이션이 어떻게  
동작해야 할지 전체 구성을 책임진다.  
이제 각 배우들은 담당 기능을 실행하는 책임만 지면 된다.  
OrderServiceImpl 은 기능을 실행하는 책임만 지면 된다.  
//  
이후엔 AppConfig를 리팩터링했다.(한눈에 차지하는 영역을 볼 수 있게) 주석을 AppConfig에서 보기  
//  
AppConfig의 등장으로 애플리케이션이 크게 사용 영역과(AppConfig를 뺀 나머지), 객체를 생성하고 구성(Configuration)하는 영역으로 분리  
즉 이젠 AppConfig를 통해 OCP와 DIP 둘다 만족하게 됨  
//  
//  
새로운 할인 정책 개발,  관심사의 분리 AppConfig 등장,  AppConfig 리팩터링 위에서부터 이렇게 세개의 순서로 코딩함.  
//  
좋은 객체 지향 설계의 5가지 원칙의 적용 중에 여기까지 3가지 SRP, DIP, OCP 적용  
SRP - 한 클래스는 하나의 책임만 가져야 한다.  
(구현 객체를 생성하고 연결하는 책임은 AppConfig가 담당 클라이언트 객체는 실행하는 책임만 담당)  
DIP - 프로그래머는 “추상화에 의존해야지, 구체화에 의존하면 안된다.” 의존성 주입은 이 원칙을 따르는 방법 중 하나  
(AppConfig 생성)  
OCP - 소프트웨어 요소는 확장에는 열려 있으나 변경에는 닫혀 있어야 한다  
(소프트웨어 요소를 새롭게 확장해도 config만 변경하고, 사용 영역의 변경은 닫혀 있다.)  
//  
IoC,제어의 역전 (Inversion of Control)  
AppConfig가 등장한 이후에 구현 객체는 자신의 로직을 실행하는 역할만 담당한다. 프로그램의  
제어 흐름은 이제 AppConfig가 가져간다. 예를 들어서 OrderServiceImpl 은 필요한 인터페이스들을  
호출하지만 어떤 구현 객체들이 실행될지 모른다.  
이렇듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 제어의 역전(IoC)이라  
한다.  
//  
정적인 클래스 의존관계  
import 코드만 보고 의존관계를 쉽게 판단할 수 있다. 정적인 의존관계는 애플리케이션을 실행하지 않아도 분석가능   
(인터페이스와 하위클래스)  
//  
동적인 객체 인스턴스 의존 관계  
애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계다  
(Fix를 사용할지 Rate를 사용할지)  
//  
의존관계 주입을 사용하면 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계를  
쉽게 변경할 수 있다.  
//  
IoC 컨테이너, DI 컨테이너  
AppConfig 처럼 객체를 생성하고 관리하면서 의존관계를 연결해 주는 것을  
IoC 컨테이너 또는 DI 컨테이너라 한다. (나중엔 Spring 이 할 역할)  
//  
//  
3번의 변경 후  
이젠 Spring을 사용! (only java → Spring)  
by @Configuration,@Bean in AppConfig  
ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);  in MemberApp  
AppConfig 환경설정 정보가지고 @Bean 붙은것들 컨테이너에서 생성해서 관리해줌.  
MemberService memberService = applicationContext.getBean("memberService", MemberService.class);  in MemberApp  
"memberService" 이거는 이 이름의 매서드를 찾을거라는 뜻. 그러면 주석 없이  OrderApp 바꿔보기.  
//  
쉽게말해서 applicationContext 에 AppConfig 할당.  
memberService 에 applicationContext를 사용한 매서드(getBean)로 MemberService 할당.  
//  
스프링 컨테이너  
ApplicationContext 를 스프링 컨테이너라 한다.  
기존에는 개발자가 AppConfig 를 사용해서 직접 객체를 생성하고 DI를 했지만, 이제부터는 스프링 컨테이너를 통해서 사용한다.  
스프링 컨테이너는 @Configuration 이 붙은 AppConfig 를 설정(구성) 정보로 사용한다.   
여기서 @Bean 이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다.   
이렇게 스프링 컨테이너에 등록된 객체를 스프링 빈이라 한다.  
스프링 빈은 @Bean 이 붙은 메서드의 명을 스프링 빈의 이름으로 사용한다.   
( memberService , orderService ) 매서드 in AppConfig  
이전에는 개발자가 필요한 객체를 AppConfig 를 사용해서 직접 조회했지만, 이제부터는 스프링 컨테이너를 통해서 필요한 스프링 빈(객체)를 찾아야 한다.   
스프링 빈은 applicationContext.getBean() 메서드를 사용해서 찾을 수 있다.  
기존에는 개발자가 직접 자바코드로 모든 것을 했다면 이제부터는 스프링 컨테이너에 객체를 스프링 빈으로 등록하고,   
스프링 컨테이너에서 스프링 빈을 찾아서 사용하도록 변경되었다.  
//  
//
  
여기서부터는 컨테이너와 빈의 내용 (AppConfig까지 마친 후)  
//  
//  
ApplicationContextInfoTest 클래스 추가로 컨테이너의 Bean 확인.  
빈 이름은 항상 다른 이름을 부여,  
스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입  
예를 들어, (AppConfig 클래스에서 memberService와 orderService는 memberRepository를 의존)  
단순히 자바 코드를 호출하는 것 같지만, 차이가 있다. 이 차이는 뒤에 싱글톤 컨테이너에서 설명.  

beanfind 패키지 안에서 컨테이너로 올려놓은 빈을 찾는 각각의 방법을 가진 클래스들을 만들었다.  