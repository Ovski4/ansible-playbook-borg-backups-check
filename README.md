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
      ansible_ssh_private_key_file: /etc/ansible/ssh_private_key_file
  children:
    backup_servers:
      hosts:
        34_133_111_133:
```

Alternatively, you can update the `main.yml` hosts key directly.

2. Create a `.env` file referencing variables that should not be committed in the `docker-compose.yml` file. Example:

```bash
SSH_KEY=/home/my_username/.ssh/id_rsa
BORG_REPOSITORY_PARENT_FOLDER=/home/backup_user/borg_repositories/
BORG_REPOSITORY_NEXTCLOUD_PASSPHRASE=xxxxxxxxx
BORG_REPOSITORY_JELLYFIN_PASSPHRASE=xxxxxxxxx
# MAIL_* variables can be omitted if send_email_on_failure or send_email_on_success are both set to 'false'
MAIL_HOST=smtp.us.mailgun.org
MAIL_PORT=587
MAIL_PASSWORD=very_secure_password
MAIL_FROM=user.from@some-domain.com
MAIL_TO=user.to@some-domain.com
```

3. Update the `group_vars/backup_servers/config.yml` file with your repository configurations.

- The `file_paths_to_check` key can be used to verify some files are present in the repository as you would expect. The playbook will fail if the listed files are absent.
- The `sql_dump_folder_path` key can be used to ensure a `.sql` dump file was created at the same date as the last repository, in the specified path.
- The `send_email_on_failure` key must be set to true if you want to be alerted of failures by email.
- The `send_email_on_success` key must be set to true if you want to be alerted of success by email.

```yml
send_email_on_failure: true
send_email_on_success: false
borg_repositories:
  - name: jellyfin
    passphrase: "{{ borg_repository_jellyfin_passphrase }}"
    file_paths_to_check:
      - /var/docker_volumes/jellyfin/config/root/default/Movies/options.xml

  - name: nextcloud
    passphrase: "{{ borg_repository_nextcloud_passphrase }}"
    sql_dump_folder_path: /var/docker_volumes/nextcloud # the folder path within the borg repo
```

With the above configuration, the playbook will check:
- if a Borg repository named 'jellyfin' has been created in the past 2 days, and if the options.xml file is present in the backup at the specified path.
- if a Borg repository named 'nextcloud' has been created in the past 2 days, with a `.sql` dump created as well.

4. Update the `docker-compose.yml` file by adding missing environment variables to the command.
5. Run the Ansible playbook using Docker:

```bash
docker compose run --rm --remove-orphans play
```

Store sensitive variables in a vault.yml file
---------------------------------------------

Instead of creating a `.env` file, we can create a vault file with encrypted variables inside.

```
# First we store the password in a file
nano .ansible_vault_pass
chmod 600 .ansible_vault_pass

# Create a vault file using the same password
ansible-vault create group_vars/backup_servers/vault.yml
```

Copy the variables in the `vault.yml` file with the correct values:
```yml
borg_repository_parent_folder: /home/backup_user/borg_repositories/
borg_repository_nextcloud_passphrase: xxxxxxxxx
borg_repository_jellyfin_passphrase: xxxxxxxxx
borg_repository_baptiste_accounts_passphrase: xxxxxxxxx
borg_repository_semaphore_passphrase: xxxxxxxxx
borg_repository_elasticsearch_passphrase: xxxxxxxxx
borg_repository_immich_passphrase: xxxxxxxxx
borg_repository_beszel_passphrase: xxxxxxxxx
# mail_* variables can be omitted if send_email_on_failure or send_email_on_success are both set to 'false'
mail_host: smtp.us.mailgun.org
mail_port: 587
mail_password: very_secure_password
mail_from: user.from@some-domain.com
mail_to: user.to@some-domain.com
```

Then run the command with the specific docker compose file:
```bash
docker compose -f docker-compose-with-vault.yml run --rm --remove-orphans play
```
