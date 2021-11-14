### Ralink RT 3052 SDK 개발환경 구축 및 펌웨어 생성 방법

### 기본용어 정리
- Embedded System: 기계나 기타 제어가 필요한 시스템에 대해, 제어를 위한 특정 기능을 수행하는 컴퓨터 시스템으로 장치 내에 존재하는 전자 시스템
- SoC(System on Chip): 하나의 집적회로에 집적된 컴퓨터나 전자 시스템 부품
- Host PC: 프로그램을 개발하는 PC
- Target: 프로그램 결과물을 실행하는 시스템 환경
- Cross Compiler: 컴파일러가 실행되는 플랫폼이 아닌 다른 플랫폼에서 실행 가능한 코드를 생성할 수 있는 컴파일러

### 교차개발환경
- target system에서 응용프로그램을 개발할 수 없다.
  - 하드웨어 구성이 소프트웨어 개발환경으로 부적합
  - 저성능 CPU 및 메모리 용량 부족으로 컴파일 실행에 무리(소프트웨어 개발 생산성 저하)
- target system 개발환경
  - target system보다 성능이 좋은 host system을 이용하여 target system에서 동작하는 프로그램을 생성
  - 프로그램 작성 및 컴파일은 host system에서, 프로그램 실행은 target system에서 수행
  - target system과 host system이 서로 다른 형태의 프로세스를 사용하는 경우 Cross compiler가 필요
  - 주로 host system이 target system의 console로 동작
  - 이러한 개발환경을 교차 개발환경이라고 함

### Host 시스템
- 임베디드 소프트웨어를 개발하는 시스템
- 일반 PC에 Linux 운영체제를 설치하여 사용(가상머신, uBuntu 14.04 32bit)
- cross compile 환경 필요(toolchain 설치)
- target system에 연결하는 소프트웨어 설치(minicom, JTAG프로그램 등)

### Target 시스템
- 개발된 임베디드 소프트웨어가 실제 수행되는 시스템
- target CPU가 탑재된 시스템에 Linux 운영체제 Porting
- bootloader 및 특별한 flash 메모리용 file system 설치

### host system과 target system의 연결
- Serial 선~Console 연결, zmodem/tftp를 이용한 파일 전송
- JTAG 선~Hardward debugging, Flash memory programming
- USB 선~빠른 속도의 파일 전송 지원
- Ethernet 선~빠른 속도의 파일 전송, NFS 지원

### 시스템 사양
- Ralink RT3052 SoC
  - CPU: MIPS24KEc Single Core 384MHz
  - WiFi: 802.11n 2T2R(2.4GHz)
  - NIC: 5-port integrated 10/100 Ethernet switch/PHY
  - H/W Interface: USB OTG/SPI/I2S/I2C/UART/GMAC/GPIO
- 메모리
  - DDR: 64MB
  - NOR Flash: 4MB
- SDK
  - buildroot-gcc342(SDK Compile용 Toolchain)
  - wip300_sdk_asterisk_porting(IP-PBX)

### Toolchain
- target system에서 수행되는 소프트웨어를 개발하기 위해 필요한 host system의 cross compile 환경
- 소스코드를 compile & link 하여 binary 실행 파일을 생성하는데 필요한 각종 utility 및 library의 모음
- 기본적으로 assembler, C compiler, linker, C library 등으로 구성
- GUN에서 제공하는 Toolchain을 사용
  - GNU GCC for C, C++: gcc
  - GNU binary utilities: 여러 종류의 오브젝트 파일들을 조직하기 위한 프로그램 도구(assembler(as), linker(ld), ar, objcopy, objdump, ranlib, strip 등)
  - GNU C library: glibe

### SDK 구성
|디렉토리 구성|디렉토리 설명|
|:---:|:---|
|doc|사용자 매뉴얼 및 문서|
|toolchain|MIPS 툴 체인|
|source|리눅스 커널 소스(Linux 2.6)|
|tools|스크립트|
|config|auto-configuration 파일|
|lib|uClibc 0.9.28|
|linux-2.6.21.x|리눅스 커널 소스|
|roots|root file system|
|tools|roofs 생성하기 위한 스크립트|
|user|사용자 어플리케이션|
|vendor|타겟 플랫폼을 위햔 init 스크립트(inittab,rcS등)|

### SDK 컴파일
- ubuntu 14.04 (32bit) 버전 설치(development 관련 패키지 포함)
- toolchain 설치 `/opt/build-gcc342`
- tftpboot 디렉토리 생성(컴파일 시 사용)
- wip300 `/root`

`/kct_qmw/source` 컴파일
- [x] Kernel/Library/Defaults Selection
- [x] Customize Vendor/User Settings
<Exit>  

- [x] Miscellaneous Applications
- [x] mtd write
<Exit>  

- [x] Ralink Proprietary Application
- [x] Flash
<Exit>  

### 펌웨어 업로드
- 펌웨어 위치: `images/fw.bin`
- 웹서버 접속(http://192.168.100.254:8888)
- 로그인 UserID/pwd: bluecop/bluecop
- 관리자 설정 -> 업그레이드 펌웨어 -> 파일 선택
- 펌웨어 파일 선택 -> 적용



