# tomcat-playbook
Playbooks for managing Tomcat and a Jenkins pipeline to test and deploy it to Ansible Tower or AWX.

The Jenkins pipeline is the file called Jenkinsfile.
To be able to run the pipeline, you need to first install tower-cli and ansible-lint on your Jenkins server/slave. To do that run the following on your Jenkins server/slave:
```
# yum install python2-pip
# pip2 install ansible-tower-cli
# pip2 install ansible-lint
```
Next you configure the tower-cli to automatically login. For information on how to configure tower-cli, visit: http://docs.ansible.com/ansible-tower/latest/html/towerapi/tower_cli.html
