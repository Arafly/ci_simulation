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

![](https://github.com/Arafly/ci_simulation/blob/master/assets/initial.png)

![](https://github.com/Arafly/ci_simulation/blob/master/assets/vcs.png)

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

![](https://github.com/Arafly/ci_simulation/blob/master/assets/configure.png)

- Scroll down to Build Configuration section and specify the location of the Jenkinsfile at deploy/Jenkinsfile

- Once you click on "save" you start to see the build process

*image build

> To really appreciate and feel the difference of Cloud Blue UI, it is recommended to try triggering the build again from Blue Ocean interface.

![](https://github.com/Arafly/ci_simulation/blob/master/assets/blue_ocean.png)

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
In order to make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository.
- Navigate to the Ansible project and click on “Scan repository now”
- Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.

![](https://github.com/Arafly/ci_simulation/blob/master/assets/feature_branch.png)

![](https://github.com/Arafly/ci_simulation/blob/master/assets/feature_blueocean.png)

## Additional Task
1. Create a pull request to merge the latest code into the `main branch`
2. After merging the `PR`, go back into your terminal and switch into the `main` branch.
3. Pull the latest change.
4. Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an `echo` command like we have in `build` and `test` stages)
   - Package
   - Deploy
   - Clean up
5. Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch
6. Eventually, your main branch should have a successful pipeline like this in blue ocean

```
$ git checkout -b features/jenkinspipeline-multistage
Switched to a new branch 'features/jenkinspipeline-multistage'
araflyayinde@jenkins:~/ansible$ git branch -vv
  feature/jenkinspipeline-stages      d111d6c [origin/feature/jenkinspipeline-stages] additiona
l Jenkinsfile
* features/jenkinspipeline-multistage a306dfa mod
  master                              a306dfa [origin/master] mod
```

- Enter your deploy/Jenkinsfile and modify

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

    stage('Package') {
      steps {
        script {
          sh 'echo "Packaging Stage at this point"'
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh 'echo "Deploying Stage"'
        }
      }
    }

    stage('Clean up') {
      steps {
        script {
          sh 'echo "Cleaning up..."'
        }
      }
    }
  }

}
```

```
$ git status
On branch features/jenkinspipeline-multistage
Your branch is up to date with 'origin/features/jenkinspipeline-multistage'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   deploy/Jenkinsfile
no changes added to commit (use "git add" and/or "git commit -a")
araflyayinde@jenkins:~/ansible$ git add .
araflyayinde@jenkins:~/ansible$ git commit -m "Jenkinsfile further edit"
[features/jenkinspipeline-multistage feea2e3] Jenkinsfile further edit
 1 file changed, 16 insertions(+), 16 deletions(-)

$ git push
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 2 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 405 bytes | 405.00 KiB/s, done.
Total 4 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To https://github.com/Arafly/ansible_refactor.git
   f642ab8..feea2e3  features/jenkinspipeline-multistage -> features/jenkinspipeline-multistage
```

![](https://github.com/Arafly/ci_simulation/blob/master/assets/multistage.png)

### Running Ansible Playbook from Jenkins

Now that we've got a broad overview of a typical Jenkins pipeline. Let's get the actual Ansible deployment to work by:

- Installing Ansible on Jenkins.
- Installing Ansible plugin in Jenkins UI
- Creating Jenkinsfile from scratch. (Delete all you currently have in there and start all over to get Ansible to run successfully)

You can watch a 10 minutes video <https://youtu.be/PRpEbFZi7nI> to guide you through the entire setup

Note: Ensure that Ansible runs against the Dev environment successfully.

```
$ ansible Dev_webserver -i inventory -m ping
Dev_webserver | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

*Because of the numerous build failure I got on Jenkins. I had to execute my playbook from my Ansible controller, so I can easily debug by appending "-vvv" to my playbook execution.*

> **Gotcha**
Make sure your index.html file is residing inside your "playbooks" folder

```
$ ansible-playbook playbooks/apache.yml -i inventory/dev
PLAY [webservers] *************************************************************************************************************************************************************
TASK [Gathering Facts] ********************************************************************************************************************************************************
ok: [Dev_webserver]
TASK [Install packages] *******************************************************************************************************************************************************
ok: [Dev_webserver]
TASK [Start Apache server] ****************************************************************************************************************************************************
ok: [Dev_webserver]
TASK [Deploy static website] **************************************************************************************************************************************************
ok: [Dev_webserver]
PLAY RECAP ********************************************************************************************************************************************************************
Dev_webserver              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

![](https://github.com/Arafly/ci_simulation/blob/master/assets/dev_build.png)

### Parameterizing Jenkinsfile For Ansible Deployment

To deploy to other environments, we will need to use parameters.

- Update sit inventory with new servers

```
[tooling]
<SIT-Tooling-Web-Server-Private-IP-Address>

[todo]
<SIT-Todo-Web-Server-Private-IP-Address>

[nginx]
<SIT-Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<SIT-DB-Server-Private-IP-Address>
```

- Update Jenkinsfile to introduce parameterization. Below is just one parameter. It has a default value in case if no value is specified at execution. It also has a description so that everyone is aware of its purpose.

```
pipeline {
    agent any

    parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }
```

- In the Ansible execution section, remove the hardcoded inventory/dev and replace with `${inventory}
From now on, each time you hit on execute, it will expect an input.

> Gotcha!
You have to use the "blue ocean' to this properly in action.

![](https://github.com/Arafly/ci_simulation/blob/master/assets/sit_run.png)

![](https://github.com/Arafly/ci_simulation/blob/master/assets/Jenkins-Parameter.png)

- Notice that the default value loads up, but we can now specify which environment we want to deploy the configuration to. Simply type sit and hit Run

![](https://github.com/Arafly/ci_simulation/blob/master/assets/Jenkins-Parameter-Sit.png)

![](https://github.com/Arafly/ci_simulation/blob/master/assets/sit_aftermath.png)

## CI/CD Pipeline for TODO application

We already have tooling website as a part of deployment through Ansible. Here we'll introduce another PHP application to add to the list of software products we are managing in our infrastructure. The good thing with this particular application is that it has unit tests, and it is an ideal application to show an end-to-end CI/CD pipeline for a particular application.

Our goal here is to deploy the application onto servers directly from Artifactory rather than from git.
Use this guide to create an [Ansible role for Artifactory](https://www.howtoforge.com/tutorial/ubuntu-jfrog/) (ignore the Nginx part). Configure Artifactory on Ubuntu 20.04

If all went succesfully. You should have the Jfrog Artifactory UI page, like this:

*image artifactory

### Phase 1 - Prepare Jenkins

1. Fork the repository below into your GitHub account

`https://github.com/darey-devops/php-todo.git`

- On your Jenkins server, install PHP, its dependencies including the Composer tool (Feel free to do this manually at first, then update your Ansible accordingly later)

`
sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}`

2. Install Jenkins plugins
  - Plot plugin
  - Artifactory plugin

- We'll use the plot plugin to the display tests reports, and code coverage-related nformation.

- The Artifactory plugin will be used to easily upload code artifacts into an Artifactory server.
In Jenkins UI configure Artifactory


