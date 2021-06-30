## Simulating a typical CI/CD Pipeline for a PHP Based application

![](CI_CD-Pipeline-For-PHP-ToDo-Application.png)

To get started, we will focus on these environments initially.

- Ci
- Dev
- Pentest

What we want to achieve, is having Nginx to serve as a reverse proxy for our sites and tools. Each environment setup is represented in the below table and diagrams.

![](https://github.com/Arafly/ci_simulation/blob/master/assets/Environment-setup.png)

### CI Env

![](Project-14-CI-Environment.png)

### Other Env

![](https://github.com/Arafly/ci_simulation/blob/master/assets/Project-14-Pentest-Environment.png)

<!-- DNS requirements
Make DNS entries to create a subdomain for each environment. Assuming your main domain is darey.io

You should have a subdomains list like this:

Server	Domain
Jenkins	https://ci.infradev.darey.io
Sonarqube	https://sonar.infradev.darey.io
Artifactory	https://artifacts.infradev.darey.io
Production Tooling	https://tooling.darey.io
Pre-Prod Tooling	https://tooling.preprod.darey.io
Pentest Tooling	https://tooling.pentest.darey.io
UAT Tooling	https://tooling.uat.darey.io
SIT Tooling	https://tooling.sit.darey.io
Dev Tooling	https://tooling.dev.darey.io
Production TODO-WebApp	https://todo.darey.io
Pre-Prod TODO-WebApp	https://todo.preprod.darey.io
Pentest TODO-WebApp	https://todo.pentest.darey.io
UAT TODO-WebApp	https://todo.uat.darey.io
SIT TODO-WebApp	https://todo.sit.darey.io
Dev TODO-WebApp	https://todo.dev.darey.io -->

To get started, access your home folder and create a new directory to hold your Ansible files:

```
cd ~
mkdir ansible
```

Move to that directory and open a new inventory file using your text editor of choice. Here, we’ll use nano:

```
cd ansible
mkdir inventory
```

Then create inside inventory folder the environment's inventories

Your ansible Inventory should look like this:

```
~/ansible/inventory$ tree -L 2
.
├── ci
├── dev
├── pentest
├── pre-prod
├── prod
├── sit
└── uat
```

### ci inventory file

```
[jenkins]
<Jenkins-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[sonarqube]
<SonarQube-Private-IP-Address>

[artifact_repository]
<Artifact_repository-Private-IP-Address>
```

### dev Inventory file

```
[tooling]
<Tooling-Web-Server-Private-IP-Address>

[todo]
<Todo-Web-Server-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<DB-Server-Private-IP-Address>
```

### pentest inventory file

```
[pentest:children]
pentest-todo
pentest-tooling

[pentest-todo]
<Pentest-for-Todo-Private-IP-Address>

[pentest-tooling]
<Pentest-for-Tooling-Private-IP-Address>
```

### Ansible Roles for CI Environment

Add two more roles to ansible:

- SonarQube
- Artifactory

SonarQube is an open-source platform developed by SonarSource for continuous inspection of code quality, it is used to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities.

Artifactory on the other hand, is a product by JFrog that serves as a binary repository manager. The binary repository is a natural extension to the source code repository, in that the outcome of your build process is stored.

### Configuring Ansible For Jenkins Deployment

In the previous projects, we've been launching Ansible commands manually from the CLI. Now, with Jenkins, we'll do so from Jenkins UI.

In order to achieve this, you must:
- Navigate to Jenkins URL
- Install & Open Blue Ocean Jenkins Plugin

- Create a new pipeline

*image initial
*image vcs

- connect Jenkins to a version control (GitHub) with an access token

*image access_token

- Paste back the token from GitHub and create a new Pipeline

*image blue_ocean
*image project

At this point you may not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give you some guidance to create one. But we do not need that. We will rather create one ourselves. So, click on Administration to exit the Blue Ocean console.


### Create Jenkinsfile
- Inside the Ansible project, create a new directory deploy and start a new file Jenkinsfile inside the directory.
- Add the code snippet below to start building the Jenkinsfile gradually. This pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage

```
pipeline {
    agent any


  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}
```

- Return to the Ansible pipeline in Jenkins, and select configure

*image configure

- Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile

- Once you click on "save" you start to see the build process

*image build

> To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.

*image blue-ocean

Notice that this pipeline is a multibranch one. This means, if there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.

We can demonstrate this by:

1.Create a new git branch and name it feature/jenkinspipeline-stages

2. We only have the Build stage. Let's add another stage called Test. 
Paste the code snippet below and push the new changes to GitHub.


```
 pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}
```


