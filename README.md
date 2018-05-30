# docker-sonarqube

## Basic usage

Read following after replace 'any_\*' and 'your_\*' to appropriate value.

### Create .env

```
$ cat .env.template
POSTGRES_DATABASE=any_string_database_name
POSTGRES_USER=any_string_user_name
POSTGRES_PASSWORD=any_string_user_password
$ cp .env.template .env
$ vi .env   # Edit variables to appropriate value.
```

### Run docker-sonarqube

```
$ docker-compose up -d
```

### Access to sonarqube-server

1. Access to http://localhost:9000

2. Login to SonarQube with ```admin```/```admin```. (* To change password from [Administration] > [Security] > [Users] is recommended.)

3. Generate a token in tutorial.

4. Copy the displayed token string. (* You won't be able to see it again.)

5. Press [Skip this tutorial].

### Create sonar-project.properties

Create sonar-project.properties in the root directory of project you want to analyze.

#### Example sonar-project.properties:

```
sonar.projectKey=any_string_project_name
sonar.sources=./any_string_directory/any_string_subdirectory/
sonar.host.url=http://localhost:9000
sonar.login=your_token_string
```

For details, refer to https://docs.sonarqube.org/display/SONAR/Analysis+Parameters

### Install sonar-scanner

Install sonar-scanner into host OS to analyze the project. (* sonar-scanner need to install JavaVM on the system.)

For MacOS, use homebrew.

```
$ brew install sonar-scanner
```

For other OS, download the package from the following URL.

https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner

### Analyze project

Run the following command from the project root directory to launch the analysis.

$ sonar-scanner

### Check result of analyzing

Access to http://localhost:9000 again.
