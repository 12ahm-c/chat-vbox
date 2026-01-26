ahmed@ahmed-VirtualBox:~$ docker run -d -p 8080:8080 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
d97b536a1f5d8dbefdc5593f8a882d568258ea8ee5144b932e1fa572a0085e69
docker: Error response from daemon: failed to set up container networking: driver failed programming external connectivity on endpoint condescending_faraday (b509d280750c571cf8fb2bd1a9495f283a0c75730cb6320369c3187932ce7fd4): failed to bind host port for 0.0.0.0:8080:172.17.0.2:8080/tcp: address already in use

Run 'docker run --help' for more information
