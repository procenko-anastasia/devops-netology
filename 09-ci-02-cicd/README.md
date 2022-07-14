# Домашнее задание к занятию "09.02 CI\CD"

## Проценко Анастасия

## Знакомство с SonarQube

```
	$ docker pull sonarqube:8.7-community
	Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.
	✔ docker.io/library/sonarqube:8.7-community
	Trying to pull docker.io/library/sonarqube:8.7-community...
	[..]

	$ docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:8.7-community
	059c526604bbe11ca1ea030e26075d215205629b914cdaa05cc5b5a8ae30eff2
```

 [УСТАНОВОЧНЫЙ zip-ФАЙЛ](https://binaries.sonarsource.com/?prefix=Distribution/sonar-scanner-cli/)

```
	Anastasia@MacBook-Air ~ % sonar-scanner --version
	INFO: Scanner configuration file: /home/v/bin/sonar-scanner/conf/sonar-scanner.properties
	INFO: Project root configuration file: NONE
	INFO: SonarScanner 4.7.0.2747
	INFO: Java 11.0.14.1 Red Hat, Inc. (64-bit)
	INFO: Linux 5.16.20-200.fc35.x86_64 amd64
```
Запуск сканнера для файла file.py: 

```
	Anastasia@MacBook-Air ~ % sonar-scanner   -Dsonar.projectKey=test   -Dsonar.sources=.   -Dsonar.host.url=http://127.0.0.1:9000   -Dsonar.login=a41c3893669707653c3c554cd906d9e4b440cca1 -Dsonar.coverage.exclusions=file.py
```
Результат:

![sq](img/1.png)

## Знакомство с Nexus
```
	Anastasia@MacBook-Air ~ % docker pull sonatype/nexus3
	Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.
	✔ docker.io/sonatype/nexus3:latest
	Trying to pull docker.io/sonatype/nexus3:latest...
	[..]

	Anastasia@MacBook-Air ~ % docker run -d -p 8081:8081 --name nexus sonatype/nexus3
	Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.
	006bc2cc760cdd89d0d3e8102d9258ad9a05b6f61dc30868a223fe15e5fdd5c8
```
![nexus](img/2.png)
```
	docker exec -it nexus /bin/bashsh
	Emulate Docker CLI using podman. Create /etc/containers/nodocker to quiet msg.
	bash-4.4$ cat nexus-data/admin.password 
	2a6b3053-0252-4081-b85b-68378b5b8441
```
![nexus](img/3.png)
```
	$ touch 09-ci-02-cicd/dummy.tar.gz
```
![nexus](img/4.png)

![nexus](img/5.png)

## Знакомство с  Maven

	Anastasia@MacBook-Air ~ % mvn --version
	Apache Maven 3.8.5 (3599d3414f046de2324203b78ddcf9b5e4388aa0)
	Maven home: /home/v/bin/apache-maven
	Java version: 11.0.14.1, vendor: Red Hat, Inc., runtime: /usr/lib/jvm/java-11-openjdk-11.0.14.1.1-5.fc35.x86_64
	Default locale: en_US, platform encoding: UTF-8
	OS name: "linux", version: "5.16.20-200.fc35.x86_64", arch: "amd64", family: "unix"

	Anastasia@MacBook-Air ~ % mvn package
	[INFO] Scanning for projects...
	[INFO] 
	[INFO] --------------------< com.netology.app:simple-app >---------------------
	[INFO] Building simple-app 1.0-SNAPSHOT
	[INFO] --------------------------------[ jar ]---------------------------------
	[..]

