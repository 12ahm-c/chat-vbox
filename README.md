ahmed@ahmed-VirtualBox:~$ docker stop sonarqube
sonarqube
ahmed@ahmed-VirtualBox:~$ docker rm  sonarqube
sonarqube
ahmed@ahmed-VirtualBox:~$ docker run -d -p 9000:9000 --name sonarqube sonarqube:lts
e72b594230e047ab437c9d40a03c2cbf1a20f810c46bc474574dc41ebe384883
ahmed@ahmed-VirtualBox:~$ 
