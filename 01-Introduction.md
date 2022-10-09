# 목차
- [목차](#목차)
- [Source Tree Overview](#source-tree-overview)
- [Files in build directory](#files-in-build-directory)
  - [참고](#참고)
- [Pintos File System 사용 및 user program 실행](#pintos-file-system-사용-및-user-program-실행)
  - [userprog 빌드](#userprog-빌드)
  - [파일 시스템 디스크 생성](#파일-시스템-디스크-생성)
  - [파일 시스템 디스크 초기화](#파일-시스템-디스크-초기화)
  - [examples 빌드](#examples-빌드)
  - [실행할 응용 프로그램을 파일 시스템 디스크에 복사](#실행할-응용-프로그램을-파일-시스템-디스크에-복사)
  - [프로그램 실행](#프로그램-실행)
  - [결합된 명령어](#결합된-명령어)
- [Patch 적용](#patch-적용)
  - [Patch 파일 만들기](#patch-파일-만들기)
    - [파일 vs 파일 비교하여 patch 파일 생성](#파일-vs-파일-비교하여-patch-파일-생성)
  - [Patch 파일 적용하기](#patch-파일-적용하기)


# Source Tree Overview
`pintos/src` 디렉토리의 구조를 설명한다.

* `threads/`
  * 커널을 구성하는 기초 코드 (프로젝트1)
* `userprog/`
  * 유저 프로그램 로더 코드 (프로젝트2)
* `vm/`
  * virtual memory 코드 (프로젝트3)
  * 현재 거의 비어있음
* `filesys/`
  * 파일 시스템 코드 (프로젝트4)
* `devices/`
  * I/O 인터페이스 코드 (프로젝트1)
  * 타이머 구현 제외하면 수정할 필요 없음
* `lib/`
  * 표준 C 라이브러리의 부분집합 구현. 커널과 유저 프로그램 모두에서 `include` 되며 빌드됨.
* `lib/kernel/`
  * PintOS 커널에만 포함되는 C 라이브러리.
  * 커널 코드에서 사용하는 자료구조가 구현되어 있음. 
  * bitmaps, doubly linked lists, hash table
* `lib/user/`
  * PintOS 유저 프로그램에만 사용되는 C 라이브러리
* `tests/`
  * 각 프로젝트를 위한 테스트
* `examples/`
  * 프로젝트2를 시작하기 위한 유저 프로그램 예제
* `misc/`, `utils/`
  * 로컬 머신에서 Pintos로 작업할 때 유용한 파일들

# Files in build directory
`threads/`에서 `make` 명령어로 빌드하면 그 결과물은 `threads/build`에 생성됨

* `Makefile`
  * `pintos/src/Makefile.build`의 복사본. 커널이 어떻게 빌드되었는지 서술
* `kernel.o`
  * 전체 커널에 대한 오브젝트 파일
  * 개별 커널 소스 파일로부터 컴파일된 오브젝트 파일을 링킹한 결과물
  * 디버그 정보를 포함하므로 GDB나 backtrace로 실행할 수 있음
* `kernel.bin`
  * 커널의 메모리 이미지
  * Pintos 커널을 실행하기 위해 메모리에 로드된 바이트
  * 디버그 정보가 제거된 `kernel.o`와 동일하다
  * 커널 로더 설계 상 512KB 제한이 있는데 이를 초과하지 않게 디버그 정보를 제거하여 로드한다.
* `loader.bin`
  * 어셈블리로 작성된 커널 로더의 메모리 이미지
  * PC BIOS 제약으로 512 bytes로 고정됨
  * 커널을 디스크에서 메모리로 로드하고 시작 시킴 

## 참고
* `*.o`
  * 오브젝트 파일
  * 컴파일러를 통해 기계어로 변환된 파일
* `*.d`
  * 의존성(dipendency) 파일
  * 다른 소스나 헤더 파일이 변경되었을 때 어떤 파일이 재컴파일되어야 하는지 `make`에게 알려주는 파일

# Pintos File System 사용 및 user program 실행

## userprog 빌드
```console
cd ~/pintos/src/userprog/
make
cd build
```

## 파일 시스템 디스크 생성
```console
pintos-mkdisk filesys.dsk --filesys-size=2
```
* `pintos-mkdisk` : pintos에서 제공하는 가상 디스크 생성 도구.
* `filesys.dsk` : 파일 시스템 디스크 이름 (다른 이름 사용 불가)
* `--filesys-size` : 디스크의 용량 지정

## 파일 시스템 디스크 초기화
```console
pintos –f -q
```
* `-f` : filesys.dsk 포맷 옵션. pintos의 자체 파일시스템 구조로 포맷 진행.
* `-q` : 모든 작업이 끝나면 pintos를 종료하게 하는 옵션.

## examples 빌드 
```console
cd ~/pintos/src/examples/
make
cd ../userprog/bulid
```
`pwd`, `echo`, `shell`, `rm`, `halt`, `cat` 등등 여러 명령어 빌드

## 실행할 응용 프로그램을 파일 시스템 디스크에 복사
```console
pintos –p ../../examples/echo –a echo -- -q
```
* `-p` : 시스템 디스크로 복사, -a 옵션 : 프로그램 이름 설정
* `--` : Pintos 프로그램에 전달할 인자와 Pintos에서 동작하는 가상 OS에 전달할
인자를 구분하기 위해 사용
* 실행 프로그램은 Regular ELF executable 파일 형식으로 만들어져야 함

## 프로그램 실행
```console
pintos –q run 'echo x'
```
* `run` : 응용 프로그램을 실행
* pintos의 현재 상태로는 응용 프로그램이 실행되지 않음

## 결합된 명령어
```console
pintos --filesys-size=2 –p ../../examples/echo –a echo -- -f
–q run 'echo x'
```
* 앞의 모든 옵션을 한번에 넣어서 프로그램을 실행할 수 있음.
* 옵션은 반드시 -p, -a, -- , -f, -q, -run 순서로 실행되어야 함.



# Patch 적용
## Patch 파일 만들기
Patch 파일 생성은 'diff' 명령을 사용
```console
$ diff [옵션] [원본파일] [수정된파일] > 출력파일.patch
$ diff [옵션] [원본디렉토리] [수정된디렉토리] > 출력파일.patch
```

주요 옵션(`–urN`)
* `-u` : 수정된 부분의 앞뒤 동일한 내용을 3줄만 표시
    * `--unified=2` 을 사용하면 동일한 라인을 2줄만 표시
* `-r` : recursive. 재귀적 검색, 하위 디렉토리를 모두 검색하여 비교
* `-N` : new file. 두 디렉토리 중 어느 한 디렉토리에만 파일이 있는 경우, 두 디렉
토리 모두에 같은 파일이 존재한다고 가정하고 비교를 진행
* 자세한 옵션은 `$ diff --help`로 확인

### 파일 vs 파일 비교하여 patch 파일 생성
```console
diff -u process.c process_new.c > process.c.patch
```
* 디렉토리 vs 디렉토리 비교하여 patch 파일 생성 예시( -r 옵션 사용)
```console
diff -urN pintos/ pintos_new/ > pintos.patch
```


## Patch 파일 적용하기
* Patch 파일 적용은 `patch` 명령을 사용
```console
patch [옵션] < 결과파일.patch
```
* 주요 옵션
    * `-p` : `-pNUM` 형식으로 사용(NUM=숫자)
    * NUM은 patch에 명시된 파일 경로에서 NUM 만큼 하위 디렉터리로 이동하여 patch를 적용하라는 의미
    * Ex) patch 파일 안에 `--- driver/src/display.c` 라는 경로가 있는 경우, `-p1` 을 사용하면 `src/display.c` 경로에 위치한 파일에 해당 patch를 적용
* 패치 파일 적용 예시
```console
patch -p0 < pintos.patch
```

