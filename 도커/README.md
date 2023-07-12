### 도커설치
	1. sudo apt-get update
    2. sudo apt-get install ca-certificates curl gnupg
    3. sudo install -m 0755 -d /etc/apt/keyrings
    4. curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    5. sudo chmod a+r /etc/apt/keyrings/docker.gpg
    6. echo \
        "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
        "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list > /dev/null     
    7. sudo apt-get update     
    8. sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    9. docker -v

### 도커명령(자주씀)

    1. docker cp (로컬 파일을 컨테이너 파일로 복사)
    2. docker inspect (자세한 내용확인)
    3. watch -n 1 docker ps -a (터미널에서 변경사항 수시확인)
    4. docker run --restart (재시작 정책설정)
    5. docker stats (서버자원 사용량 확인)
    6. docker run -it -d -v --name 이름 --restart=설정 컨테이너이름  

### 베이그란트로 손쉽게 초기 세팅 해보기

    # -*- mode: ruby -*-
    # vi: set ft=ruby :
    Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/focal64"

    config.vm.define "ubuntu" do |ubuntu|
        ubuntu.vm.hostname = "mz-server"
        ubuntu.vm.provider "virtualbox" do |vb|
        vb.name = "docker-server"
        vb.cpus = 4
        vb.memory = 8192
        end
        ubuntu.vm.network "forwarded_port", guest: 80, host: 80
        ubuntu.vm.network "private_network", ip: "192.168.33.10"
        ubuntu.vm.provision "shell", inline: <<-SCRIPT
        sudo apt-get update -y
        sudo apt-get install -y ca-certificates curl gnupg
        sudo install -m 0755 -d /etc/apt/keyrings
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
        sudo chmod a+r /etc/apt/keyrings/docker.gpg
        echo \
            "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
            "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
            sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        sudo apt-get update -y
        sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        sudo usermod -a -G docker vagrant
        docker run -it -d -p 80:80 --name nginx-server nginx
        docker cp /vagrant/env/sample2/ nginx-server:/usr/share/nginx/html

### 도커허브 로그인 토큰 등록
    1.env/docker_token 파일을 생성하여 토큰번호를 등록 합시다       

### 도커 스토리지 관리 해보자
    1.바인드마운트 호스트 파일시스템에서 직접 관리(내가 지정)
    2.볼륨마운트 도커 시스템에서 간접 관리(/var/lib/docker/volumes)
    3.tmpfs 메모리에만 저장, 호스트에 저장안함,성능이 좋고 빠름(시스템 메모리)
    4.볼륨마운트 커맨드
    5.볼륨 크리에이트 하고 컨테이너에 마운트하기
    6.docker run -d -p (8080:80) -v web_src_vol:/usr/local/apache2/htdocs --name 컨테이너이름 이미지(httpd)

### 도커 허브 이미지를 관리해보자
    1.docker tag ubuntu:20.04 minjun5876/myrepo:ubuntu20.04(받은 이미지 이름변경)
    2.docker run -d -p 5000:5000 --restart=always --name image_repo registry(프라이빗 도커헙 생성)
    3.sudo vi /etc/docker/daemon.json 수정하기
    # docker tag minjun5876/apache2:latest localhost:5000/apache2:latest
    4.sudo vi /etc/hosts 수정하기
    #127.0.0.1       localhost
     192.168.33.100  repo.docker.com
    # sudo systemctl restart docker
    6.sudo vi /etc/docker/daemon.json
    #{
        "insecure-registries" : ["http://localhost:5000", "http://repo.docker.com:5000"]     
    }
    # sudo systemctl restart docker
    7.docker tag localhost:5000/apache2:latest repo.docker.com:5000/apache2:latest
    # docker push repo.docker.com:5000/apache2(아이피도 가능)
    8.curl http://repo.docker.com:5000/v2/_catalog curl http://repo.docker.com:5000/v2/apache2/tags/list(프라이빗 레포확인)
    9.개인용 레포지터리를 만들면 키를 두개 만들어서 관리하고 외부에서 접속하는 서버는 공개키가 필요함.
    10./etc/hosts에 도메인 등록 (192.168.33.200  dock.repo.co.kr)
    11.sudo openssl req -newkey rsa:4096 -nodes -sha256 -keyout dock.repo.co.kr.key -subj "/C=KR/ST=Korea/L=Busan/O=mzcloud/OU=mzcloud/CN=dock.repo.co.kr/emailAddress=mj@mz.co.kr" -addext "subjectAltName = DNS:dock.repo.co.kr" -x509 -days 365 -out dock.repo.co.kr.crt (키생성)
    12. mkdir certs ,sudo mv mydomain.* certs/ 키 이동
    13. sudo mkdir -p /etc/docker/certs.d/repo.docker.com 디렉토리생성
    14.sudo cp certs/mydomain.crt /etc/docker/certs.d/repo.docker.com/mydomain.crt
    15.sudo cp certs/mydomain.crt /usr/local/share/ca-certificates/repo.docker.com.crt
    16.docker run -d --restart=always --name image_repo -v /home/vagrant/certs:/certs -e REGISTRY_HTTP_ADDR=0.0.0.0:443 -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/mydomain.crt -e REGISTRY_HTTP_TLS_KEY=/certs/mydomain.key -p 443:443 registry
    17.scp -i /vagrant/.vagrant/machines/dock1/virtualbox/private_key cets/dock.repo.co.kr.crt vagrant@192.168.33.100:/home/vagrant
    18.cp /vagrant/.vagrant/machines/dock1/virtualbox/private_key ~/dock1.key
    19.sudo chmod 600 dock1.key
    20.scp dock1.key certs/dock.repo.co.kr.crt vagrant@192.168.33.100:/home/vagrant
    21.scp -i dock1.key certs/dock.repo.co.kr.crt  vagrant@192.168.33.100:/home/vagrant dock.repo.co.kr.crt
    22.docker run -d --restart=always --name image_repo -v /home/vagrant/certs:/certs -e REGISTRY_HTTP_ADDR=0.0.0.0:443 -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/dock.repo.co.kr.crt -e REGISTRY_HTTP_TLS_KEY=/certs/dock.repo.co.kr -p 443:443 registry