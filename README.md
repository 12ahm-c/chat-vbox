docker run -d -p 9000:9000 --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -m 2g sonarqube:lts
