[![Molecule](https://github.com/iamenr0s/ansible-role-users/actions/workflows/molecule.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-users/actions/workflows/molecule.yml) ![Ansible Role](https://img.shields.io/ansible/role/d/iamenr0s/ansible_role_users) [![CodeFactor](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-users/badge)](https://www.codefactor.io/repository/github/iamenr0s/ansible-role-users)

Ansible Role: Users
====================

This role manages the creation, modification, and removal of users and groups on Linux systems, including sudoers entries and SSH key provisioning.

Features
--------
- Creates, updates, and removes users and groups (including system accounts).
- Manages sudoers entries per-user and per-group via `/etc/sudoers.d/`.
- Sets `authorized_keys` from a literal list or from an `ansible-vault`-encrypted public key store.
- Generates an SSH keypair for a user via `community.crypto.openssh_keypair`.

Requirements
------------
- Collections listed in [`requirements.yml`](requirements.yml) (`community.crypto`, for SSH keypair generation): `ansible-galaxy collection install -r requirements.yml`
- Pip packages listed in [`requirements.txt`](requirements.txt), for local testing.

### ansible-vault

Use ansible-vault to encrypt sensitive info from git (SSH public keys).

```
cat vars/vault_public_keys.yml
#encrypt if cleartext (before git commit/push)
ansible-vault encrypt vars/vault_public_keys.yml

#Edit encrypted file:
ansible-vault edit vars/vault_public_keys.yml

vi .vault-password
-Enter the password for Ansible Vault from Password Safe

chmod 600 .vault-password

cat << EOF > ansible.cfg
[defaults]
vault_password_file = ./.vault-password
EOF
```

Supported Platforms
--------------------
- AlmaLinux 8, 9, 10
- Debian 12, 13
- Fedora 42, 43, 44
- Rocky 8, 9, 10
- Ubuntu 22.04, 24.04

## Role Variables

The default values for the variables are set in [`defaults/main.yml`](https://github.com/iamenr0s/ansible-role-users/blob/main/defaults/main.yml):

```
default_user_shell: /bin/bash

users_create_home: true

users_sudo_includedir: "#includedir /etc/sudoers.d"

default_user_group: "users"

users_home_base_dir: "/home"
```

## Dependencies

None

## Example Playbook

```
- name: Converge
  hosts: all
  roles:
    - role: ansible_role_users
      # You can create groups:
      users_groups:
        - name: mygroup
          gid: 1024
        - name: users
        # You can also remove groups.
        - name: notgroup
          state: absent
          # A system group is also possible.
        - name: systemgroup
          system: true
      # You can create users.
      users:
        - name: myuser
          comment: My User
          uid: 1024
          manage_ssh_key: true
          # The `group` and `groups` listed here should exist.
          group: mygroup
          # groups: A list of groups
          groups:
            - users
          sudo_options: "ALL=(ALL) NOPASSWD: ALL"
          # Adding an authorized key.
          authorized_keys:
            - "ssh-rsa ABC123"
          expires: -1
          password_validity_days: 9
        # Here a user is removed.
        - name: notuser
          state: absent
        - name: keyuser
          manage_ssh_key: true
        - name: privkeyuser
          # This user will have ssh-keys generated.
          manage_ssh_key: true
          copy_private_key: true
        - name: multiplekeys
          authorized_keys:
            - "ssh-rsa ABC1234"
            - "ssh-rsa ABC12345"
        - name: passuser
          # You can set a password. (Hashed and salted.)
          password: "$6$mysecretsalt$qJbapG68nyRab3gxvKWPUcs2g3t0oMHSHMnSKecYNpSi3CuZm.GbBqXO8BE6EI6P1JUefhA0qvD7b5LSh./PU1"
          update_password: on_create
        - name: systemuser
          system: true
        - name: multisudo
          # An account that can run just a few commands without a password.
          sudo_options:
            - "ALL= NOPASSWD: /usr/bin/systemctl restart httpd"
            - "ALL= NOPASSWD: /usr/bin/systemctl start httpd"
            - "ALL= NOPASSWD: /usr/bin/systemctl stop httpd"
        - name: myprivkeyuser
          private_keys:
            - name: id_rsa
              content: |
                -----BEGIN OPENSSH PRIVATE KEY-----
                ...
                ...
                -----END OPENSSH PRIVATE KEY-----
```

Contributing & Security
------------------------
- Contributions are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).
- Report vulnerabilities privately per [SECURITY.md](SECURITY.md); do not open public issues for them.

CI & Release (maintainers)
---------------------------
A single workflow (`.github/workflows/molecule.yml`) runs lint and the full Molecule distro matrix on pushes to `main`, PRs, and `v*` tags. On `v*` tags, a `release` job publishes to Ansible Galaxy after all tests pass.

The Galaxy API key lives in the `galaxy` GitHub environment, which only `v*` tags may target. One-time setup:

```bash
# Galaxy publishing key (environment-scoped, get it from galaxy.ansible.com/ui/token)
gh secret set GALAXY_API_KEY --env galaxy --repo iamenr0s/ansible-role-users

# Code scanning notifications (Slack webhook URL; for Discord append /slack to the webhook URL)
gh secret set SECURITY_ALERT_WEBHOOK --env galaxy --repo iamenr0s/ansible-role-users

# ansible-vault password used to decrypt vars/vault_public_keys.yml during Molecule CI
gh secret set ANSIBLE_VAULT_PASSWORD --repo iamenr0s/ansible-role-users
```

`.github/workflows/code-scanning-notify.yml` polls the code-scanning API every 6 hours and posts new or updated open alerts to that webhook (GitHub Actions cannot trigger on `code_scanning_alert` directly).

To release: tag a commit `vX.Y.Z` and push the tag — CI gates the Galaxy publish.

## License

This project is licensed under the [MIT License](LICENSE).

## Author Information

Author: iamenr0s
Galaxy: `iamenr0s.ansible_role_users`
