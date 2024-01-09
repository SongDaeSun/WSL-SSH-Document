# WSL-SSH-Document-Intro
일반 윈도우 컴퓨터에 WSL을 설치하여 게임할 때는 윈도우,  
개발할 때는 리눅스 서버처럼 사용하고자 하는 야망이 있다.  
다만 이 과정이 생각보다 복잡하기에 셋팅법을 적어두고 공유하려고 한다.

# Install WSL
아래 공식 사이트에서 명하는 대로 하라  
https://docs.microsoft.com/ko-kr/windows/wsl/install-win10  
개인적으로는 windows terminal까지 설치하는 것을 추천한다.

# Install SSH Server
APT Repository 최신 업데이트
```
sudo apt-get update
sudo apt-get upgrade
```
  
SSH 서버 완전 삭제 후 재설치
```
sudo apt-get purge openssh-server
sudo apt-get install openssh-server
```
  
config 파일 설정
```
sudo nano /etc/ssh/sshd_config
```

다음의 내용을 '/etc/ssh/sshd_config'에 추가한다.
```
Port 22
Protocol 2
```

```
PasswordAuthentication no -> yes
```

SSH 서버 재시작
```
sudo service ssh --full-restart
sudo service ssh restart
```

로컬접속 확인
```
ssh localhost
```
접속에 성공했다면 다음 스텝으로 넘어갈 준비가 되었다.

# Local Port Forward
WSL의 경우 새롭게 부여된 가상의 IP(ex. 172.18.48.1)를 사용하는데,  
이는 컴퓨터(Windows)가 원래 사용하던 IP(ex. 192.168.0.4)와는 다른 것이다.  
  
따라서 외부 시스템에서 우리의 WSL SSH 서버에 접속하려면  
Windows - 192.168.0.4 -> WSL - 172.18.48.1로 연결되는 local 포트포워딩이 필요하다.  
  
그 포트포워딩을 하는 코드는 어떤 능력자 행님께서 이미 만들어 주셨다.  

출처 : https://dev.to/vishnumohanrk/wsl-port-forwarding-2e22  
```
$remoteport = bash.exe -c "ifconfig eth0 | grep 'inet '"
$found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

if( $found ){
  $remoteport = $matches[0];
} else{
  echo "The Script Exited, the ip address of WSL 2 cannot be found";
  exit;
}

#[Ports]

#All the ports you want to forward separated by coma
$ports=@(22, 80,443,10000,3000,5000);


#[Static ip]
#You can change the addr to your ip config to listen to a specific address
$addr='0.0.0.0';
$ports_a = $ports -join ",";


#Remove Firewall Exception Rules
iex "Remove-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' ";

#adding Exception Rules for inbound and outbound Rules
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort $ports_a -Action Allow -Protocol TCP";
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort $ports_a -Action Allow -Protocol TCP";

for( $i = 0; $i -lt $ports.length; $i++ ){
  $port = $ports[$i];
  iex "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$addr";
  iex "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$remoteport";
}
```
1. 위의 스크립트를 .ps1이라는 확장자로 저장해두자.  
ex) C:\local_portforward.ps1  

2. 아래 코드에서 원하는 포트를 추가하거나 삭제하면 된다.  
```$ports = @(3000, 3001, 5000, 5500, 19000, 19002, 19006); ```   
SSH의 기본 포트는 22이므로 필자는 22를 추가해줬다.  
  
3. 관리자 권한으로 power shell을 켜서 위에서 만든 스크립트를 실행한다.  
  
4. 중간에 보안 이슈 때문에 실행이 안된다면  
```Set-ExecutionPolicy RemoteSigned```이라는 명령어를  
관리자 권한으로 power shell에 실행시켜보자

```
PowerShell.exe -ExecutionPolicy Bypass -File .\local_portforward.ps1
```

5. 
만일 다음과 같은 문구만이 나오고 제대로 실행이 되지 않는다면,
```
/bin/bash: line 1: ipconfig: command not found
The Script Exited, the ip address of WSL 2 cannot be found
```

ubuntu에 ifconfig관련 패키지가 설치되지 않은 것이므로,
```
sudo apt-get install net-tools
```
으로 새로 설치해주자.

6. 외부에서 SSH 접속이 성공하면 다음 스텝으로 넘어간다.  
  
# 재부팅시 자동으로 SSH 서버 실행
컴퓨터를 재부팅하면 SSH 서버가 꺼져서 다시 안켜진다.  
일일이 다시 켜는 것이 귀찮고, 외부에서 WoL으로 윈도우 컴을 키는 경우 역시 잦음으로  
다음의 과정을 통해 재부팅시 자동으로 SSH 서버를 켜보자  
  
1. 시작(윈도우 아이콘) -> 실행 -> "shell:startup"으로 "시작프로그램" 폴더에 접근한다.  
  
2. sshd.bat이라는 파일명에 아래의 코드를 작성한다.  
```
@echo off
"C:\Windows\System32\bash.exe" -c "sudo service ssh start"
``` 
  
3. Ubuntu(WSL)에서 ```sudo visudo``` 명령어로 /etc/sudoers.tmp접근  
  
4. /etc/sudoers.tmp 맨 아래에 아래 코드를 붙혀넣기
```
%sudo ALL=NOPASSWD: /usr/sbin/service
```

만일 sshd.bat 파일이 실행자체가 안된다면, 
윈도우 11기준 "제어판"-"프로그램"-"프로그램 및 기능"-좌측 상단의 "Windows 기능 켜기/끄기"-"Linux용 Windows 하위 시스템"체크 후  
프로그램 지시 따라 재부팅을 해보자.


5. 재부팅 후 ```ssh localhost```로 접속이 잘되는지 

# 재부팅시 자동으로 local port forward 
재부팅을 하면 SSH 서버만 재실행 해야하는 것이 아니라,  
Local 포트포워딩도 재실행 해줘야 한다.  
~~어지간히 귀찮은 일이다.  
그냥 리눅스 서버를 새로 사는게 나을지도 모른다.  
하지만 그럴 돈이 없다.~~  
  
windows의 "작업 스케줄러"를 사용하여 이 문제를 해결한다.  
1. 작업 스케줄러 실행  
  
2. 오른쪽의 "작업 만들기"  
  
3. "일반" 탭에서 "이름"은 자유롭게,  
"가장 높은 수준의 권한으로 실행" 체크  
"구성 대상"은 "Windows 10"으로 설정  
  
4. "트리거" 탭에서 "새로 만들기"  
"작업시작" : "로그온할 때" 선택
  
5. "동작" 탭에서 "새로 만들기"  
"프로그램/스크립트" : ```C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe```  
"인수 추가(옵션)" : ```-ExecutionPolicy Unrestricted -File "C:\local_portforward.ps1"```  
```-File 뒤에는 경로+파일명을 적자```
  
6. 재부팅 후 외부접속이 잘 되는지 확인한다.

# WSL + Docker Configuration
window용 docker를 설치할 때 wsl를 체크해주면 자동으로 설치 된다.  
참 세상 좋아졌다.


# bare ubuntu ssh start on boot
```
sudo systemctl enable ssh
```
