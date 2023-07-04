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

![스크린샷(4)](https://github.com/minjunkim5876/Project_Owl/assets/110665712/a2b68f07-b6d4-4ad0-b1bd-39c33c4ead32)
