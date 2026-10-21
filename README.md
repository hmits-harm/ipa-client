# ipa-client

Enroll the hosts in the Ansible inventory group `ipa_client` with the existing
servers in `ipa_server`, using `freeipa.ansible_freeipa.ipaclient`. The domain is
`lab.shared` and the Kerberos realm is `LAB.SHARED`. These are Ansible inventory
groups; the playbook does not create IPA host groups or install IPA servers.

Install the pinned collection on the controller:

```bash
ansible-galaxy collection install -r requirements.yml
```

Client settings live in `inventory/group_vars/ipa_client.yml`. The IPA `admin`
password references `ansible_password`, which is already encrypted with Ansible
Vault in `inventory/all.yml`. Keep passwords encrypted; do not put plaintext
credentials in group variables or command-line extra variables.

Before enrollment, ensure the server is installed and healthy, clients have
static addresses and working package repositories, and client/server FQDNs
resolve correctly through DNS. Clients must reach the server's IdM services and
have synchronized clocks. Inventory hostnames must be lowercase FQDNs; the role
sets the client hostname from the inventory. DNS resolver configuration is a
prerequisite. The role manages time synchronization with its default settings
and enables home directory creation at first login.

The existing `ansible.cfg` selects `inventory/`, SSH user `local_admin`, privilege
escalation, and Vault password file `~/.vault/.vault`. Ensure that Vault file is
available and restricted to its owner. The existing configuration disables SSH
host key checking; for normal use, verify and populate the controller's
`known_hosts` and enable it with `ANSIBLE_HOST_KEY_CHECKING=True`.

Validate and enroll from the repository root:

```bash
ansible-playbook -i inventory/all.yml install-client.yml --syntax-check
ansible-playbook -i inventory/all.yml install-client.yml --list-hosts
ANSIBLE_HOST_KEY_CHECKING=True ansible-playbook -i inventory/all.yml install-client.yml
```

Use `--ask-vault-pass` if you need to supply the Vault password interactively
instead of using the configured password file. Enrollment uses `state: present`
and does not force a join or enable repair. After enrollment, verify identity
resolution on the clients:

```bash
ANSIBLE_HOST_KEY_CHECKING=True ansible -i inventory/all.yml ipa_client -m ansible.builtin.command -a 'id admin'
```

This follows Red Hat's [Ansible client enrollment procedure](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/installing_identity_management/installing-an-identity-management-client-using-an-ansible-playbook_installing-identity-management)
and the collection's [ipaclient role documentation](https://github.com/freeipa/ansible-freeipa/blob/master/roles/ipaclient/README.md).
