## Mockito?

- Mock객체를 쉽게 만들고 검증할 수 있게 한다

### Mock

- 진짜 객체와 비슷하지만 물리적으로 같지 않고 프로그래머가 관리하는 객체
- Mock은 모든 상호작용을 기억한다.

### Stubbing

- test code에서 Mock 객체 사용시, Mock의 특정 메서드 호출과 응답을 정의하는 것

## Mock

- @Mock으로 만든 객체는 가짜 객체이며, 그안에 메서드 사용하려면 stubbing을 해야한다
- stubbing을 거치지 않으면 null값이 나온다
- **사용 목적:** 의존성을 제거하고 특정 메서드의 동작을 원하는 방식으로 조작할 때 사용.
- **일반적인 사용 사례:** 데이터베이스나 외부 API 호출 같은 의존성을 제거하고 단위 테스트 수행.

```
import static org.mockito.Mockito.*;

import org.junit.jupiter.api.Test;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import java.util.List;

class MockTest {

    @Mock
    private List<String> mockList;

    @Test
    void testMock() {
        MockitoAnnotations.openMocks(this);

        // Mock 객체의 동작 설정
        when(mockList.size()).thenReturn(5);

        // 호출 시 설정된 값이 반환됨
        System.out.println(mockList.size()); // 5
    }
}

```

- 실제 List의 동작이 수행되지 않고 5반환

## Spy

- @Spy로 만든 Mock객체는 진짜 객체이다.
- 메서드 실행시 스터빙 하지 않으면 기존 객체의 로직을 실행하며, 스터빙한 경우, 스터빙 한 값을 리턴한다.
- **사용 목적:** 기존 로직을 유지하면서 일부 메서드만 `stub` 처리하고 싶을 때.
- **일반적인 사용 사례:** 기존 메서드가 일부 필요한 경우, 단순 `Mock` 대신 `Spy`를 사용하여 부분적인 `stubbing` 수행.

```
import static org.mockito.Mockito.*;

import org.junit.jupiter.api.Test;
import org.mockito.MockitoAnnotations;
import org.mockito.Spy;

import java.util.ArrayList;
import java.util.List;

class SpyTest {

    @Spy
    private List<String> spyList = new ArrayList<>(); // 실제 객체 생성

    @Test
    void testSpy() {
        MockitoAnnotations.openMocks(this);

        spyList.add("hello"); // 실제 메서드 실행됨
        spyList.add("world");

        // spy 객체이므로 실제 List의 동작을 유지하지만, 특정 메서드만 stub 처리 가능
        when(spyList.size()).thenReturn(100);

        System.out.println(spyList.get(0)); // hello (실제 동작)
        System.out.println(spyList.size()); // 100 (Stubbed)
    }
}

```

### 📚 스터빙 하는 방법

Mockito 에서는 when 메소드를 이용해서 스터빙을 지원한다.
when에 스터빙할 메소드를 넣고 그 이후에 어떤 동작을 어떻게 제어할지를 메소드 체이닝 형태로 작성하면 된다.

| 메소드 명  | 설명                                                      |
| ---------- | --------------------------------------------------------- |
| when       | 스터빙 조건                                               |
| thenReturn | 스터빙한 메소드 호출 후 어떤 객체를 리턴할 건지 정의      |
| thenThrow  | 스터빙한 메소드 호출 후 어떤 Exception 을 Throw 할지 정의 |

## Captor

- 메서드에 전달된 인자를 캡처하는 역할
- 캡처할 인자의 타입에 해당하는 Captor객체를 생성해야
- Verify메서드로 메서드의 인자 캡처 가능

```
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

import org.junit.jupiter.api.Test;
import org.mockito.ArgumentCaptor;
import org.mockito.Captor;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import java.util.List;

class CaptorTest {

    @Mock
    private List<String> mockList;

    @Captor
    private ArgumentCaptor<String> argumentCaptor;

    @Test
    void testCaptor() {
        MockitoAnnotations.openMocks(this);

        // mockList의 add 메서드 호출
        mockList.add("Mockito");

        // 캡처 실행
        verify(mockList).add(argumentCaptor.capture());

        // 전달된 값 검증
        assertEquals("Mockito", argumentCaptor.getValue());
    }
}

```

### **📌 언제 무엇을 사용할까?**

- ✅ **Mock** → 외부 의존성을 제거하고 가짜 객체로 테스트하고 싶을 때
- ✅ **Spy** → 실제 객체를 사용하면서 일부 동작만 변경하고 싶을 때
- ✅ **Captor** → 특정 메서드의 인자로 전달된 값이 올바른지 검증하고 싶을 때

## inject Mock

- 의존성 주입을 @Mock이나 @Spy로 생성된 mock객체를 자동으로 주입해주는 어노테이션
- 예를 들어 service단을 테스트 하는 중이면
  - Service는 @InjectMock하고, Repository는 @Mock을 한다!

## Verify

- 스터빙한 메서드가 제대로 실행되는지 확인

| 메서드명   | 설명                      |
| ---------- | ------------------------- |
| times(n)   | 몇번 호출된건지           |
| never      | 한 번도 호출되지 않았는지 |
| atLeastOne | 최소한 한번은 호출된건지  |
| atLeast(n) | 최소 n번이 호출됐는지     |
| atMostOnce | 최대 한 번은 호출됐는지   |

## 참고자료

https://developer-nyong.tistory.com/33
https://velog.io/@choidongkuen/Mockito-%EB%A5%BC-%EB%BF%8C%EC%85%94-%EB%B4%85%EC%8B%9C%EB%8B%A4#-injectmocks
