# ELF (Eexcutable and Linkable Format)

리눅스에서 C언어로 코드를 작성한 뒤, gcc 컴파일러를 이용하여 컴파일하고 나면 .exe 확장자를 가진 실행파일이 생성된다. ELF는 해당 실행 파일의 포맷이다.

![image](https://github.com/Ohjiwoo-lab/linux-kernel-study/assets/74577768/ad9d2932-a553-4441-b0ae-493ef3f4bee9)   
출처: [위키피디아](https://ko.wikipedia.org/wiki/ELF_%ED%8C%8C%EC%9D%BC_%ED%98%95%EC%8B%9D)   

ELF 포맷 형태는 다음과 같다.

- .text: 소스코드가 컴파일된 후에 어셈블리 명령어로 된 코드
- .rodata: 읽기 전용 데이터
    - `printf("hello");`라는 코드에서 `hello` 문자열이 .rodata에 저장된다.
- .data: 글로벌로 선언된 변수

## ELF 분석 도구 (Binutils)

리눅스에서 ELF 파일을 분석하기 위한 도구로 readelf, objdump, nm 을 지원해준다.

### readelf
실행파일(elf)를 분석해주는 도구이다. ELF 헤더와 프로세스 메모리 레이아웃의 섹션 요소 (.text, .rodata, .data, .bss)를 확인할 수 있다.

![image](https://github.com/Ohjiwoo-lab/cloud-project/assets/74577768/80a39f49-8645-4280-b0bf-4cff1ae21137)
![image](https://github.com/Ohjiwoo-lab/cloud-project/assets/74577768/6e40da3a-1824-4782-88fa-5fec962bd553)

### objdump   
이는 실행파일을 기계어로 번역해주는 도구이다. 이를 이용하여 어떤 라이브러리를 썼는지, 어떤 데이터가 어느 곳에 저장되었는 지 등을 기계어를 통해 자세하게 분석할 수 있다.

![image](https://github.com/Ohjiwoo-lab/cloud-project/assets/74577768/9b0e208a-d97c-417b-b00d-5f262b1f202c)

### nm
컴파일하고 난 후의 심볼 정보를 보여주는 도구이다. 엔트리포인트인 _end, _start, 실행된 함수 정보, 사용된 라이브러리 정보 등을 심볼 형태로 확인할 수 있다.

![image](https://github.com/Ohjiwoo-lab/cloud-project/assets/74577768/2f82722a-a761-494c-b2d7-342cc05ce2a9)

**참고사항**   

![image](https://github.com/Ohjiwoo-lab/cloud-project/assets/74577768/b3eee920-9cce-4dd5-b965-4739ff0dda4d)

`file` 명령어를 이용하면 파일의 형태를 간단하게 확인할 수 있다. 보통 해당 명령어를 많이 사용한다.


# 프로세스 메모리 레이아웃

ELF 파일이 실행을 위해 메모리에 적재되었을 때의 메모리 구조이다. ELF 파일이 메모리에 적재되면 이를 `프로세스`라고 부른다. 즉, 프로세스의 메모리 구조를 말한다.

![image](https://github.com/Ohjiwoo-lab/linux-kernel-study/assets/74577768/db5d00c7-5fb8-4dbb-9418-c656e71832ae)   
출처: [The Linux Programming Interface](https://sciencesoftcode.files.wordpress.com/2018/12/the-linux-programming-interface-michael-kerrisk-1.pdf)   

다음과 같은 요소로 이루어져있다.

- text: 어셈블리 코드와 읽기 전용 데이터
- data: 글로벌 변수
    - Uninitialized data (bss)와 Initialized data로 구분된다.
- stack: 함수의 지역 변수
- heap: malloc으로 동적 메모리 할당 시 할당되는 공간

가상 메모리 4G 공간 중에 처음 1GB는 커널을 위해 할당되고, 나머지 3GB에 text, data, stack, heap 영역이 할당된다. 이때 stack과 heap은 서로 마주보면서 자라나게 된다.

![image](https://github.com/Ohjiwoo-lab/linux-kernel-study/assets/74577768/0c0b2aa3-3a2d-4a3a-9070-8b900c512bcc)

실제 코드를 보면서 각 변수들이 어디 영역에 할당되는지 분석해보면 다음과 같다.

- globBuf: bss 영역
- primes: 초기화된 data 영역
- result: square 함수의 지역변수이므로 stack 영역
    - 리턴되는 값은 별도의 정해진 레지스터에 저장된다.
- t: doCalc 함수의 지역변수이므로 stack 영역
- key: static으로 선언되어 글로벌 변수인데, 초기화되었으므로 초기화된 data 여역
- mbuf: 이 역시 static으로 선언되었지만 초기화되지 않았으므로 bss 영역
- *p: main의 지역변수이므로 stack 영역
- malloc(): 동적 할당이므로 heap 영역

