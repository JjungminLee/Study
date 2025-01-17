- 스프링 프레임워크 없이도 자바만으로도 스프링처럼 구현할 수 있다!

## 테스트코드 짜는 방법

```
package com.example.springexercise;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

public class MemberServiceTest {
    MemberService memberService = new MemberServiceImpl();
    @Test
    void join(){
        //given
        Member member = new Member(1L,"memberA",Grade.VIP);

        //when
        memberService.join(member);
        Member findMember = memberService.findMember(1L);

        //then join한것과 찾은게 똑같은지 test        Assertions.assertEquals(member,findMember);
    }
}
```

- given, when, then으로 구성

```
package com.example.springexercise.discount;

import com.example.springexercise.Grade;
import com.example.springexercise.Member;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

public class RateDiscountPolicyTest {

    RateDiscountPolicy discountPolicy = new RateDiscountPolicy();

    @Test
    @DisplayName("VIP는 10% 할인이 적용되어야 한다")
    void vip_o(){
        //given
        Member member = new Member(1L,"memberVIP", Grade.VIP);
        //when
        int discount = discountPolicy.discount(member,10000);
        //then
        Assertions.assertEquals(discount,1000);
    }

    @Test
    @DisplayName("VIP가 아니면 10% 할인이 적용되지 않아야한다")
    void vip_x(){
        //given
        Member member = new Member(1L,"memberVIP", Grade.BASIC);
        //when
        int discount = discountPolicy.discount(member,10000);
        //then
        Assertions.assertEquals(discount,1000);
    }
}
```

## DIP 위반

```
package com.example.springexercise;

public class MemberServiceImpl implements  MemberService{

    // 구현체에 의존하기 때문에 DIP 위반
    private  final MemberRepository memberRepository = new MemoryMemberRepository();
    @Override
    public void join(Member member) {
        memberRepository.save(member);

    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```

## DIP에 만족하도록 인터페이스에만 의존하게끔 해야한다.

```
private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
```

- 이거 때문에 의존성이 생김 -> RateDiscountPolicy에 의존성이 생긴다.

```
private DiscountPolicy discountPolicy;
```

- 이렇게 하면 NPE가 떠버린다.

## 관심사의 분리

- 중간에서 중재하며, 구현객체를 생성하고 연결하는 책임을 가지는 별도의 설정 클래스
  - AppConfig
    - 배우 섭외하는 역할하는 클래스

```
package com.example.springexercise.member;

public class MemberServiceImpl implements  MemberService{

    // 구현체에 의존하기 때문에 DIP 위반
    private  final MemberRepository memberRepository;

    public MemberServiceImpl(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
    @Override
    public void join(Member member) {
        memberRepository.save(member);

    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```

- new를 통해 MemoryMemberRepository 객체를 생성하는 코드를 지운다. (의존하지 않는다)
  - 단지 MemberRepository 인터페이스에만 의존한다.
  - new를 통해 객체 생성은? AppConfig에서 해준다.

```
package com.example.springexercise;

import com.example.springexercise.member.MemberService;
import com.example.springexercise.member.MemberServiceImpl;
import com.example.springexercise.member.MemoryMemberRepository;

public class AppConfig {

    public MemberService memberService(){
        return new MemberServiceImpl(new MemoryMemberRepository());
    }
}

```

- 생성자 주입

```
package com.example.springexercise.order;

import com.example.springexercise.member.Member;
import com.example.springexercise.member.MemberRepository;
import com.example.springexercise.member.MemoryMemberRepository;
import com.example.springexercise.discount.DiscountPolicy;

public class OrderServiceImpl implements  OrderService {

    private final MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

    public OrderServiceImpl(MemberRepository memberRepository,DiscountPolicy discountPolicy){
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }

    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        // 단일 책임 원착
        Member member = memberRepository.findById(memberId);
        // Order Service 쪽은 할인정책을 모르고, 할인 정책 쪽이 알아서 하게끔 위임
        int discountPrice = discountPolicy.discount(member,itemPrice);
        return new Order(memberId,itemName,itemPrice,discountPrice);
    }
}
```

- OrderServiceImpl은 인터페이스에만 의존하고 있다! -> 철저히 DIP 만족

```
package com.example.springexercise;

import com.example.springexercise.discount.FixDiscountPolicy;
import com.example.springexercise.member.MemberService;
import com.example.springexercise.member.MemberServiceImpl;
import com.example.springexercise.member.MemoryMemberRepository;
import com.example.springexercise.order.OrderService;
import com.example.springexercise.order.OrderServiceImpl;

public class AppConfig {

    public MemberService memberService(){
        return new MemberServiceImpl(new MemoryMemberRepository());
    }

    public OrderService orderService(){
        return new OrderServiceImpl(new MemoryMemberRepository(),new FixDiscountPolicy());
    }
}
```

## AppConfig

- 구현객체를 생성
- 생성한 객체의 참조를 생성자를 통해 주입 (연결) 해준다.
- 의존관계에 대한 고민은 외부에 맡기고, 실행에만 집중한다.
- 정리
  - DIP완성 : MemberServiceImpl은 MemberRepository 추상에만 의존. 구체 클래스는 몰라도 된다!
  - 관심사의 분리 : 객체 생성하는 역할과 실행하는 역할의 분리!
  - 의존관계를 외부에서 주입해 준다고 해서 DI

