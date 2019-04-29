### Sonarqube deployment on Kubernetes GCP.

# Prerequisites

- Bash/PowerShell terminal with kubectl installed

- PostgreSQL database to store SonarQube’s data

- Kubernetes cluster


## 1. Create Postgress DB GCP Cloud SQL.

[Click here](https://console.cloud.google.com/sql) to create postgress db.


## 2. Generate base64 encoded password.
Create a Secret to store PostgreSQL password, Kubernetes has a built-in capability to store secrets. To create a secret you need to base64 encode a secret value.
```
echo -n 'yourpassword' | base64
```
Copy the password and put it in the secret file.

## 3. Applly secret file.
```
kubectl apply -f secret.yaml
```

## 4. Create PVC storage.
We need to create 2 PVCs since SonarQube uses two locations to store data /opt/sonarqube/data/ and /opt/sonarqube/extensions/.

PVC for Sonar’s data directory

```
kubectl apply -f data-pvc.yaml 
```

PVC for Sonar’s extensions directory
```
kubectl apply -f extension-pvc.yaml
```

## 5. Apply deployment file.
After creating PVCs and Postgres secret we are ready to deploy using the following YAML file.
```
kubectl apply -f deployment.yaml
```


## 6. Expose it service to the load balancer.
```
kubectl apply -f service.yaml
```

You can access it by URL. Default username and password will be admin/admin. Please change it after login.


## Plugin Installation.
You have to go to the  Administration > Marketplace and search the plugin name you want and install, Restart server after plugin installed.


# Setting Up the Code Scanner.

SonarQube's code scanner is a separate package that you can install on a different machine than the one running the SonarQube server, such as your local development workstation or a continuous delivery server. There are packages available for Windows, MacOS, and Linux which you can find at the SonarQube web site.

```
mkdir /opt/sonarscanner
```
```
cd /opt/sonarscanner
```

### Download the SonarQube scanner for Linux using wget. you can download as per your OS.
```
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.2.0.1227-linux.zip
```
### Unzip, Extract the scanner.
```
unzip sonar-scanner-cli-3.2.0.1227-linux.zip
```
After that, you will need to modify a few settings to get the scanner working with your server install. Edit the configuration file. Un-comment the line starting with sonar.host.url and add yours.
```
vim sonar-scanner-3.2.0.1227-linux/conf/sonar-scanner.properties
```
```
#Configure here general information about the environment, such as SonarQube server connection details for example
#No information about specific project should appear here

#----- Default SonarQube server
sonar.host.url=https://mysonarqube.com

#----- Default source code encoding
#sonar.sourceEncoding=UTF-8
```
### Make the binary executable.
```
chmod +x sonar-scanner-3.2.0.1227-linux/bin/sonar-scanner
```
### Create the symbolic link so that you don't need to specify full path.
```
ln -s /opt/sonarscanner/sonar-scanner-3.2.0.1227-linux/bin/sonar-scanner /usr/local/bin/sonar-scanner
```
Now the scanner is up and running, Now run your fisrt code scan.

# Run code scan for your project.
Go to your project directory and create a file name "sonar-project.properties", Define the project name, Project key, project version and the current directory.

```
sonar.projectKey=my-project
sonar.projectName=my-project
sonar.projectVersion=1.0

sonar.sources=.
```
Now you can run the code scan from your machine. To run code scan you will need token of the sonarqube server. So create sonarsqube user token first.
```
Go to My account > Security and generate the token.
```

### Run the code scan now. 
```
sonar-scanner -D sonar.login=your_token_here
```
Once the scan is complete, you'll see a summary screen similar to this:
```
INFO: Task total time: 7.933 s
INFO: ------------------------------------------------------------------------
INFO: EXECUTION SUCCESS
INFO: ------------------------------------------------------------------------
INFO: Total time: 19.249s
INFO: Final Memory: 19M/296M
```
### And the project's code quality report will now be on the SonarQube dashboard.


# Gitlab integration with SonarQube.
As with sonnar-scanner, you will need to have a sonar.properties file in your project's root folder.
To run the scan, add the following to your gitlab-ci.yml

```

analysis:
  image: emeraldsquad/sonar-scanner
  stage: analysis
  artifacts:
  script: sonar-scanner -Dsonar.host.url=$SONAR_URL -Dsonar.login=$SONAR_TOKEN
```

### Variables : 

```
SONAR_URL=URL
```
```
SONAR_TOKEN=YOURTOKEN
```
