# 임베디드 리눅스 커널 포팅 및 구조
## Day1
### 개발환경 구성
- Vmware Workstation 17 Player
- Ubuntu14.04.05 vmwarevm 
  - VMware virtual machine config(.vmx) -> VMware에서 오픈
  - VMware virtual disk file
- 리눅스에서 인식할 수 있도록 XML 수정 -> device vid, pid 설정
- tftp directory만 target board에서 host에 접근할 수 있음
- 리눅스에서 파일명에 . 이 붙으면 숨김 파일임
- root 권한이 필요한 디렉토리에 접근 시 sudo 명령어 사용

### TFTP
- 임베디드 보드(Target)가 호스트로부터 이미지를 다운로드 할 때 사용하는 프로토콜

### NFS (Network File System)
- 임베디드 리눅스 환경에서 스토리지(Disk)를 관리하는 프로토콜
- 네트워크를 통해 파일 시스템을 mount할 수 있게 해준다.
- 호스트의 Remote Storage에 저장해두고 사용하는 방법
- 외부에 있는 Disk 장치를 본인 것처럼 사용 가능
- 개발 단계에서 용이하지만, 네트워킹이 필요없는 제품의 경우 최종적으로는 타겟 보드의 NAND Flash에 모두 저장한다.

### Toolchain
- 타겟 시스템에서 실행할 프로그램 개발을 위해 소스코드를 컴파일하고 링킹하며 실행가능한 파일로 만들어주는 개발도구를 통칭
- 크로스컴파일러 제작을 위해 활용됨
- [툴체인(Toolchain)이란?](https://kkhipp.tistory.com/176)

### Root File System
- 리눅스 동작에 필요한 기능과 시스템 초기화 및 주변 장치 제어를 위한 부팅 과정에 필요한 프로그램들과 자료를 관리하는 파일 시스템
- [루트 파일 시스템 구성](https://m.blog.naver.com/ibank97/221971880145)

---
## Day2
### Bootloader
- 운영체제가 시동되기 이전에 미리 실행되는 프로그램 (운영체제 시동 목적)
- 하드디스크이 첫번째 섹터에 위치
- U-BOOT(임베디드 리눅스), NTLDR(Window), GRUB(PC 리눅스)

### BSP(Board Support Package)
- 부트로더 소스 및 이미지 파일
- 리눅스 커널 소스, 디바이스 드라이버 소스 및 이미지 파일
- 램디스크 이미지 파일
- Cross Compiler Tool Package(gcc)

### U-Boot 소스코드 분석
- start_armboot(void) => main 함수
- 어셈블리 코드(.s)는 하드웨어 초기화도 하지만, C언어 소스코드를 호출해서 부팅을 진행할 수도 있다.

### Makefile
- 규모가 큰 프로그램의 경우 여러 개의 모듈로 나누어 프로그램을 개발하게 된다.
- 이 때, 입력 파일이 바뀌면 바뀐 파일과 관계 있는 다른 파일들이 다시 컴파일이 되어야 하는데, 이러한 과정을 쉽게 할 수 있도록 해주는 파일이다.
- [Makefile이 뭘까?](https://velog.io/@woodstock1993/Makefile)


### Vector Table
- 프로그램 맨 시작부분에 위치하여 프로그램이 동작 중에 문제가 발생하거나 인터럽트가 걸렸을 때 점프하는 부분이다.
- [Vector Table](https://velog.io/@audgus47/%EB%B2%A1%ED%84%B0-%ED%85%8C%EC%9D%B4%EB%B8%94Vector-Table)

### Porting
- 호스트에서 제작한 프로그램을 타겟 보드에서 실행될 수 있게 만들기 위해 요구되는 작업
- SoC 포팅
  - Reference 보드의 SoC와 target 보드의 SoC 차이점을 파악하고 소스코드에 반영하는 작업

### U-boot 포팅 실습
- 삼성에서 제공하는 smdk2450 용 부트로더를 Reference로 활용한다.
- 컴파일러를 수정하고 mds2450 보드용 디렉터리를 추가한다.
- mds2450과 유사한 시스템 보드의 디렉터리와 configuration 파일을 이용하여 mds2450에 맞도록 수정한다.
- 리눅스 커널을 메모리에 올리기 전이므로 U-boot가 메모리에서 차지하는 위치는 테스트 시에는 상관이 없으나, 리눅스 커널 자리를 미리 확보해두고 U-boot 위치를 정하는 것이 좋다.
- DRAM (64MB) 메모리 주소 범위
  - 0x30000000 ~ 0x33ffffff

### MMU(Memory Management Unit)
- 가상 메모리 주소 -> 물리 메모리 주소 변환
- 메모리 보호

### 커널 이미지 구조
- zImage: 압축된 형태
- 압축을 풀기 위한 head.S, 커널 시작을 위한 head.S 존재

---
## Day3
### 부팅 과정 이해
1. Linux Kernel Start
2. SVC 모드 전환 및 인터럽트 중지
3. 프로세서 정보 검색
4. 머신 아키텍처 정보 검색
5. 페이지 테이블 설정
6. MMU 및 Cache 설정
7. Stack 및 시스템 정보 설정
8. BSS clear
9. 시스템 정보 저장
10. start_kernel() 호출(C언어)
  - 리눅스 커널 부팅의 거의 모든 과정이 이루어짐
- MMU에 의해 메모리 보호
  - 커널(SVC) 모드: 모든 메모리 접근 가능
  - 사용자(User) 모드: 허용된 메모리만 접근 가능

### init kernel thread
- root file system mount
- /dev/console open
  - stdin
  - stdout
  - stderr

### 리눅스 디렉터리 구성
- /bin, /sbin, /usr/bin, /usr/local/bin
  - 실행파일이 존재하는 공간
- /etc
  - 시스템 환경 정보가 존재하는 공간
- /root
  - root 사용자가 사용하는 기본 디렉터리
- /usr, /usr/local/lib, /usr/local/include
  - 윈도우의 Program files 디렉터리와 같은 역할
  - 설치 프로그램이 존재하는 공간
- /lib
  - 커널 모듈(kernel object)이 존재하는 공간
  - 기본 라이브러리 위치
- /boot
  - 부트로더가 위치하는 공간 (부트 섹터)
- /dev
  - 장치(하드웨어)를 다루기 위한 진입점 역할
  - devtmpfs : 사용되는 파일 시스템
- /proc
  - procfs: 사용되는 파일 시스템
  - 커널 정보가 존재하는 공간
- /sys
  - device driver 정보가 존재하는 공간
  - sysfs: 사용되난 피일 시스템
- /mnt
  - mount 된 파일들이 존재하는 공간

### 커널 빌드 방법
```
build_kernel clean
  # build 파일 삭제
  # config 파일도 삭제됨
  # make mrproper
```
``` 
./build_kernel config
  # 새롭게 config를 만들어주어야 함
  # make menuconfig
```
```
./build_kernel all /tftpboot
```
```
make ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabi- zImage modules
```
- module을 만드는 정식 명령어
- zImage 는 /arch/arm/boot/ 에 만들어짐
```
make INSTALL_MOD_PATH=$SYSROOT1 modules_install
```
- root 폴더에 .ko를 복사하여 붙여넣기

### 커널 포팅 실습
1. 리눅스 커널 소스를 다운받고 압축을 해제한다.
2. linux-3.0.22/Makefile 수정
  - ARCH와 CROSS_COMPILE 변수 수정
  - CROSS_COMPILE 변수는 해당 툴체인의 cross prefix에 알맞게 지정
3. 타겟 보드와 유사한 보드 선택 후 수정
  - 표준 커널에 포함된 S3C2416 기반 보드들 가운데 SMDK2416 보드가 MDS2450 보드와 가장 유사
  - smdk2416 설정 로드 후 mds2450 보드 관련 설정으로 수정
```
make s3c2410_defconfig
cd arch/arm/mach-s3c2416
cp mach-smdk2416.c mach-mds2450.c
```
4. mach-mds2450.c 편집
  - smdk2416 문자열(함수명, 변수)을 모두 mds2450으로 수정
  - MACHINE_START(MDS2450, "MDS2450") 수정
5. arch number를 가지고 동작하도록 수정
  - arch/arm/tools/mach-types에 추가
```
mds2450   MACH_MDS2450    MDS2450   3495
```
6. 커널 옵션 설정
- 타겟 보드 추가
- Kconfig는 Makefile과 연동되고, Makefile은 .config를 만들기위해 사용된다.
- Kconfig는 menuconfig에 어떻게 나타날지를 결정
- menuconfig에서 커널 옵션 설정 (제일 까다로움)

7. 커널 부트 테스트
- 빌드가 완료되면 arch/arm/boot/zImage가 생성되고, 생성된 파일을 /tftpboot로 복사한다.
```
cp arch/arm/boot/zImage /tftpboot
```
- Bootloader에서 새로운 커널 이미지를 로드하고, 부팅을 확인한다.
```
MDS2450# tftp 32000000 zImage;bootm 32000000
```

8. Device 설정
- 7번까지의 포팅 과정은 최소한으로 필수적인 것들만!
- 필요한 디바이스 드라이버를 포팅하는 단계

---
## 알아두면 좋은 내용
### 백업 방법
1. Linux shut down
```
sudo shutdown -h now
```
2. 압축
- Ubuntu14.04.05 - 복사본.vmx
- Ubuntu14.04.05.vmx
- Ubuntu14.04.05-dis1.vmdk


#### 명령어
```
md5sum linux-kernelporting.tar.gz
  - 파일의 유효성 검사
```
```
cd / 
  - 최상위 디렉토리로 이동
  - 리눅스 최상위 디렉토리
```
```
tar zxf linux-kernelporting.tar.gz
tar jxf linux-gnu.tar.bz2
  - 압축 해제
  - 세미콜론(;)으로 명령문 구분 역할로 여러개 명령문 한번에 입력 가능
```
```
sudo mv arm-2010q1/ /usr/local/
  - arm-2010q1 폴더 경로 이동
```
```
ping 8.8.8.8
  - 네트워크 상태 점검
  - 구글 도메인(8.8.8.8)
```
```
grep -rni tftp /etc
  - /etc 하위경로에 tftp 문자열이 들어간 파일들을 대소문자 구분하지 않고 찾는다.
 ```
 ```
 sudo /etc/init.d/xinetd restart
  - Ubuntu14.04.05 애소 tftp 서버 시작
```
```
gcc -o hello main.c
  - gcc로 컴파일
```
```
subl '우분투 14 apt-get 안될때.txt'
  - 인용부호(') 사용하여 공백문자가 포함된 파일 편집하기
```
```
alias
  - 단축 명령어 만들기
```
```
ctrl+shift+c / ctrl+shift_v
  - 복사 / 붙여넣기
ctrl+L
  - 화면 지우기
ctrl+shift+N
  - 새 터미널 띄우기
```
```
subl ~/.bashrc
  - 문서 편집기 (bashrc 편집)
source ~/.bashrc
  - bashrc 편집 전 열었던 터미널에 수정사항 적용
h
  - 명령어 history 확인 가능
ctrl 누르고 커서 방향키 이동하면 이동속도가 빠름(단어단위 이동)
```
```
which ct-ng
  - ct-ng가 설치된 경로
```
```
cp -ra linux-3.0.22 linux-3.0.22_v1
  - 백업본 만들기
```
