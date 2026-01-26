ahmed@ahmed-VirtualBox:~$ ^Ccker exec <container_id> cat /var/jenkins_home/secrets/initialAdminPassword
ahmed@ahmed-VirtualBox:~$ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                                    NAMES
e72b594230e0   sonarqube:lts         "/opt/sonarqube/dock…"   24 minutes ago   Up 23 minutes   0.0.0.0:9000->9000/tcp, [::]:9000->9000/tcp              sonarqube
181fa306eb50   jenkins/jenkins:lts   "/usr/bin/tini -- /u…"   46 minutes ago   Up 46 minutes   50000/tcp, 0.0.0.0:8081->8080/tcp, [::]:8081->8080/tcp   magical_hoover
ahmed@ahmed-VirtualBox:~$ docker exec 181fa306eb50  cat /var/jenkins_home/secrets/initialAdminPassword
cat: /var/jenkins_home/secrets/initialAdminPassword: No such file or directory
ahmed@ahmed-VirtualBox:~$ docker exec -it 181fa306eb50 bash 
jenkins@181fa306eb50:/$  cat /var/jenkins_home/secrets/initialAdminPassword
cat: /var/jenkins_home/secrets/initialAdminPassword: No such file or directory
jenkins@181fa306eb50:/$ cat /var/jenkins_home/secrets/initialAdminPassword
cat: /var/jenkins_home/secrets/initialAdminPassword: No such file or directory
jenkins@181fa306eb50:/$  cat /var/jenkins_home/secrets/initialAdminPassword
cat: /var/jenkins_home/secrets/initialAdminPassword: No such file or directory
jenkins@181fa306eb50:/$ 
