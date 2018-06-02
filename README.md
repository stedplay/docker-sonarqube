# docker-sonarqube

## Basic usage

Read following after replace 'any_\*' and 'your_\*' to appropriate value.

### Create .env

```
$ cat .env.template
TIME_ZONE=your_continent/your_region   # e.g. Asia/Tokyo
POSTGRES_DATABASE=any_string_database_name
POSTGRES_USER=any_string_user_name
POSTGRES_PASSWORD=any_string_user_password
GDRIVE_ACCOUNT=your_mail_address@gmail.com
GDRIVE_SYNC_DST_DIR=/any_string_directory/any_string_subdirectory/
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

Install sonar-scanner into host OS to analyze the project. (* sonar-scanner need to install JavaVM on the system.)

For MacOS, use homebrew.

```
$ brew install sonar-scanner
```

For other OS, download the package from the following URL.

https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner

### Analyze project

Run the following command from the project root directory to launch the analysis.

```
$ sonar-scanner
```

### Check result of analyzing

Access to http://localhost:9000 again.

## Usage of the container to backup database

### How to change the time to backup

Backup is run by crond. If want to change the default time to run, edit crontab_root file and restart the container.

```
$ cat ./sonarqube-db-backup/crontab_root
0 2 * * * /opt/script/backup.sh
$ vi ./sonarqube-db-backup/crontab_root
$ docker-compose restart sonarqube-db-backup
```

### How to recover sonarqube

#### About data

Restore the backup file to sonarqube-db.

Remove container and database data. Then, restart sonarqube-db-backup (with sonarqube-db) only.

```
$ docker-compose down --rmi all
$ rm -r ./_data/sonarqube/ ./_data/sonarqube-db/
$ docker-compose up -d sonarqube-db-backup
```

Decompress the backup file, and restore it to database.

```
$ docker-compose exec sonarqube-db-backup gzip -dk /var/backup/dump.sql_YYYYMMDD_HHMISS.gz
$ docker-compose exec sonarqube-db-backup sh -c \
  'PGPASSWORD=any_string_user_password \
  psql --host=sonarqube-db \
  --username=any_string_user_name \
  --no-password \
  --dbname=any_string_database_name \
  --file=/var/backup/dump.sql_YYYYMMDD_HHMISS'
```

Finally, restart the whole docker-sonarqube.

```
$ docker-compose up -d
```

#### About plugins

For simplification, there is no backup of plugins.

Copy default plugins, and restart sonarqube to activate the plugins.

```
$ docker-compose exec sonarqube mkdir -p /opt/sonarqube/extensions/plugins/
$ docker-compose exec sonarqube sh -c 'cp -p /opt/sonarqube/lib/bundled-plugins/* /opt/sonarqube/extensions/plugins/'
$ docker-compose restart sonarqube
```

Install additional plugins by manual from [Administration] > [Marketplace] in SonarQube.

### How to upload the backup file to google drive.

GDrive(https://github.com/prasmussen/gdrive) access your Google Account. Authentication is needed to send the backup file to google drive.

1. Execute the command described below to get the url for authentication.
2. Access to the displayed url with browser.
3. Select the same account set to GDRIVE_ACCOUNT in .env file.
4. Copy verification code, and enter it into the console.

```
$ docker-compose exec sonarqube-db-backup gdrive-linux-x64 about
Authentication needed
Go to the following url in your browser:
https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=...

Enter verification code: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
User: ..., your_mail_address@gmail.com
Used: ... GB
Free: ... GB
Total: ... GB
Max upload size: ... TB
$
```

#### How to change the account

If want to change the account to upload, execute following commands.

1. Change GDRIVE_ACCOUNT to the another acount in .env file.
2. Reload environment variables in the container. (/root/.gdrive/ directory that save the data to upload is cleared concomitantly with recreating the container.)
3. Authenticate again with the another acount.

```
$ vi .env   # Change GDRIVE_ACCOUNT
$ docker-compose up -d
$ docker-compose exec sonarqube-db-backup gdrive-linux-x64 about
```

### How to disable backup

If don't need backup, execute following command when run docker-sonarqube.

```
$ docker-compose up -d --scale sonarqube-db-backup=0
```
