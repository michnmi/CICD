# CICD - ansible

## Purpose
Here you have all the code for the project. This is WIP 

## Pre-requisites
### AWS account
You need to have an `AWS` account and a `programmatic` user with `admin` access. This build is setup to deploy into `AWS`
### ansible
You need to be able to execute `ansible` commands through your host. 
What you need is the following: 
#### RPMS
- `pip`
#### pip packages
- `ansible`
- `boto3`
- `botocore`

## Usage

- Clone the repo locally:

    ```
    $ git clone github.com:/michnmi/CICD
    ```

- Change to the `ansible` directory:

    ```
    $ cd ansible
    ```

- Create the environment. This will output the `private` and `public` `IPs` of the `EC2` instance. 
    ```
    $ ansible-playbook -i inventories/dev/hosts.ini -l localhost playbooks/setup_environment.yml --skip-tags setup_docker_host
    ```

- Use the `public_IP` so you can connect to the host. I usually edit the `.ssh/config` file
    ```
    $ cat /home/micham07/.ssh/config 
    Host cicd
        hostname <public_IP>
    ```
- Setup the host. 
    ```
    ansible-playbook -i inventories/dev/hosts.ini -l localhost,docker_host playbooks/setup_environment.yml -e "jenkins_master_username=<username> jenkins_master_password=<password>"
    ```
  Use the `-e "ansible_ssh_private_key_file=<your_pem_file>"` if you have one. 
- It's now ready for you. Login to `jenkins-master` at `public_IP:8080` and use the `password` that has been printed out. 
- Setup a new `cloud` for `docker`
- Setup a new template for the new `jenkins-slave` 
- Setup a pipeline for building and testing the `nodejs` app like this 
    ```
    pipeline {
     agent { label 'testing' }
     stages {
          stage('Checkout') {
               steps {
                    git url: 'https://github.com/ahfarmer/emoji-search.git'
               }
          }
          stage('install') {
               steps {
                   sh 'npm install'
               }
          }
          stage('test') {
               steps {
                   sh &'CI=true npm test'
               }
          }
     }
    ```
