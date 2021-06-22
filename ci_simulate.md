## Simulating a typical CI/CD Pipeline for a PHP Based application

*image cid/ci piepline

To get started, we will focus on these environments initially.

- Ci
- Dev
- Pentest

What we want to achieve, is having Nginx to serve as a reverse proxy for our sites and tools. Each environment setup is represented in the below table and diagrams.

*image table env

### CI Env

*image for ci

### Other Env

*image for other envs

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

Observations:

