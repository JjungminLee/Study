[https://selog.tistory.com/entry/가상화-가상머신VM과-하이퍼바이저-쉽게-이해하기](https://selog.tistory.com/entry/%EA%B0%80%EC%83%81%ED%99%94-%EA%B0%80%EC%83%81%EB%A8%B8%EC%8B%A0VM%EA%B3%BC-%ED%95%98%EC%9D%B4%ED%8D%BC%EB%B0%94%EC%9D%B4%EC%A0%80-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

https://suyeon96.tistory.com/54

https://suyeon96.tistory.com/52

https://docs.openstack.org/arch-design/design-compute/design-compute-overcommit.html

https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/chap-overcommitting_with_kvm

## 가상화

- OS를 배웠을때 내가 원래 가지고 있는 것 보다 더 많아보이는 것처럼 보이게 속이는 것!
- 가상화를 VM, Container로 구현함
- VM은 하이퍼바이저를 이용해 리소스 전체를 가상화
- 컨테이너는 OS수준에서 프로세스를 컨테이너 형태로 격리

## 하이퍼바이저

![image.png](attachment:fd4f687b-b1ff-44ad-ba82-0aa802e81c1b:image.png)

- 물리적인 하드웨어를 논리적으로 가상화하는데 이것을 담당하는 애가 하이퍼바이저

### 하이퍼바이저 vs 컨테이너

![image.png](attachment:25cfa317-d697-4e9a-be06-25122264779e:image.png)

- VM
  - 하이퍼바이저를 사용해 컴퓨터가 많아보이게 한다 (VM별로 OS도 다르게 설정가능하다!)
- Container
  - 하이퍼바이저 없고 OS도 동일한데 컨테이너 엔진으로 컴퓨터가 많아보이게 한다

### 종류

- Type1 하이퍼바이저
  - Native/Baremetal Hypervisor
  - 베어 메탈 하드웨어에 직접 설치되어 구동됨
    - Xen,KVM
    - cf) Baremetal
      - 운영체제나 가상화 소프트웨어등 어떤 소프트웨어도 설치되지 않은 순수 물리 서버
- Type2 하이퍼바이저
  - Hosted 하이퍼바이저
  - 베어메탈 하드웨어 위에 Host OS가 설치되고, 그 위에 하이퍼바이저가 실행딤
  - Oracle Virtual Box, Vmware Workstation
- 실제 CSP들은 Type1 하이퍼바이저를 사용함

## Xen과 KVM

- XEN
  - AWS, AMD, NaverCloud 등에서 사용하는 오픈소스 하이퍼바이저
  - 대표적인 반가상화 하이퍼바이저
- KVM
  - 대표적인 전가상화 하이퍼바이저
  - CPU에서 HVM(하드웨어 기반 가상화) 기능을 제공해야함
  - 하드웨어 기반 가상화를 위해 emulation이 필요한데 이를 QEMU가 수행함
  - QEMU로 OS이미지 실행하면 QEMU가 KVM에 요청하고 이 과정에서 vcpu등이 생성됨

![image.png](attachment:f482cb4a-4e85-486d-8766-e3ca01697c4a:image.png)

## OverCommit

https://brunch.co.kr/@leedongins/83

https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/virtualization_deployment_and_administration_guide/sect-overcommitting_with_kvm-overcommitting_memory

https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/6/html/virtualization_administration_guide/form-virtualization-overcommitting_with_kvm-overcommitting_virtualized_cpus

- 물리적 서버에서 사용가능한 양보다 더 많은 메모리를 할당할 수 있음
  - 대부분의 프로세스는 항상 할당된 리소스의 100%에 엑세스 하지 않기에 가능함
  - KVM 하이퍼바이저는 CPU 및 메모리를 자동으로 Overcommit한다!
- CSP의 경우 컴퓨팅 노드에 CPU와 RAM을 실제 용량보다 더 크게 할당 가능하다. 클라우드 시스템에서 제공하는 프로세서를 virtual CPU라고 한다.
  - 인스턴스 성능이 줄어든 비용 만큼 CSP는 가진 것 보다 더 많은 인스턴스를 고객에게 제공하게 된다. 이것을 OverCommit이라고 함!
- 하이퍼바이저가 Overcommit을 수행하고
  - memory overCommit, cpu overCommit이 가능하다!

## OverCommit Ratio

https://docs.openstack.org/arch-design/design-compute/design-compute-overcommit.html

- 오픈 스택의 경우 CPU는 16:1, RAM은 1.5:1
- CPU 물리코어 하나당 오버커밋을 16번 한다는 것 → 즉, cpu 1개를 가지고 16개가 있다는 환상을 심어주는것!
- 비율로 따지면 1600%

### 메모리 OverCommit

- RAM의 경우, 평균 부하율이 CPU보다 높기 때문에 overcommit 비율을 낮게 책정하는 것이 일반적
- 메모리의 자원 충돌이 발생하는 경우 Swap을 통해 문제가 해결되기에 인스턴스 성능이 떨어짐

### CPU OverCommit

- OverCommit 비율이 너무 크면 인스턴스간 자원 충돌 문제가 발생할 수 있다!

## HA Ratio

- 물리 서버가 죽으면 다른 서버로 넘어가야하는데, 그 비율

## CSP입장에서 가용량

- 운영체제에서 가상화는 크게 두가지 → CPU 가상화, 메모리 가상화
- CPU는 생각보다 가득차기가 쉽지 않음
  - CPU는 짧고 폭발적인 사용패턴을 보이는 경우가 많아서 VM이 동시에 CPU 100%를 사용하는 경우는 극히 드물다!
  - 흔히 배우는 프로세스 스케쥴링 기법 이를 테면 라운드로빈, MLFQ같이 CPU는 타임 슬라이스를 쪼개서 쓰기 때문에 100% 쓰는 일은 작정해서 쓰는것 아닌 이상 쉽지 않다
- 왜 메모리는 가득찰까?
  - 메모리는 한번 채우면 방을 잘 빼지 않는다.
- 만약 메모리가 가득찬다면?
  - 디스크에 갔다오는 시간이 오래 걸린다.
  - 사용량이 적은 메모리의 페이지를 디스크에 스왑이라는 곳에 옮겨두는데 ⇒ 스와핑도 오래걸림
  - 시간이 오래걸리면 사용자 입장에서는 성능이 낮아지고 오래 기다려야하기 때문에 서버를 증설해줘야한다!
- **고로 CSP도 CPU는 오버커밋하더라고 메모리는 오버커밋하지 않는다!**
