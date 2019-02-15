# CICD

## Purpose
This repository will contain the Ansible[1] Playbooks (and associated
materials) in order to build a CICD environment to deploy a `nodejs` application.

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
