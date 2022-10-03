# 목차
- [목차](#목차)
- [패키지 설치](#패키지-설치)
  - [repository 변경](#repository-변경)
  - [openssh server 설치](#openssh-server-설치)
  - [방화벽 설정](#방화벽-설정)
  - [gcc-4.4 설치](#gcc-44-설치)
  - [bochs-2.6.2 설치](#bochs-262-설치)
  - [qemu 설치](#qemu-설치)
- [pintos 설치 및 테스트](#pintos-설치-및-테스트)
  - [pintos 코드 다운로드](#pintos-코드-다운로드)
  - [pintos 컴파일](#pintos-컴파일)
  - [pintos 환경 변수 등록](#pintos-환경-변수-등록)
  - [pintos 테스트](#pintos-테스트)

# 패키지 설치

## repository 변경
지원이 끝난지 한참 지난 버전이므로 패키지 관리자가 보는 repository 주소를 old-releases로 바꿔야 한다.

```console
sudo sed -i -re 's/([a-z]{2}\.)?archive.ubuntu.com|security.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
sudo apt-get update && sudo apt-get upgrade -y
```

## openssh server 설치
```console
sudo apt-get install openssh-server
```

> hyper-v 사용한 guest os 사용하는 경우 IP가 계속 바뀌기 때문에 ssh 서버로 접속해서 사용하기 불편할 수 있으나, `virtual switch setting`을 `External network`로 설정하면 공유기 DHCP 기능을 사용하므로 IP를 고정할 수 있다.

## 방화벽 설정
```console
ufw allow ssh
```

> 이후부터는 ssh로 접속하여 작업 진행한다.

## gcc-4.4 설치
```console
sudo apt-get install gcc-4.4
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.4 50
```

## bochs-2.6.2 설치
```console
curl -LJo bochs-2.6.2.tar.gz https://github.com/kuro11pow2/pintos/blob/main/install/bochs-2.6.2.tar.gz?raw=true
tar xvf bochs-2.6.2.tar.gz
```
> 출처: https://src.fedoraproject.org/lookaside/pkgs/bochs/bochs-2.6.2.tar.gz/82ecaff9826d4f29fa46f3062e2957b8/bochs-2.6.2.tar.gz

```console
cd bochs-2.6.2
./configure --enable-gdb-stub --with-nogui
make
sudo make install
```


## qemu 설치
```console
sudo apt-get install qemu
sudo ln -s /usr/bin/qemu-system-i386 /usr/bin/qemu
```

# pintos 설치 및 테스트
## pintos 코드 다운로드
```console
cd ..
curl -LJo pintos.tar.gz https://github.com/kuro11pow2/pintos/blob/main/install/pintos.tar.gz?raw=true
tar xvf pintos.tar.gz
```
> 출처: http://www.stanford.edu/class/cs140/projects/pintos/pintos.tar.gz

## pintos 컴파일
```console
cd pintos/src/threads
make
```

## pintos 환경 변수 등록
```console
echo -e "\n# pintos" | sudo tee -a ~/.bashrc
echo export PATH="\$PATH:/home/$USER/pintos/src/utils" | sudo tee -a ~/.bashrc
source ~/.bashrc
```

## pintos 테스트
```console
cd ~/pintos/src/threads
pintos -- run alarm-multiple
```