# CICD

## Purpose
This repository will contain the Ansible[1] Playbooks (and associated
materials) in order to build a CI/CD environment to deploy a `nodejs` application.

## Architecture
This CI/CD environment has been built using AWS for virtualization. That helps, since it takes away a lot of extra configuring needed. It also beats vagrant because of the accessibilty it gives us. The environment itself now, is built per spec, in a single host. That host is responsible for serving the [`emoji`](https://github.com/ahfarmer/emoji-search) application (`port 3000`) but also for having the tools needed for the integration of the code and the deployment/delivery of  the component. 
So the `EC2` instance working as our host, a virtual server, would need to have its own virtual environments to serve those purposes. Obviously just installing the tools was out of the question. 
The Virtualization I went with here, was with `docker` instead of `KVM` or `LXC` which is what I'm used to working. The reason here is that the `docker-registry` and `docker-hub` for building layers of Virtual Machines, is essential to the speedy delivery of the exercise. Also, the tools in general and the adapting of `Docker` is such that it enables more people to work together. `KVM` here would have been too painful to setup plus an overkill. 
The tool used for building the `nodejs` application was `Jenkins` a fairly common tool which has a huge list of `plugins`, some of which I've used here. 
The tool used for deploying the infra of the overall solution is `Ansible`. It's an agentless configuration management system with again, a very big open source community. I am also quite familliar with the tool which enables me to do extensible components within it without spending too much time. 

So, [`Docker`](https://www.docker.com/), [`Ansible`](https://www.ansible.com/), [`Jenkins`](https://jenkins.io/). 

## Workflow
The [`ansible PlAY`](https://github.com/michnmi/CICD/blob/master/ansible/playbooks/setup_environment.yml) will install `docker` on the host as a first step and then install the tools needed for CI/CD. It does that by building a number of containers that are needed: 
1. [`jenkins-master`](https://github.com/michnmi/CICD/blob/master/ansible/roles/services/docker_CICD/files/Dockerfiles/jenkins-master/Dockerfile)
That container runs the `jenkins` app, and also is capable of connecting to the host's `docker engine`, thus creating more containers on demand. 
2. [`jenkins-slave-ansible`](https://github.com/michnmi/CICD/blob/master/ansible/roles/services/docker_CICD/files/Dockerfiles/jenkins-slave-ansible/Dockerfile)
That container is connected with the `jenkins-master` through `ssh` and can run `ansible` commands
3. [`jenkins-slave-npm`](https://github.com/michnmi/CICD/blob/master/ansible/roles/services/docker_CICD/files/Dockerfiles/jenkins-slave-npm/Dockerfile)
That container is connected with the `jenkins-master` through `ssh` and can run `nodejs` related commands. 
4. `docker registry`
A local repo to contain our images. 

The `jenkins-master` is setup in such a way ([`config.xml`](https://github.com/michnmi/CICD/blob/master/ansible/roles/services/docker_CICD/templates/jenkins-master/config.xml.j2)) that it knows from the beginning which containers it needs and how to bring them up. 
The [jobs](https://github.com/michnmi/CICD/tree/master/ansible/roles/services/docker_CICD/files/jenkins_master/jobs) are pre-built during the deployment. 
What happens then, is that the user, selects the job they care about (in our case [`build_test_emoji`](https://github.com/michnmi/CICD/tree/master/ansible/roles/services/docker_CICD/files/jenkins_master/jobs/build_test_emoji)) and executes it. This is a pipeline job, which means it is controlled by a [`Jenkinsfile`](https://jenkins.io/doc/book/pipeline/jenkinsfile/), a relatively new tool. In our case, the `Jenkinsfile` has several stages that are executed in different containers, all managed by `jenkins-master`, all existing in the overlay host. 

### Steps

1. The `npm` agent (container) is called upon to build and test the `emoji` application. 
2. If the test is successful, then the `docker` agent (`jenkins-master`) is called upon to build an image for the `emoji` app (as seen here: [`Dockerfile`](https://github.com/michnmi/CICD/blob/master/ansible/roles/services/docker_CICD/files/jenkins_master/jobs/build_test_emoji/Dockerfile)). The code for that image is *the same `revision`* that passed the tests before. That image is pushed to the `docker-registry` _tagged_ by the `BUILD_NUMBER`. 
3. If that is successful, then the `ansible` container is called to execute the [`playbook`](https://github.com/michnmi/CICD/blob/master/ansible/playbooks/setup_environment.yml) with `tag: setup_node_app`. For that to succeed, the `ansible` container needs `ssh` access to the `docker host`. It gets that, by utilizing [`Jenkins Credentials`](https://jenkins.io/doc/book/using/using-credentials/). It will deploy the image that was built in the previous step, using the `docker tag` it got. 

## TODO
Not necessarily in order, there are several things that should be considered: 
1. _Who is responsible for the deployments_? 

In this case, it's `Jenkins`. But also `Ansible`. I prefer `Ansible` due to its simplicity and the fact that we can use `Jenkins` to `build`/`test` and push `artifacts` and for `Ansible` to handle the deployment process.  

2. _What about the secrets_? 

In this case, `Jenkins` manages the secrets. It's not that it is bad, it's just that if `Jenkins` is compromised, so are all of our systems (in this model). I would have built another host with [`HashiVault`](https://www.vaultproject.io/), a tool I'm quite familliar with and a tool that's built to specifically manage secrets, thus decoupling the two services. 
In the meantime, I should have used [`ansible vault`](https://docs.ansible.com/ansible/latest/user_guide/vault.html). 

3. _What about the `docker-engine REST API`_? 

I did disable that in [`issue #39`](https://github.com/michnmi/CICD/issues/39). What I needed was my CI to start containers. There was no need to have the `API` open. A very nice article about the privileged flag here: _[Using Docker-in-Docker for your CI or testing environment? Think twice.](http://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)_

4. _What's all this with the `cicd` host and the `IP` placed later on_?

That's because it's a test environment and I was just trying to get it working quickly. It's fiiiiiiiiiiiiiiiiiiiiiiine.  (with more time , I'd have used [Dynamic Inventories](https://docs.ansible.com/ansible/latest/user_guide/intro_dynamic_inventory.html))

5. _Can you version `Jenkinsfiles`?_

Indeed you can. I didn't. Well, kind of: [This is very ugly](https://github.com/michnmi/CICD/blob/master/ansible/roles/services/docker_CICD/files/jenkins_master/jobs/build_test_emoji/config.xml#L29). If I had more time, or maybe later on, I would definitely have used this: [Pipeline script from SCM](https://jenkins.io/doc/book/pipeline/getting-started/#defining-a-pipeline-in-scm)

6. _More repos?_

Definitely. In the last step I have to download all of the roles just to execute a `docker push`. Should have been more modular. 

7. _Last thoughts?_

In real life, I'd never use `Jenkins` for the deployments. Jenkins knows (and rightly so) about the code. It should have no knowledge of the environments. In here it kind of does, through the `jenkins-slave-ansible` component. Had I more time I'd have setup a [`Rundeck`](https://www.rundeck.com/) workflow to do that. It would be triggered by `Jenkins` and then it would push the images further on. 

8. _WAIT! How about the fact that nothing is over `https` here_?

Right... I would have a `CA` built that would be responsible for creating and authenticating certs. `Jenkins` would not have `username` and `password` methods, only `CN` reading. And yes, I would have setup everything to use `https`. 

9. Even though this is a `dev` environment, a _smoke test_ (i.e. a `curl` command) _could_ also be used in the end of the deployment. 
## Structure
___
### `README.md`
This document.

### `ansible`
The directory containing the `ansible` code

#### `ansible/README.md`
The relevant information on how to execute the code provided here. 

#### `ansible/ansible.cfg`
A base Ansible configuration [2] that is assumed when using this
repository.

#### `ansible/inventories/`
A description of the system estate managed by this repository. Each
subdirectory represents a different environment, with the hosts listed in the
file: `hosts.ini`.

#### `ansible/playbooks/`
The Ansible playbooks [3] that can be run against the systems listed in the
`inventories`.

#### `ansible/roles/`
The Ansible roles [4] that are used (by the `playbooks`) to provision the
system estate.

## Usage
- Create an `issue` with a description containing the purpose of the change.

- Clone the repo locally:

    ```
    $ git clone github.com:/michnmi/CICD
    ```

- Create a local branch with the `issue` number as a name:

    ```
    $ git checkout -b issue_#<number> master
    ```

- Make your changes locally.

- Commit your changes, ensuring the message contains the `issue` number reference:

    ```
    $ git commit -m '#<number> Your message here'
    ```

- Push your local branch and create a Merge Request:

    ```
    $ git push origin issue_#<number>
    ```

- Create a GitHub `PR`

- Assign the `PR` to @michnmi and I will review and merge the request.

## References
 - [[1](https://www.ansible.com/)] https://www.ansible.com/
 - [[2](https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html#configuration-file)] https://docs.ansible.com/ansible/latest/installation_guide/intro_configuration.html#configuration-file
 - [[3](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)] https://docs.ansible.com/ansible/latest/user_guide/playbooks.html
 - [[4](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)] https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html