```
package com.example.springexercise.member;

import com.example.springexercise.AppConfig;

public class MemberApp {
    public static void main(String[]args){
        AppConfig appConfig = new AppConfig();
        MemberService memberService = appConfig.memberService();
        Member member = new Member(1L,"memberA",Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println("newMember = "+member.getId());
        System.out.println("findMember = "+findMember);


    }
}
```

- MemberService memberService = new MemberServiceImpl(); -> 기존에 이렇게 의존성을 생기는 것을 방지하기 위해 AppConfig를 통해 외부에서 의존관계를 주입한다.

## 테스트 코드에서는?

```
package com.example.springexercise;

import com.example.springexercise.member.Grade;
import com.example.springexercise.member.Member;
import com.example.springexercise.member.MemberService;
import com.example.springexercise.member.MemberServiceImpl;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

public class MemberServiceTest {

    MemberService memberService;
    @BeforeEach
    public void beforeEach(){
        AppConfig appConfig = new AppConfig();
        memberService = appConfig.memberService();
    }

    @Test
    void join(){
        //given
        Member member = new Member(1L,"memberA", Grade.VIP);

        //when
        memberService.join(member);
        Member findMember = memberService.findMember(1L);

        //then join한것과 찾은게 똑같은지 test        Assertions.assertEquals(member,findMember);
    }
}
```

- @BeforeEach
  - 각 테스트 메서드가 실행되기 전에 실행해야 할 초기화 작업

## AppConfig를 리팩터링 해보자!

- 역할과 구현이 드러나게 짜는 것이 중요하다!

```
package com.example.springexercise;

import com.example.springexercise.discount.DiscountPolicy;
import com.example.springexercise.discount.FixDiscountPolicy;
import com.example.springexercise.member.MemberService;
import com.example.springexercise.member.MemberServiceImpl;
import com.example.springexercise.member.MemoryMemberRepository;
import com.example.springexercise.order.OrderService;
import com.example.springexercise.order.OrderServiceImpl;

public class AppConfig {

    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }

    private static MemoryMemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService(){
        return new OrderServiceImpl(memberRepository(),discountPolicy());
    }

    public DiscountPolicy discountPolicy(){
        return new FixDiscountPolicy();
    }
}
```

- 이제 사용영역과 구성영역으로 구성된다.
  - 구성영역이 AppConfig
  - 사용영역은 OrderServiceImpl 같은거
  - 구성영역만 바뀌고, 사용영역은 바꾸지 않는다!

## 좋은 객체지향 설계의 5가지 원칙의 적용

- DIP
  - 추상화에 의존해야지, 구현 클래스에 의존하면 안된다!
  - 기존 클라이언트(OrderServiceImpl)코드는 DIP를 지키며 DiscountPolicy 추상화 인터페이스에 의존하는 것 같았지만 FixDiscountPolicy인 구체화 구현클래스에도 의존
  - AppConfig가 FixDiscountPolicy 객체를 대신 생성해서 의존관계 주입
- OCP
  - 확장에는 열려있나 변경에는 닫혀있다
  - 구성영역은 열려있으나 사용영역은 닫혀있다.

## Ioc,DI, 컨테이너

- Ioc (제어의 역전)
  - 내가 호출하는게 아니라 프레임워크가 대신 호출
  - 제어흐름을 AppConfig가 가지고 있음.
  - 프레임워크 vs 라이브러리
    - Junit (프레임워크)
      - 자신만의 라이프 사이클에서 내 코드를 실행시켜준다.
    - 라이브러리
      - 내가 작성한 코드가 직접 제어의 흐름을 담당
      - 내가 직접 호출
- 의존관계 주입
  - 실제 어떤 구현객체가 사용될지는 모름
  - 정적인 클래스 의존관계와 동적인 객체 의존관계
  - 정적인 클래스 의존관계
    - 실행하지 않고도 판단
    - import코드만 보고 판단 가능
  - 동적인 객체 의존관계
    - 실행시점에 결정
  - 정적인 의존관계를 변경하지 않고, 동적인 의존관계를 쉽게 변경 가능
- IoC 컨테이너, DI 컨테이너
  - AppConfig처럼 객체 생성하고 관리하면서 의존관계 연결해주는 것을 IoC컨테이너 또는 DI컨테이너
  -

## 스프링으로 전환

```
package com.example.springexercise.member;

import com.example.springexercise.AppConfig;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;


public class MemberApp {
    public static void main(String[]args){
//        AppConfig appConfig = new AppConfig();
//        MemberService memberService = appConfig.memberService();
        // @Bean을 스프링 컨테이너에 등록해서 찾아와줌
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        MemberService memberService = applicationContext.getBean("memberService",MemberService.class);
        Member member = new Member(1L,"memberA",Grade.VIP);
        memberService.join(member);

        Member findMember = memberService.findMember(1L);
        System.out.println("newMember = "+member.getId());
        System.out.println("findMember = "+findMember);


    }
}
```

- @Configuration
- @Bean
  - 스프링 컨테이너에 등록
  - 빈 등록한 메서드 모두 호출해서 스프링 컨테이너에 등록
  - 메서드 명을 스프링 빈의 이름으로 사용
- `ApplicationContext`는 Spring 컨테이너를 의미
- 기존에는 AppConfig를 통해 DI를 했지만 스프링 컨테이너를 통해 DI
- 이전에는 AppConfig를 통해 객체를 조회, 이제는 스프링 컨테이너를 통해 스프링 빈을 조회
