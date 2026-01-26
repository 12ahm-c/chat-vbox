docker run -d -p 8080:8080 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts

docker exec <container_id> cat /var/jenkins_home/secrets/initialAdminPassword


