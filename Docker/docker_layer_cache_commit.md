https://malwareanalysis.tistory.com/236

- 빌드 캐시

### Docker Build Cache

- 빌드 중에 동일한 이미지 레이어가 있다면 빌드하지 않고 재사용
- 빌드속도 향상과 중복된 이미지 레이어 저장을 막음으로써 이미지 레이어 저장공간 최소화

- 과정
  1. 빌드 수행 (이미지 레이어가 없기 때문)
  2. 이미지 레이어가 생성되고 레이어 저장공간에 저장됨
     1. 각 레이어가 docker commit을 수행하여 도커 이미지 생성
        1. 궁금한점
           1. NCP source build의 경우 매번 docker commit을 해서 캐싱의 장점을 하나도 못쓰고 있음
  3. 첫번째 도커 빌드 이후에 두번째 도커 빌드 수행시, 레이어 저장공간에 이전에 생성한 동일한 이미지 레이어가 있기에 빌드를 수행하지 않고 이미지 레이어 재사용
  4. 만약 상위 이미지 레이어가 변경되면 하위 이미지 레이어가 같을지라도 캐시를 사용하지 못함
- 이미지 레이어 수정이 빈번하다면
  - 해당 docker run 명령어는 최대한 하단에 쓰는게 중요함

### Docker Image Layer

https://hackerpark.tistory.com/entry/Docker-image%EB%A5%BC-%EA%B5%AC%EC%84%B1%ED%95%98%EB%8A%94-image-layer#google_vignette

https://malwareanalysis.tistory.com/234

- 도커 이미지
  - 특정 환경, 파일, 라이브러리등을 실행할 수 있는 컨테이너를 생성하기 위한 파일
    - VM으로 치면 iso
    - image하나 가지고 여러 컨테이너 만드릭도 가능
- Image Layer
  - container를 실행하기 위한 요소들을 효율적을 관리하기 위해 Layer 사용
  - 컨테이너 생성하기 위해 OS부터 필요한 라이브러리와 실행파일들을 Image에 모두 담는데 이 모든 것을 하나의 파일로 관리하면 비효율
  - Layer를 사용하면 Image를 구성하는 파일 전체를 수정하는게 아니라 마지막으로 수정된 Layer만 변경 가능하다!

![alt text](image.png)

### 멀티 스테이징 빌드

https://docs.docker.com/build/building/best-practices/

https://kimjingo.tistory.com/63

https://hgko-dev.tistory.com/522

- 컨테이너 이미지를 만들면서 빌드에는 필요하지만 최종 컨테이너 이미지에는 필요 없는 환경을 제거할 수 있도록 단계를 나누어 기반 이미지를 만드는 방법

![alt text](image-1.png)

컨테이너 실행 시에는 빌드에 사용한 파일 및 디렉토리와 같은 의존 파일들이 모두 삭제된 상태로 실행됨 → 더 가벼운 크기의 컨테이너 사용가능

### 1.5GB → 895.66MB로 줄여보기!

```jsx
FROM python:3.9 as builder

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir --progress-bar off -r requirements.txt

FROM python:3.9-slim

WORKDIR /app

COPY ./app /app

COPY requirements.txt .
RUN pip install --no-cache-dir --progress-bar off -r requirements.txt && \
    rm requirements.txt

EXPOSE 8000
```
