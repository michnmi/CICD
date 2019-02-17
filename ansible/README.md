# CICD - ansible

## Purpose
Here you have all the code for the project.

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

- You can create everything in one go. 
    ```
    $ ansible-playbook -i inventories/dev/hosts.ini -l localhost,docker_host playbooks/setup_environment
    ```
    You will have to use though, `-e` with the following variables: 
    `jenkins_master_username` and `jenkins_master_password` that you want. 
    `ansible_ssh_private_key_file` which is the location of your `pem` file, should you be connecting to the `EC2` instances, that way
    `emoji_build_version` which if you're setting this up for the first time, it will be `1`.

- You can create just the environment. This will output the `private` and `public` `IPs` of the `EC2` instance. 
    ```
    $ ansible-playbook -i inventories/dev/hosts.ini -l localhost playbooks/setup_environment.yml --skip-tags setup_docker_host,setup_node_app
    ```

- Use the `public_IP` so you can connect to the host. I usually edit the `.ssh/config` file
    ```
    $ cat /home/micham07/.ssh/config 
    Host cicd
        hostname <public_IP>
    ```
- It's now ready for you. Login to `jenkins-master` at `public_IP:8080` and use `username` and `password` you've set above. _Unfortunately_ you will have to create the `ansible` secret manually. At some point I'll implement this: [`Create credentials from groovy`](https://support.cloudbees.com/hc/en-us/articles/217708168-create-credentials-from-groovy) 
- A pipeline for building and testing the `nodejs` app has already been setup and will be executing every 6 hours. 
- There is a chance the `emoji` app is up and running. You can reach it at `public_IP:3000`
