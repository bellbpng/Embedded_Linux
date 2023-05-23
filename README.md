# Embedded_Linux

## 임베디드 리눅스 커널 포팅 및 구조
### 개발환경 구성
- Vmware Workstation 17 Player
- Ubuntu14.04.05 vmwarevm 
  - VMware virtual machine config(.vmx) -> VMware에서 오픈
  - VMware virtual disk file
- 리눅스에서 인식할 수 있도록 XML 수정 -> device vid, pid 설정
- tftp directory만 target board에서 host에 접근할 수 있음
- 리눅스에서 파일명에 . 이 붙으면 숨김 파일임

### TFTP
- 임베디드 보드(Target)가 호스트로부터 이미지를 다운로드 할 때 사용하는 프로토콜

### NFS (Network File System)
- 임베디드 리눅스 환경에서 스토리지(Disk)를 관리하는 프로토콜
- 네트워크를 통해 파일 시스템을 mount할 수 있게 해준다.
- 호스트의 Remote Storage에 저장해두고 사용하는 방법
- 외부에 있는 Disk 장치를 본인 것처럼 사용 가능
- 개발 단계에서 용이하지만, 네트워킹이 필요없는 제품의 경우 최종적으로는 타겟 보드의 NAND Flash에 모두 저장한다.

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
