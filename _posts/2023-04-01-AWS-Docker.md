---
title : AWS에 Docker 설치하기
date : 2023-04-01 00:00:00 +09:00
categories : [CI/CD, AWS]
tags : [study, ci, cd, docker, aws] 
mermaid: true
---

## AWS 가입 및 EC2 생성

>Free-Tier 아이디 [[생성](https://aws.amazon.com/ko/free/?trk=fa2d6ba3-df80-4d24-a453-bf30ad163af9&sc_channel=ps&ef_id=Cj0KCQjwl8anBhCFARIsAKbbpyTEOoWJDgzVkqnzjHqZw3SFrMDKf5vrrGAi5ZUu284m-MCtDed3zp4aAqD5EALw_wcB:G:s&s_kwcid=AL!4422!3!563761819834!e!!g!!aws!15286221779!129400439466&all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all)]
{:.prompt-info}

무료버전인 AWS프리티어로 아이디를 만들어서 지역을 한국(서울)로 변경하고 EC2 인스턴스를 생성하였다.

### key.pem 
인스턴스에 접근하기위해 key.pem파일을 만들고 보안설정을 해주어야한다. 보안은 속성-보안탭-고급 에서 administrator이외의 다른 것이 존재하면 ssh명령으로 접근이 안된다.

```ssh -i "pem 파일 이름.pem" ubuntu@ec2-ip주소.리전.compute.amazonaws.com``` 명령을 통해 접근을 할 수 있다.

### swap 메모리 설정
>AWS 공식 가이드 [[링크](https://repost.aws/ko/knowledge-center/ec2-memory-swap-file)]
{:.prompt-info}

프리티어는 기본 메모리가 1GB밖에 안되기 때문에 swap 메모리 설정을 통해 3GB 까지 늘려서 쓰곤 한다.

	sudo dd if=/dev/zero of=/swapfile bs=128M count=32
	sudo chmod 600 /swapfile
	sudo mkswap /swapfile
	sudo swapon /swapfile
	sudo swapon -s
	sudo vi /etc/fstab

파일끝에 ```/swapfile swap swap defaults 0 0``` 를 추가한다.

<br>

## Jupyter Notebook (주피터 노트북)

jupyter notebook은 파이썬의 라이브러리이다. 웹 브라우저로 파일에 접근 할수 있게 되어 편리하다.

	//linux의 sudo는 권한이 없다고 뜰 때 사용
	sudo apt-get update // linux의 apt-get 최신버전으로
	sudo apt-get python3-pip // python3-pip 설치
	sudo pip3 install notebook // Jupyter Notebook 설치
	mkdir ssl //ssl 설정을 위한 폴더생성
	cd ssl
	
```sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout "공개키.key" -out "개인키.pem" -batch``` 를 입력하여 ssl 설정을 위한 키를 생성한다.



### 환경설정

다음으로는 Jupyter Notebook의 비밀번호를 설정해야하는데 다음과 같이 입력한다. ```from jupyter_server.auth import passwd; passwd()```

![실행화면1](/assets/img/aws-docker/001.png)

이때나온 해쉬값을 메모장에 잘 보관한다.

	jupyter notebook --generate-config
	// 설정파일의 경로가 나옴 복사 ctrl+c
	sudo vi 설정파일 경로

```ctrl+D``` 혹은 ```exit()```로 파이썬을 빠져나올수 있다.

해당 파일을 ```PageDown``` 키등으로 쭉 내려서 제일 밑에 가서 a키를 입력하면 수정모드가 되는데, 다음과 같이 입력하면 된다.

	c=get_config() //설정 객체 제일 상단에 존재하는 경우 입력 X
	c.ServerApp.password =u(우클릭)
	c.ServerApp.ip='cmd 상단의 ip'
	c.ServerApp.port = 8888
	c.ServerApp.notebook_dir = '/'
	c.ServerApp.certfile = u'/home/ubuntu/ssl/개인키.pem'
	c.ServerApp.keyfile = u'/home/ubuntu/ssl/공개키.key'

위와 같이 입력하면 된다. 설정 객체에서 설정법이 ```NotebookApp -> ServerApp``` 으로 바뀌었다.

![실행화면2](/assets/img/aws-docker/002.png)


### 서비스 등록 및 자동 실행

```sudo vi /etc/systemd/system/jupyter.service``` 이라고 입력하여 새로운 서비스 파일을 만들고 다음과 같이 입력한다.

	[Unit]
	Description=Jupyter Notebook Server

	[Service]
	Type=simple
	User=ubuntu
	ExecStart=/usr/bin/sudo /usr/local/bin/jupyter-notebook --allow-root --config=/home/ubuntu/.jupyter/jupyter_notebook_config.py

	[Install]
	WantedBy=multi-user.target

만든 서비스를 실행한다.

	sudo systemctl daemon-reload
	sudo systemctl enable jupyter
	sudo systemctl start jupyter
	sudo systemctl restart jupyter // 최초 실행 후 변경점이 있다면 이 명령어 실행

![실행화면3](/assets/img/aws-docker/003.png)

>http://www.gstatic.com/generate_204 로 이동된다면 크롬의 보안 설정을 내려야한다. [참고](https://tisiphone.tistory.com/282)
{:.prompt-tip}

<br>

## Docker EC2에 설치

>[Docker 공식] [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)
{.:prompt-info}

	sudo apt-get update
	sudo apt-get install ca-certificates curl gnupg software-properties-common
	sudo install -m 0755 -d /etc/apt/keyrings
	curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
	sudo chmod a+r /etc/apt/keyrings/docker.gpg

	echo \
	  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
	  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
	  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	  sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

다 설치를 했다면 ```sudo docker run hello-world``` 를 입력해서 hello-world 이미지를 pull 받아 보자.

<br>

## Reference
- [도커 활용및 배포 자동화 실전 초급] [https://youtu.be/LoYpXoBJPMc?si=pc2PYsvg1W_3PGLg](https://youtu.be/LoYpXoBJPMc?si=pc2PYsvg1W_3PGLg)
- [http://www.gstatic.com/generate_204 화면으로 리다이렉션 될 경우] [https://tisiphone.tistory.com/282](https://tisiphone.tistory.com/282)
- [Docker 공식] [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)
- [SWAP 메모리] [https://repost.aws/ko/knowledge-center/ec2-memory-swap-file](https://repost.aws/ko/knowledge-center/ec2-memory-swap-file)