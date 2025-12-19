Ansible playbook to check borg backups
======================================

Roles to check if borg backups are created daily as expected. Tested on ubuntu server 22.04.

Setup
------

### .env

Create a .env file in the repository folder. Update the env variable value as needed.

```
SSH_KEY=/home/user/.ssh/id_rsa
BACKUP_USER=some_user
BORG_REPOSITORY_NEXTCLOUD_PASSPHRASE=mypassphrase
BORG_REPOSITORY_BAPTISTE_ACCOUNTS_PASSPHRASE=mypassphrase
```

Create a hosts file in the repository folder. Update the variable values as needed.

### hosts

```yml
all:

  hosts:

    my_server:
      ansible_host: xx.xxx.xxx.xxx
      ansible_connection: ssh
      ansible_user: root
      ansible_ssh_extra_args: '-o StrictHostKeyChecking=no'

  children:

    backup_servers:
      hosts:
        my_server:
```

### Run the play

```bash
ansible-playbook /var/www/ansible/main.yml
# or
docker compose run play
```
