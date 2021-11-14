### Implement Realtek RTL8196D/E SDK-based web-redirection function

### SDK 기본 구조
|디렉토리 구성|디렉토리 설명|
|:---:|:---|
|boards|칩셋 별 설정 파일 및 스크립트 파일|
|bootcode_rtl8196d|칩셋 별 부트로더 소스|
|config|메뉴 설정 구성|
|linux-2.6.39|리눅스 커널 소스|
|toolchain|칩셋 별 크로스 컴파일러|
|users|어플리케이션 소스|

### 컴파일
- make menuconfig: `model/kernel/uses/config busybox`
- make: `image/fw.bin`

### 시리얼 설정
- port: /dev.cu.usbserial-AL03FT2B
- baud rate: 38400
- data bits: 8
- parity: None
- stop bits: 1

### 업로드 절차
1. 타겟보드 전원 인가와 동시에 터미널 프로그램에서 'esc'키 입력(부트보드 명령어 모드 진입)
2. 타겟보드의 기본 IP주소는 192.168.1.6
  - ipconfig 명령어로 ip주소 확인 및 변경 가능
3. 타겟보드와 pc를 lan케이블로 연결
  - 업로드할 펌웨어 파일 복사(fw.bin)
  - pc의 ip주소를 192.168.1.x/24로 설정
  - pc의 tftp 클라이언트 프로그램을 사용하여 타겟보드로 펌웨어 파일 전송

### 공장 초기값 설정
- 공장 초기값 설정 파일:`users/goahead-2.1.1/LINUX/flash.c`
- 시리얼 터미널에서 설정값 관련 명령어  
`flash get MIB_XXX`  
`flash set MIB_XXX XXX`  
`flash all`  
`flash allhw`
- 공장 초기화 방법
  - WEB UI 관리자 설정->시스템 관리->설정 공장 초기화
  - reset전원(3초이상)
  - 시리얼 터미널(디버그 용도): `flash default-sw`

### web redir

### 기능설명
- 클라이언트가 접속한 URL주소를 새로운 다른 주소로 자동으로 연결해주는 기능

### 구현방법
```mermaid
graph LR
A[Client] -- web --> B[wireless router]
B[wireless router] -- web --> C[web server]
```
- 리눅스 커널의 넷필터 프레임워크 사용
- 클라이언트의 웹페이지 요청(port:80)은 Redirection URL만 허용, 나머지 차단
- 클라이언트의 웹페이지 요청(port:80) 패킷을 후킹
- 해당 패킷을 고유기 자체 웹서버(port:81)로 전달
- Redirection URL 클라이언트로 응답
- 클라이언트 웹 브라우저는 Redirection URL로 이동

### 개발환경
- Linux Ubuntu-14.04.1
- RTL8196D/E SDK
```
npm install build-essential
npm install gwak
npm install bison
npm install zlib1g-dev
```

### 기능구현(상세)
- busybox httpd 간이 웹 서버 사용체크
`make menucofig -> Config busybox -> Networking Utilites -> httpd`
- 사용자 설정 웹 페이지 구성: `users/goahead-2.1.1/web-wevo.html`
  - header.asp 수정, web_redir.asp 추가
- 설정 값 Read/Write ASP 구현: `users/goahead-2.1.1/LUNUX/`
  - apmib.h, mibdef.h, wevo.c 수정
  - mibdef.h
```
#if 1   // WEB_Redirect - KwanDong Univ.
MIBDEF(unsigned char,   web_redir_en,   ,   WEB_REDIR_EN,   BYTE_T, APMIB_T, 0, 0)
MIBDEF(unsigned char,   web_redir_url, [40], WEB_REDIR_URL,  	STRING_T,   APMIB_T, 0, 0)
#endif

```
- 설정 값에 따른 Web Redirection기능 시스템에 반영: `users/goahead-2.1.1/LINUX/system/`
  - set_firewall.c 기능구현
```
#if 1   // URL Redirect - KwanDong Univ.
   char WEB_REDIR_CHAIN[]="web_redir";
#endif

#if 1   // URL Redirect - KwanDong Univ.

int set_web_redir()
{
 int val, webredirEn;
 unsigned int	lan_addr;
 char	webredirURL[32] = {0, };
 char	strLanIp[16];
 char	dest[40];
 FILE	*fp;

 system("killall -q httpd");

 RunSystemCmd(NULL_FILE, Iptables, _table, nat_table, NEW, WEB_REDIR_CHAIN, NULL_STR);
 RunSystemCmd(NULL_FILE, Iptables, _table, nat_table, INSERT, PREROUTING, jump, WEB_REDIR_CHAIN, NULL_STR);
 RunSystemCmd(NULL_FILE, Iptables, _table, nat_table, FLUSH, WEB_REDIR_CHAIN, NULL_STR);

 apmib_get(MIB_WEB_REDIR_EN, (void *)&webredirEn);

 if (webredirEn == 1)
 {
     apmib_get(MIB_WEB_REDIR_URL, (void *)&webredirURL);
       system("httpd -p 81 -h /var/web");

     fp = fopen("/var/web/index.html", "w");
     if (fp)
     {
         fprintf(fp, "<frameset>\n");
         fprintf(fp, "<frame name='CONTENT' src='http://%s' frameborder='0'>\n", webredirURL);
         fprintf(fp, "</frameset>\n");
         fclose(fp);
     }

     fp = fopen("/var/web_redir.sh", "w");
     if (fp)
     {
         apmib_get(MIB_IP_ADDR, &lan_addr);
         strcpy(strLanIp, inet_ntoa(*(struct in_addr *)&lan_addr));
         fprintf(fp, "#!/bin/sh\n");
         fprintf(fp, "iptables -A web_redir -t nat -p tcp -d %s -j ACCEPT\n", webredirURL);
         fprintf(fp, "iptables -A web_redir -t nat -p tcp -d %s --dport 80 -j DNAT --to %s:80\n", strLanIp, strLanIp);
         fprintf(fp, "iptables -A web_redir -t nat -p tcp --dport 80 -j DNAT --to %s:81\n", strLanIp);
         fclose(fp);
         chmod("/var/web_redir.sh", 0700);
         system("/var/web_redir.sh");
     }
 }
 return 0;
}
#end if

#if 1
 	apmib_get(MIB_WEB_REDIR_EN, (void *)&intVal);
 	if(intVal ==1){
     	set_web_redir();
 	}
#endif
```
