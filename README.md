ahmed@ahmed-VirtualBox:~$ docker pull jenkins/jenkins:lts
lts: Pulling from jenkins/jenkins
53c88f1dfeb7: Pull complete 
17e22d939f17: Pull complete 
a1c44ba8dd7a: Pull complete 
5b302cbb8ad3: Pull complete 
213d96e0ed06: Pull complete 
e92c5cf141dc: Pull complete 
406f94702a48: Pull complete 
39483126681f: Pull complete 
99ddb13f3ebf: Pull complete 
ed559592686a: Pull complete 
57364cf49183: Pull complete 
7ad60f966cc5: Pull complete 
Digest: sha256:d1ea795c6facd7f549a21c40e5e43ffcc5fbc5f48683d9b24750f26e8079d772
Status: Downloaded newer image for jenkins/jenkins:lts
docker.io/jenkins/jenkins:lts
ahmed@ahmed-VirtualBox:~$ docker run -d -p 8080:8080 -v jenkins_home:/var/jenkins_home
jenkins/jenkins:lts
--argumentsRealm.passwd.Ahmed122005=Aa2233
--argumentsRealm.roles.admin=Ahmed122005
docker: 'docker run' requires at least 1 argument

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

See 'docker run --help' for more information
bash: jenkins/jenkins:lts: No such file or directory
--argumentsRealm.passwd.Ahmed122005=Aa2233: command not found
--argumentsRealm.roles.admin=Ahmed122005: command not found
ahmed@ahmed-VirtualBox:~$ 

