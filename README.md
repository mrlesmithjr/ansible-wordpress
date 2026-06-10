# ansible-wordpress

An Ansible role that creates the WordPress database and user, downloads and extracts WordPress, fetches security salts, and writes `wp-config.php`. Apache and PHP must be installed separately before this role runs.

## Requirements

**Ansible:** 2.14 or later

**Collection:** Install `ansible.mysql` (>= 5.0.0) before running the role:

```bash
ansible-galaxy collection install -r requirements.yml
```

**Python on target hosts:** The role installs `python3-pymysql` on Debian and RedHat family hosts automatically via `db.yml`. No manual pre-installation is needed.

**Apache and PHP:** This role does not install or configure Apache or PHP. Both must be present and configured on the target host before this role runs. See the [Dependencies](#dependencies) section for the companion roles used in the reference playbook.

## Role Variables

| Name | Default | Description |
|---|---|---|
| `wordpress_db` | `wordpress` | Name of the WordPress database to create |
| `wordpress_db_user` | `wordpress` | MySQL user granted access to the WordPress database |
| `wordpress_db_password` | `""` | Password for `wordpress_db_user`. **Must be set before use.** |
| `wordpress_db_host` | `localhost` | MySQL host used for database and user creation |
| `wordpress_db_host_user` | `""` | Privileged MySQL user used to create the database and user. **Must be set before use.** |
| `wordpress_db_host_password` | `""` | Password for `wordpress_db_host_user`. **Must be set before use.** |
| `wordpress_db_table_prefix` | `wp_` | WordPress database table prefix written into `wp-config.php` |
| `wordpress_version` | `6.5` | WordPress version to download from wordpress.org |
| `wordpress_dl_uri` | `https://wordpress.org` | Base URI for the WordPress download |
| `wordpress_dl_file` | `wordpress-{{ wordpress_version }}.tar.gz` | Archive filename constructed from the version |
| `wordpress_dl` | `{{ wordpress_dl_uri }}/{{ wordpress_dl_file }}` | Full download URL |
| `wordpress_url_path` | `/` | URL path written into `wp-config.php` |

### Security note

`wordpress_db_password`, `wordpress_db_host_password`, and `wordpress_db_host_user` all default to empty strings and will cause the role to fail or produce an insecure installation if left unset. Store these values in an Ansible Vault-encrypted file and never commit plaintext passwords to version control.

```bash
ansible-vault encrypt_string 'yourpassword' --name wordpress_db_password
```

## Dependencies

The following roles from `requirements.yml` provide the Apache and MySQL services this role depends on at runtime:

- [mrlesmithjr/ansible-apache2](https://github.com/mrlesmithjr/ansible-apache2) - Apache web server
- [mrlesmithjr/ansible-mysql](https://github.com/mrlesmithjr/ansible-mysql) - MySQL server
- [mrlesmithjr/ansible-php](https://github.com/mrlesmithjr/ansible-php) - PHP and modules

Install all role dependencies:

```bash
ansible-galaxy install -r requirements.yml
```

## Example Playbook

The following example mirrors the reference `playbook.yml` included in this repository. It assumes a two-host inventory with a `wordpress_db` group (MySQL) and a `wordpress_app` group (Apache + PHP + WordPress).

```yaml
- hosts: wordpress_db
  vars:
    mysql_allow_remote_connections: true
  roles:
    - role: ansible-mysql

- hosts: wordpress_app
  vars:
    apache2_config: true
    apache2_config_php: true
    apache2_install_php: true
    apache2_virtual_hosts:
      - documentroot: /var/www/html/wordpress
        default_site: true
        port: 80
        serveradmin: admin@example.com
        servername: ""
    wordpress_db_host: "{{ groups['wordpress_db'][0] }}"
    wordpress_db_host_user: root
    wordpress_db_host_password: "{{ vault_mysql_root_password }}"
    wordpress_db_password: "{{ vault_wordpress_db_password }}"
  roles:
    - role: ansible-apache2
    - role: ansible-php
      become: true
    - role: ansible-wordpress
```

The `vault_*` variables should be defined in a Vault-encrypted vars file.

## License

MIT

## Author

Larry Smith Jr. ([@mrlesmithjr](https://github.com/mrlesmithjr))
