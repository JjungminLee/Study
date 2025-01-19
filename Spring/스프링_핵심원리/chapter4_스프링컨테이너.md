## CRUD 프로젝트에서 사용했던 어노테이션 정리

**@NoArgsConstructor**

- 기본 생성자
- JPA 엔터티에 필수
  @EntityListeners(AuditingEntityListener.class)
- 엔터티의 변경사항 자동 감지
- JPA 이벤트 리스너
  @MappedSuperclass
- 공통 속성 정의
- DB테이블로 매핑되지 않음

## 스프링 컨테이너 생성

- Application Context를 스프링 컨테이너라고 함
- Application Context는 인터페이스
- 스프링 컨테이너는 BeanFactory, ApplicationContext로 구분해서 발함
- 스프링 빈 저장소는 <빈이름, 빈객체>로 구성됨
  - <memberService,MemberServiceImpl@x01>
- 빈 이름은 항상 다르게 부여해야함!

## 컨테이너에 등록된 모든 빈 조회

- AppConfig도 스프링 빈에 해당

```
@Test
@DisplayName("모든 빈 출력하기")
void findAllBean(){
    String[] beanDefinitionNames = ac.getBeanDefinitionNames();
    for(String beanDefname : beanDefinitionNames){
        Object bean = ac.getBean(beanDefname);
        System.out.println("name "+beanDefname+"object: "+bean);

    }
}
```

## 테스트코드

- isInstanceOf

```
Assertions.assertThat(memberService).isInstanceOf(MemberService.class);
```

특정 객체가 예상한 클래스 타입의 **인스턴스인지 검증**할 때 사용

## 동일한 타입이 둘 이상일 때, 스프링 빈 조회

```
package com.example.springexercise;

import com.example.springexercise.member.MemberRepository;
import com.example.springexercise.member.MemoryMemberRepository;
import org.assertj.core.api.Assertions;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

import static org.junit.jupiter.api.Assertions.assertThrows;

public class ApplicationContextSameBeanFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

    @Test
    @DisplayName("타입으로 조회 시 같은 타입이 둘 이상 있으면 중복 오류가 발생한다")
    void findBeanByTypeDuplicate(){
        MemberRepository bean = ac.getBean(MemberRepository.class);
        assertThrows(NoUniqueBeanDefinitionException.class,()->ac.getBean(MemberRepository.class));
    }

    @Test
    @DisplayName("타입으로 조회 시 같은 타입이 둘 이상 있으면, 빈 이름을 지정하면 된다.")
    void findBeanByName(){
        MemberRepository memberRepository = ac.getBean("memberRepository1",MemberRepository.class);
        Assertions.assertThat(memberRepository).isInstanceOf(MemberRepository.class);
    }

    @Test
    @DisplayName("특정 타입을 모두 조회하기")
    void findAllBeanByType(){
        Map<String,MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
        for(String key : beansOfType.keySet()){
            System.out.println("key= "+key+"value "+beansOfType.get(key));
        }
    }
    // static 을 선언했다는 것은 이 클래스는 상위 클래스 안에서만 사용하겠다는것
    @Configuration
    static class SameBeanConfig{
        @Bean
        public MemberRepository memberRepository1(){
            return new MemoryMemberRepository();
        }

        @Bean
        public MemberRepository memberRepository2(){
            return new MemoryMemberRepository();
        }
    }
}
```

- static class를 사용해서 AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class); -> 현재 클래스에서만 사용하는 클래스를 주입하는 방법!
- SameBeanConfig에서 MemoryMemberRepository객체를 두개나 생성해서 중복 오류가 발생한게 이 코드의 특징이다.

## 똑같은 Assertions인데 클래스가 다르다?

- org.junit.jupiter.api.Assertions
  - JUnit 테스트를 위해 기본적으로 제공되는 Assertion 기능.
  - 테스트가 간단하고, JUnit 내장 기능만으로 충분한 경우.
  - 예: 값 비교, 단순 조건 확인, 예외 검증.
- org.assertj.core.api.Assertions
  - AssertJ라이브러리
  - 복잡한 테스트 조건을 검증하거나, 가독성이 중요한 경우.
  - 예: 리스트의 순서, 포함 여부 검증, 객체의 속성 확인, 체이닝 방식의 검증이 필요한 경우.
    - 체이닝 방식?
    - **메서드 호출을 연속적으로 연결**하여 코드의 가독성을 높이고, **명령을 순차적으로 수행**할 수 있도록

## 스프링 빈 조회, 상속관계

- 부모 타입 조회하면, 자식 타입도 다 끌려나온다!

## BeanFactory와 Application Context

- BeanFactory
  - 스프링 컨테이너의 최상위 인터페이스
  - 스프링 빈 관리하고 조회하는 역할 담당
  - getBean()제공
- ApplicationContext
  - BeanFactory의 기능을 모두 상속받아서 제공
  - 애플리케이션 개발할때는 빈 관리 뿐만아니라 많은 부가 기능 필요
  - MessageSource, ApplicationEventPublisher, ResourcePatternResolver
    - 메세지 소스 : 한국에서 들어오면 한국으로 출력, 미국에서 들어오면 영어로 출력
    - 환경변수 : 로컬, 개발환경
    - 애플리케이션 이벤트 : 이벤트를 발행하고 구독하는 모델 편리하게 지원
    - 편리한 리소스 조회

## BeanDefinition

- 스프링 빈 설정 메타 정보
- 스프링 컨테이너는 BeanDefinition 자체에만 의존
- AnnotatedBeanDefinitionReader가 AppConfig.class를 읽어서 Bean definition 빈 메타정보를 생성한다.
-
