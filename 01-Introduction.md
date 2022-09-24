# 환경 설정
* hypervisor: [hyper-v gen 1](https://en.wikipedia.org/wiki/Hyper-V)
    * virtual switch setting을 External network로 설정하면 공유기 DHCP 기능 사용하여 쉽게 고정 IP 생성 가능
* OS: [ubuntu-12.04 LTS](https://old-releases.ubuntu.com/releases/12.04/ubuntu-12.04-desktop-i386.iso)
* x86 emulator: [bochs-2.6.2](https://en.wikipedia.org/wiki/Bochs)

## ubuntu-12.04 LTS 설치
는 알아서

## 패키지 설치

### repository 변경
지원이 끝난지 한참 지난 버전이므로 패키지 관리자가 보는 repository 주소를 old-releases로 바꿔야 한다.
```
sudo sed -i -re 's/([a-z]{2}\.)?archive.ubuntu.com|security.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
sudo apt-get update && sudo apt-get upgrade -y
```

### openssh server 설치
```console
sudo apt-get install openssh-server
```

### 방화벽 설정
```console
ufw allow ssh
```

> 여기까지 끝나면 ssh 로 붙어서 편하게 작업 가능

### gcc-4.4 설치
```console
sudo apt-get install gcc-4.4
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.4 50
```

### bochs-2.6.2 설치
```console
curl -O https://src.fedoraproject.org/lookaside/pkgs/bochs/bochs-2.6.2.tar.gz/82ecaff9826d4f29fa46f3062e2957b8/bochs-2.6.2.tar.gz
tar xvf bochs-2.6.2.tar.gz
```
> 파일 백업 : https://github.com/kuro11pow2/pintos/blob/main/install/bochs-2.6.2.tar.gz

```console
cd bochs-2.6.2
./configure --enable-gdb-stub --with-nogui
make
sudo make install
```


### qemu 설치
```console
sudo apt-get install qemu
sudo ln -s /usr/bin/qemu-system-i386 /usr/bin/qemu
```

## pintos 설치 및 테스트
### pintos 코드 다운로드
```console
cd ..
curl -O pintos.tar.gz http://bowbowbow.tistory.com/attachment/cfile9.uf@216A3B4956D876CB03A881.gz
tar xvf pintos.tar.gz
```
> 파일 백업 : https://github.com/kuro11pow2/pintos/blob/main/install/pintos.tar.gz

### pintos 컴파일
```console
cd pintos/src/threads
make
```

### pintos 환경 변수 등록
```console
echo -e '\n'# pintos | sudo tee -a ~/.bashrc
echo export PATH="\$PATH:/home/$USER/pintos/src/utils" | sudo tee -a ~/.bashrc
source ~/.bashrc
```

### pintos 테스트
```console
cd ~/pintos/src/threads
pintos -- run alarm-multiple
```

# 소개

## Pintos File System 사용 및 user program 실행

### userprog 빌드
```console
cd ~/pintos/src/userprog/
make
cd build
```

### 파일 시스템 디스크 생성
```console
pintos-mkdisk filesys.dsk --filesys-size=2
```
* `pintos-mkdisk` : pintos에서 제공하는 가상 디스크 생성 도구.
* `filesys.dsk` : 파일 시스템 디스크 이름 (다른 이름 사용 불가)
* `--filesys-size` : 디스크의 용량 지정

### 파일 시스템 디스크 초기화
```console
pintos –f -q
```
* `-f` : filesys.dsk 포맷 옵션. pintos의 자체 파일시스템 구조로 포맷 진행.
* `-q` : 모든 작업이 끝나면 pintos를 종료하게 하는 옵션.

### examples 빌드 
```console
cd ~/pintos/src/examples/
make
cd ../userprog/bulid
```
`pwd`, `echo`, `shell`, `rm`, `halt`, `cat` 등등 여러 명령어 빌드

### 실행할 응용 프로그램을 파일 시스템 디스크에 복사
```console
pintos –p ../../examples/echo –a echo -- -q
```
* `-p` : 시스템 디스크로 복사, -a 옵션 : 프로그램 이름 설정
* `--` : Pintos 프로그램에 전달할 인자와 Pintos에서 동작하는 가상 OS에 전달할
인자를 구분하기 위해 사용
* 실행 프로그램은 Regular ELF executable 파일 형식으로 만들어져야 함

### 프로그램 실행
```console
pintos –q run 'echo x'
```
* `run` : 응용 프로그램을 실행
* pintos의 현재 상태로는 응용 프로그램이 실행되지 않음

### 결합된 명령어
```console
pintos --filesys-size=2 –p ../../examples/echo –a echo -- -f
–q run 'echo x'
```
* 앞의 모든 옵션을 한번에 넣어서 프로그램을 실행할 수 있음.
* 옵션은 반드시 -p, -a, -- , -f, -q, -run 순서로 실행되어야 함.



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

### 예제
* 파일 vs 파일 비교하여 patch 파일 생성 예시
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
