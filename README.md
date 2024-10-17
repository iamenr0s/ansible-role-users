[![Molecule](https://github.com/iamenr0s/ansible-role-users/actions/workflows/molecule.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-users/actions/workflows/molecule.yml) [![Release](https://github.com/iamenr0s/ansible-role-users/actions/workflows/release.yml/badge.svg)](https://github.com/iamenr0s/ansible-role-users/actions/workflows/release.yml) ![Ansible Role](https://img.shields.io/ansible/role/d/iamenr0s/ansible_role_users)

# Ansible Role Users

The objective of this role is to manage the addition of users and groups to your system.

## Requirements

Pip packages listed in  [requirements.txt](https://github.com/iamenr0s/ansible-role-users/blob/master/requirements.txt)

### ansible-vault

Use ansible-vault to encrypt sensitive info from git (SSH keys).

```
cat vars/vault.yml
#encrypt if cleartext (before git commit/push)
ansible-vault encrypt vars/vault.ymlvars/vault.yml

#Edit encrypted file:
ansible-vault edit vars/vault.yml

vi .vault-password
-Enter the password for Ansible Vault from Password Safe

chmod 600 .vault-password

cat << EOF > ansible.cfg
[defaults]
vault_password_file = ./.vault-password
EOF
```

## Role Variables

The default values for the variables are set in [`defaults/main.yml`](https://github.com/iamenr0s/ansible-role-users/blob/master/defaults/main.yml):

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

## Issues and Contributions

If you encounter any issues with this Ansible role or have suggestions for improvements, please open an issue on the repository.

Contributions in the form of pull requests are also welcome. Feel free to contribute to making these Ansible roles better!

## License

This project is licensed under the [MIT License](LICENSE).

## Author Information
