# 시스템 콜

## 운영체제란?

운영체제는 사용자가 하드웨어 자원에 편리하게 사용할 수 있도록 도와주는 소프트웨어이다. 대표적으로 Windows, MacOS, Linux 등이 있다.   

컴퓨터가 발전하면서 점점 어렵고 복잡한 문제들을 다룰 수 있게 되면서, 이러한 문제를 편리하면서 효율적으로 처리할 수 있는 운영체제가 등장하게 되었다. 그래서 운영체제는 복잡한 문제를 해결하기 위한 다양한 기능을 제공한다.

### 운영체제가 제공하는 기능
![image](https://github.com/Ohjiwoo-lab/linux-kernel-study/assets/74577768/6e399d1b-6b9a-4842-9478-38c3da6c9958)   
출처: [홍공러들의 스터디공간](https://hongong.hanbit.co.kr/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C%EB%9E%80-%EC%BB%A4%EB%84%90%EC%9D%98-%EA%B0%9C%EB%85%90-%EC%9D%91%EC%9A%A9-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%A8-%EC%8B%A4%ED%96%89%EC%9D%84-%EC%9C%84%ED%95%9C/)

- 하드웨어 추상화: 사용자가 하드웨어를 전혀 알지 못하더라도 응용 프로그램을 실행하여 하드웨어를 제어할 수 있다. (아래에 등장하는 시스템 콜을 이용함)
- 다중화: 운영체제 상에서 여러 애플리케이션이 실행된다.
- 격리: 여러 애플리케이션은 독립적으로 실행된다. (서로 통신하려면 시스템 콜을 이용해야 함)
- 공유: 여러 사용자가 파일 시스템 등을 공유할 수 있다.
- 보안: 운영체제 내부에서 보안 기능도 제공한다.
- 성능: CPU의 독점을 방지하기 위해 시분할 시스템을 이용하여 성능을 향상시킨다.

이처럼 운영체제는 RAM에 적재되어 실행 중인 프로세스 관리, 메모리 할당, 파일 시스템 관리, 액세스 컨트롤, 네트워크 등 다양한 기능을 제공한다.


## 시스템 콜이란?

사용자가 하드웨어에 직접 접근하는 방식은 굉장히 위험하다. 여러 명의 사용자가 동일한 컴퓨터 자원을 사용하고 있다면, 더더욱 그럴 것이다. 하드웨어에 대한 자세한 정보 없이 하드웨어를 직접 다루다보면, CPU나 RAM 등 중요한 자원을 망가트려 사용하지 못하게 될 수도 있다.   

위와 같은 이유 때문에 사용자와 하드웨어 사이에 `운영체제`를 두고, 일반적인 사용자가 사용할 수 있는 명령과 운영체제가 사용할 수 있는 명령을 구분하였다. 이런 식으로 사용할 수 있는 명령을 구분한 것이 `이중 모드`이다.

### 이중 모드

cpu는 여러 가지 모드를 가지고 있다. 그 중 `유저 모드`와 `특권 모드`가 있다.

- 유저 모드: 일반 사용자가 응용 프로그램을 통해 명령어를 실행할 때의 모드
- 특권 모드: 운영체제 커널이 하드웨어에 직접 접근하기 위해 사용하는 모드

사용자는 절대 특권 모드에서 명령을 실행할 수 없다. 모든 명령은 유저 모드에서 실행되어야 하며, 운영체제가 제공하는 API인 `시스템 콜`을 통해서만 특권 모드에 진입할 수 있다.   

### 시스템 콜 동작 방식

```c
fd = open("out", 1);
write(fd, "hello\n", 6);
pid = fork();
```
위는 시스템 콜을 사용하는 예시이다. 일반적인 함수처럼 보이는 open, write, fork 같은 것들이 모두 시스템 콜이다. 시스템 콜이 호출되면 CPU에서 모드의 전환이 이루어진다. open()은 파일을 여는 시스템 콜이기 때문에, 파일을 열기 위해서는 하드웨어의 파일 시스템에 접근해야 한다. 사용자는 직접 접근할 수 없기 때문에 open()을 통해 운영체제에게 요청을 보내는 것이다.

![image](https://github.com/Ohjiwoo-lab/linux-kernel-study/assets/74577768/77cb9c5c-d3fd-4474-bbca-53d702f24861)   
출처: [The Linux Programming Interface](https://sciencesoftcode.files.wordpress.com/2018/12/the-linux-programming-interface-michael-kerrisk-1.pdf)

그림은 시스템 콜이 호출된 후의 실행 흐름을 나타낸다. 흐름을 요약하자면 다음과 같다.

1. 응용 프로그램에서 시스템 콜 execve() 호출
2. 유저 모드에 있던 레지스터 값과 Program Counter를 메모리에 저장
3. CPU를 특권 모드로 전환 (kernel page table과 kernel stack으로 변경)
4. 시스템 콜 핸들러에서 호출한 execve() 함수를 검색
5. 함수에 해당하는 C 코드로 분기
6. 코드 실행
7. 유저 모드로 복귀

시스템 콜이 어떻게 동작하는 지를 공부하기 위해서 [tlpi](https://man7.org/tlpi/code/download/) 소스코드를 이용할 것이다. 이는 GPL 라이선스기 때문에 누구나 확인할 수 있다.

해당 소스코드를 다운받은 후, `make` 명령을 통해 소스코드를 컴파일해준다.

![image](https://github.com/Ohjiwoo-lab/linux-kernel-study/assets/74577768/4c658145-45d0-40a8-9e28-a23e8d30a941)
![image](https://github.com/Ohjiwoo-lab/linux-kernel-study/assets/74577768/be9e9681-1f6a-4f79-a661-6dd680e4bab2)

`libtlpi.a` 파일이 생성되었다면 정상적으로 컴파일된 것이다.
![image](https://github.com/Ohjiwoo-lab/linux-kernel-study/assets/74577768/8bd1a69e-772b-4f38-aa49-50f0f28159b9)

디렉토리 중 하나인 `fileio`에 들어가보면, C 코드와 실행파일을 확인할 수 있다.
![image](https://github.com/Ohjiwoo-lab/linux-kernel-study/assets/74577768/1e60d89e-4a6e-4e97-9cc2-213f6b89737e)

이제 코드를 분석하면서 시스템 콜을 공부해나갈 것이다.