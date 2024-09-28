## Mac Terminal 에서 개발하기

[1] UTM 에서 우분투 실행

[2] ip addr로 아이피 확인하기

<img width="800" alt="Screenshot 2024-09-28 at 7 18 27 PM" src="https://github.com/user-attachments/assets/74d79639-3528-47fc-8c8f-3c41e12ee01c">

[3] Terminal 에서 `ssh username@ip addr`

---

## UTM 에서 Port Forwarding 하기

<img width="350" alt="Screenshot 2024-09-28 at 10 23 47 PM" src="https://github.com/user-attachments/assets/4582e09c-1a62-454d-930a-b4678d359885">


> Edit 클릭

<img width="815" alt="Screenshot 2024-09-28 at 10 24 22 PM" src="https://github.com/user-attachments/assets/81d796b6-cb85-4c63-b2cc-a190751d145b">

> New 버튼 눌러서 Network가 3개 만들어지도록 한다 <br>
> 원래는 NAT 로 진행해야 하는데 없으므로 슬롯 네트워크 (Bridged Network) 모 드로 설정하여 가상머신과 호스트가 동일한 네트워크 내에서 통신할 수 있도록 한다.

[1] Shared Network 1개

<img width="805" alt="Screenshot 2024-09-28 at 10 27 03 PM" src="https://github.com/user-attachments/assets/cf8cf320-15e4-46fe-8213-265aa7913c23">

[2] Emulated VLAN - Port Fowarding 1개

<img width="805" alt="Screenshot 2024-09-28 at 10 27 22 PM (1)" src="https://github.com/user-attachments/assets/b28bca86-841c-43b9-a001-1955cbadb513">


> 필요한 포트는 여기서 열어준다.

<img width="806" alt="Screenshot 2024-09-28 at 10 26 31 PM" src="https://github.com/user-attachments/assets/5fb651ff-648d-4aab-b8fa-676bfe079a33">


[3] Bridged (Advanced) 1개

<img width="804" alt="Screenshot 2024-09-28 at 10 27 44 PM" src="https://github.com/user-attachments/assets/fbba9704-de10-4abd-a681-86bf2a40dae7">


---

## Jenkins 설치

`docker run --name myjenkins --privileged -p 8080:8080 jenkins/jenkins:lts-jdk17`

<aside>

🔫 docker: permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Head "http://%2Fvar%2Frun%2Fdocker.sock/_ping": dial unix /var/run/docker.sock: connect: permission denied.
See 'docker run --help'.

현재 사용자가 docker 그룹에 속해 있지 않아서 Docker 데몬에 접근할 권한이 없다는 것을 의미

**문제 해결 방법 :** *현재 사용자를 docker 그룹에 추가하기*

**sudo usermod -aG docker $USER** >> docker 그룹에 추가

**newgrp docker** >> 변경 사항 적용

**docker run hello-world** >> docker 권한 확인

</aside>

```jsx
pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/ayleeee/jenkinstest.git'
            }
        }
          
        stage('Build') {   
            steps {
                dir('./step18_empApp') {                   
                    sh 'chmod +x gradlew'                    
                    sh './gradlew clean build -x test'   //*.jar
                    sh 'echo $WORKSPACE'
                }
            }
        }
        
        stage('Copy JAR') {  // 큰따옴표 오류 수정  
            steps {
                script {
                    def jarFile = 'step18_empApp/build/libs/step18_empApp-0.0.1-SNAPSHOT.jar'                   
                    sh "cp ${jarFile} /var/jenkins_home/appjar/"
                }
            }
        }
    }
}
```

<aside>

🔫  cp step18_empApp/build/libs/step18_empApp-0.0.1-SNAPSHOT.jar /var/jenkins_home/appjar/
cp: cannot create regular file '/var/jenkins_home/appjar/step18_empApp-0.0.1-SNAPSHOT.jar': Permission denied

Jenkins환경에서 생긴 권한 문제

**문제 해결 방법 : 권한 수정하기**

docker exec -u root -it myjenkins /bin/bash <br>
chown -R jenkins:jenkins /var/jenkins_home/appjar

</aside>
