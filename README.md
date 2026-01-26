docker run -d -p 8081:8080 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts


docker stop d97b536a1f5f
docker rm d97b536a1f5f

docker volume rm jenkins_home

docker run -d -p 8081:8080 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
