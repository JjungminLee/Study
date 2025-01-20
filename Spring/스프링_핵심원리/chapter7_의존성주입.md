## 생성자 주입

- 생성자를 통해 의존관계 주입
- 생성자 호출 시점에 딱 1번 호출 보장
  - 불변, 필수 의존관계에 사용

```
private final MemberRepository memberRepository;
private final DiscountPolicy discountPolicy;
@Autowired
public OrderServiceImpl(MemberRepository memberRepository,DiscountPolicy discountPolicy){
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
```

- 생성자가 딱 1개만 있으면 Autowired 생략해도 된다!
- 프레임워크에 의존하지 않고, 순수한 자바언어의 특징을 잘 살리는 방법!
- 테스트 돌릴때 굉장히 좋음!

## 수정자 주입

- 선택,변경이 있는 의존관계에 사용
- @Autowired(required=false) -> 변경이 있을 경우 사용됨
  - 주입할 대상이 없으면 오류가 발생하기에 @Autowired(required=false)를 써야함

## 옵션처리

- 주입할 스프링 빈이 없어도 동작해야할 때가 있다
  - @Autowired만 사용하면 자동 주입 대상이 없으면 오류 발생
  - @Autowired(required=false) 사용해야함!
    - 자동 주입할 대상이 없으면 수정자 매서드 자체가 호출이 안된다!
  - @Nullable
    - 자동 주입할 대상이 없으면 null이 입력
  - Optional<>
  - Member는 스프링 빈이 아니다!

```
@Autowired(required = false)
public void setNoBean(Member member){
    System.out.println("member "+member);
}
```

- required=false이기에 호출자체가 안된다!

## Lombok

- getter,setter 자동으로 만들어준다!

```
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements  OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
```

- 생성자 주입을 RequiredArgsConstructor로 간편하게 정리할 수 있다!

## 조회되는 빈이 2개 이상일떄?

- @Autowired 필드명 매칭
- @Qualifier -> @Qualifier끼리 매칭 -> 빈 이름 매칭
- @Primary사용

### @Autowired

- 타입 매칭 시도
  - 여러 빈이 있으면
    - 필드이름, 파라미터 이름으로 추가 매칭 실행

### @Qualifier

- 추가 구분자
- Qualifier끼리 매칭한다
  - Qualifier없으면 빈 이름 매칭

### @Primary (추천!)

- DiscountPolicy 구현한게 RateDiscount, FixedDiscount가 있는데
  - @Primary를 RateDiscount에 붙이면
  - DiscountPolicy 빈 불러올때 RateDiscount로 불러옴

### @Primary와 @Qualifier활용

- 메인데이터베이스와 커넥션을 획득하는 스프링 빈이 있고 가끔 사용하는 서브 데이터베이스의 커넥션을 획득하는 스프링 빈이 있음
- 메인 데이터베이스의 커넥션 획득은 @Primary
- 서브 데이터베이스의 커넥션 획득은 @Qualifier
- 우선순위는 @Qualifier가 높다!

## Qualifier에서 자꾸 문자열 써야할때!

- Qualifier("mainDiscountPolicy")
  - 문자열이기에 오타 가능!

```
package com.example.springexercise.annotation;


import org.springframework.beans.factory.annotation.Qualifier;

import java.lang.annotation.*;

@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {
}
```

- 어노테이션을 붙여서 해결한다!

## 조회한 빈이 모두 필요할때 List,Map

```
package com.example.springexercise.autowired;

import com.example.springexercise.AutoAppConfig;
import com.example.springexercise.discount.DiscountPolicy;
import com.example.springexercise.member.Grade;
import com.example.springexercise.member.Member;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;


import java.util.List;
import java.util.Map;

public class AllBeanTest {

    @Test
    void findAllBean(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class,DiscountService.class);
        DiscountService discountService = ac.getBean(DiscountService.class);
        Member member = new Member(1L,"userA", Grade.VIP);
        int discountPrice = discountService.discount(member,10000,"fixDiscountPolicy");
        Assertions.assertThat(discountPrice).isEqualTo(1000);

    }
    static class DiscountService{

        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policies;

        @Autowired
        public DiscountService(Map<String,DiscountPolicy>policyMap,List<DiscountPolicy> policies){
            this.policyMap=policyMap;
            this.policies=policies;
        }
        public int discount(Member member,int price,String discountCode){
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            return discountPolicy.discount(member,price);
        }

    }
}
```

- DiscountService는 Map으로 모든 DiscountPolicy주입받음
- 동적으로 빈 선택 시 -> map을 사용하면 편리

## 자동 의존관계, 수동 의존관계 주입?

- 자동 의존관계 주입을 선호
- 업무로직 -> 자동 의존관계
  - controller, service, repository
- 기술 지원 로직 -> 수동 의존 관계
  - 수동 빈 등록
- 비즈니스 로직중 다형성이 포함 시 -> 수동등록
  - DiscountPolicyConfig
    - 이런식으로 해줘야 다른 사람이 볼때 편함
