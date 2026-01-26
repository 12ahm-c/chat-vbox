ahmed@ahmed-VirtualBox:~$ docker run -d -p 8081:8080 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
181fa306eb50767a4643a7b9cf3f99cb9725df679b13bbb343453f54cd73de08
ahmed@ahmed-VirtualBox:~$ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                                    NAMES
181fa306eb50   jenkins/jenkins:lts   "/usr/bin/tini -- /u…"   14 seconds ago   Up 14 seconds   50000/tcp, 0.0.0.0:8081->8080/tcp, [::]:8081->8080/tcp   magical_hoover
ahmed@ahmed-VirtualBox:~$ 

