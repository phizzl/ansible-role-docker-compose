# Ansible role docker-compose
A role for booting a docker-compose setup

## Requirements
- Hosts should be bootstrapped for ansible usage (have python,...)
- `docker` and `docker-compose` should be installed. This can be archived by using the role [`nickjj.docker`](https://github.com/nickjj/ansible-docker) for example.

## Role Variables

| Variable | Description | Default value |
|----------|-------------|---------------|
| `docker_compose_setups`| See details below | / |
| `docker_logins` | List of docker logins. See official docs. | {} |

### `docker_compose_setups` details

The docker-compose list is used to define the setups to run or terminate per host.   Each item in
the list can have following attributes:

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `dest` | Destination to store the setup | yes | / |
| `git` | See [official docs](https://docs.ansible.com/ansible/latest/modules/git_module.html) | no | / |
| `compose_files` | List of docker-compose files to use. Will be added with the `-f` option to command. Changes will cause the container to restart if already started. | no | [] |
| `definition` | Define docker-compose file in this YAML. You can put more YAML or a string containing YAML here. The generated docker-compose file is automatically appended to `compose_files`. Changes will cause the container to restart if already started. | no | / |
| `state` | state of the setup (present, stopped or absent) | no | present |
| `build` | docker-compose build?  | no | no |
| `pull` | Same as `docker-compose pull`. Recreates application until `recreate` is set to no. | no | no |
| `recreate` | By default containers will be recreated when their configuration differs from the service definition. Setting to `never` ignores configuration differences and leaves existing containers unchanged. Setting to `always` forces recreation of all existing containers.  | no | smart |
| `volumes` | List of volumes to manage. See the [official docs](https://docs.ansible.com/ansible/latest/modules/docker_volume_module.html). | no | [] |
| `networks` | List of networks to manage. See the [official docs](https://docs.ansible.com/ansible/latest/modules/docker_network_module.html). | no | [] |
| `logins` | List of docker logins to manage. See the [official docs](https://docs.ansible.com/ansible/2.4/docker_login_module.html). | no | [] |
| `files` | List of files to manage. For each file item the defined `dest` of this `docker_compose_setups` item is automatically prepended to the file items required `path`. See the [official docs](https://docs.ansible.com/ansible/latest/modules/file_module.html). | no | [] |
| `copies` | List of file and directories to copy. For each copy item the defined `dest` of this `docker_compose_setups` item is automatically prepended to the copy items required `dest`. See the [official docs](https://docs.ansible.com/ansible/latest/modules/copy_module.html). | no | [] |
| `templates` | List of templates to copy. For each copy item the defined `dest` of this `docker_compose_setups` item is automatically prepended to the copy items required `dest`. See the [official docs](https://docs.ansible.com/ansible/latest/modules/template_module.html). You can also pass `vars` that will be accessible in variable `vars` in the template. | no | [] |
| `syncs` | List of `synchronoize` operations. See the [official docs](https://docs.ansible.com/ansible/latest/modules/synchronize_module.html). | no | [] |
| `lineinfiles` | List of `lineinfile` operations. For each item the defined `dest` of this `docker_compose_setups` item is automatically prepended to the lineinfile items required `path`. See the [official docs](https://docs.ansible.com/ansible/latest/modules/lineinfile_module.html). | no | [] |
| `replaces` | List of `replace` operations. For each item the defined `dest` of this `docker_compose_setups` item is automatically prepended to the linefile items required `path`. See the [official docs](https://docs.ansible.com/ansible/latest/modules/replace_module.html). | no | [] |
| `pre_commands` | List of shell commands to run before docker-compose is started or stopped. See also the additional envs vars section below. | no | / |
| `post_commands` | List of shell commands to run after docker-compose is started or stopped. See also the additional envs vars section below. | no | / |
| `restart_on_change` | Option to automatically restart the container if the docker-compose setup has changed. | no | yes |
| `docker_user` | If defined we will try to become the given user | no | / |
| `scale` | Scale services | no | / |


#### `post_commands` and `pre_commands` additional enviroment vars
During the execution of the pre and post commands there are additional environment vars to use in the commands

| Variable | Description |
|----------|-------------|
| `SETUP_STATE`| Contains the given `state` of this `docker_compose_setups` item |
| `SETUP_CHANGED`| Contains `1` if some docker-compose file had changed |
| `SETUP_RESTARTED`| Contains `1` if the setup will be or has been restarted. |
| `SETUP_STOPPED`| Contains `1` if `stopped` of `docker_compose_setups` is set to `yes` |


## Example Playbooks

### Run from git
```yaml
---
- hosts: docker
  roles:
    - role: phizzl.dockercompose
      vars:
        docker_compose_setups:
          - dest: /tmp/test-sentry
            git:
              repo: https://github.com/getsentry/onpremise.git
              version: 10.0.0
            pre_commands:
              - "[ -f installed.lock ] || (CI=1 bash install.sh && touch installed.lock)"
```

### Run by definition
```yaml
---
- hosts: docker
  roles:
    - role: phizzl.dockercompose
      vars:
        docker_compose_setups:
          - dest: /tmp/test-apache
            definition:
              version: '3.3'
              services:
                web:
                  image: phizzl/php:7.4-apache-ubuntu-xenial
            restart_on_change: no
```

### Combine definition and existing file 
```yaml
---
- hosts: docker
  roles:
    - role: phizzl.dockercompose
      vars:
        docker_compose_setups:
          - dest: /tmp/test-apache
            # If you have a `definition` you are required to explicitly 
            # set which files should be additionally included - even the 
            # default `docker-compose.yml` file
            compose_files:
              - docker-compose.yml
            definition:
              version: '3.3'
              services:
                web:
                  image: phizzl/php:7.4-apache-ubuntu-xenial
```

### Run command if container was restarted
```yaml
---
- hosts: docker
  roles:
    - role: phizzl.dockercompose
      vars:
        docker_compose_setups:
          - dest: /tmp/test-apache
            git:
              repo: https://github.com/johndoe/awesome.git
            post_commands:
              - '[ "$SETUP_RESTARTED" == "0" ] || echo Restarted!'
```
