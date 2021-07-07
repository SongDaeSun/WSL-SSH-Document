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
  
5. 외부에서 SSH 접속이 성공하면 다음 스텝으로 넘어간다.
