ahmed@ahmed-VirtualBox:~$ docker pull sonarqube:lts
lts: Pulling from library/sonarqube
Digest: sha256:f709975ab31d2d08f5a3ae2dc73a31ee011afc8cf28845082c17c55d45df9df5
Status: Image is up to date for sonarqube:lts
docker.io/library/sonarqube:lts
ahmed@ahmed-VirtualBox:~$ docker run -d -p 9000:9000 --name sonarqube sonarqube:lts
docker: Error response from daemon: Conflict. The container name "/sonarqube" is already in use by container "b3f3460328de2725fb53eebfadf5cf20b9d8ec6cad830ddc70ef07dfcec7f6b3". You have to remove (or rename) that container to be able to reuse that name.

Run 'docker run --help' for more information
ahmed@ahmed-VirtualBox:~$ 

