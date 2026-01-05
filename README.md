Ansible playbook to check borg backups
======================================

An Ansible playbook to ensure Borg backup repositories are created daily on 1 or multiple remote servers. Additionally it can also make sure `.sql` dump files are present in the Borg repositories. An email will be sent over SMTP in case of failure.

Requirements
------------

- Ansible or Docker
- SSH access to the target backup servers

> This playbook assumes all Borg repos are located in the same folder.

Usage with Docker
-----------------

1. Create a `hosts` file with a `backup_servers` group. Example:

```yaml
all:
  hosts:
    34_133_111_133:
      ansible_host: 34.133.111.133
      ansible_connection: ssh
      ansible_user: root
      ansible_ssh_extra_args: '-o StrictHostKeyChecking=no'
      ansible_ssh_private_key_file: /root/.ssh/id_rsa
  children:
    backup_servers:
      hosts:
        34_133_111_133:
```

Alternatively, you can update the `main.yml` hosts key directly.

2. Create a `.env` file referencing variables that should not be committed in the `docker-compose.yml` file. Example:

```
MAIL_HOST=smtp.us.mailgun.org
MAIL_PORT=587
MAIL_PASSWORD=very_secure_password
MAIL_FROM=user.from@some-domain.com
MAIL_TO=User To <user.to@some-domain.com>
SSH_KEY=/home/my_username/.ssh/id_rsa
BACKUP_USER=backup_user
BORG_REPOSITORY_PARENT_FOLDER=/home/backup_user/borg_repositories/
BORG_REPOSITORY_NEXTCLOUD_PASSPHRASE=xxxxxxxxx
BORG_REPOSITORY_JELLYFIN_PASSPHRASE=xxxxxxxxx
```

3. Update the `group_vars/backup_servers.yml` file with your repository configurations.

```
backup_configs:
  - borg_repository_name: jellyfin
    borg_repository_passphrase: "{{ borg_repository_jellyfin_passphrase }}"

  - borg_repository_name: nextcloud
    borg_repository_passphrase: "{{ borg_repository_nextcloud_passphrase }}"
    sql_dump:
      dump_folder_path_within_borg_repository: /var/docker_volumes/nextcloud
```

With the above configuration, the playbook will check:
- if a Borg repository named 'jellyfin' has been created in the past 2 days.
- if a Borg repository named 'nextcloud' has been created in the past 2 days, with a `.sql` dump created as well.

4. Update the `docker-compose.yml` file by adding missing environment variables to the command.
5. Run the Ansible playbook using docker:

```bash
docker compose run --rm --remove-orphans play
```
