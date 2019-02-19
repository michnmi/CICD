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

- I suggest you just create the AWS env first, since you'll need the public IP on your local `~/.ssh.config`. It will be output during the `play`.
    ```
    $ ansible-playbook -i inventories/dev/hosts.ini -l localhost playbooks/setup_environment --skip-tags setup_docker_host,setup_node_app
    ```
- Use the `public_IP` so you can connect to the host. I usually edit the `.ssh/config` file
    ```
    $ cat /home/micham07/.ssh/config 
    Host cicd
        hostname <public_IP>
    ```
- Now (and from now on) you can execute the `playbook` with the tags you care about. I suggest you just stick to the `docker_CICD` role in the beginning
    ```
    $ ansible-playbook -i inventories/dev/hosts.ini -l localhost,docker_hosts playbooks/setup_environment --skip-tags setup_node_app
    ```
    You will have to use though, `-e` with the following variables: 
    `jenkins_master_username` and `jenkins_master_password` that you want. 
    `ansible_ssh_private_key_file` which is the location of your `pem` file, should you be connecting to the `EC2` instances, that way.

- It's now ready for you. Login to `jenkins-master` at `public_IP:8080` and use `username` and `password` you've set above. _Unfortunately_ you will have to create the `ansible` secret manually. At some point I'll implement this: [`Create credentials from groovy`](https://support.cloudbees.com/hc/en-us/articles/217708168-create-credentials-from-groovy) 
- A pipeline for building and testing the `nodejs` app has already been setup and will be executing every 6 hours. 
- When the `emoji` app is up and running. You can reach it at `public_IP:3000`
