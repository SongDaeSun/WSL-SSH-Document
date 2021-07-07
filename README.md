# WSL-SSH-Document-Intro
일반 윈도우 컴퓨터에 WSL을 설치하여 마치 리눅스 서버처럼 사용하고자 하는 야망이 있다.  
다만 이 과정이 생각보다 복잡하기에 셋팅법을 적어두고 공유하려고 한다.

# Install WSL
아래 공식 사이트에서 명하는 대로 하라  
https://docs.microsoft.com/ko-kr/windows/wsl/install-win10

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
접속에 성공했다면 다음 스텝으로 넘어갈 준비가 

# Local Port Forward
