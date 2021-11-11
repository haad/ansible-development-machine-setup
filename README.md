# Deploying Manual

## Jalapeno

### Ansible Secrets management

Secrets are managed with `ansible-vault` to make sure _ansible_ can dynamically decrypt vault file place a file to root of this repository named *.vault_pass* with vault password inside.

with *.vault_pass* file present ansible can dynamically access and use variables defined inside `var/jalapeno-secrets.yml`

Editing jalapeno ansible vault

```
ansible-vault edit vars/jalapeno-secrets.yml
```

### Bootstrap

Machine boostrap deploys networking configuration and some basic settings. You can runt it with

```
ansible-playbook -D -v -i inventory/prod/prod bootstrap.jalapeno.yml
```

#### Network Configuration

Persistent network configuration is done with `systemd.network`, we use ansible role `setup.systemd-network`. On server files are placed at `/etc/systemd/network`

### Deployment

For deploying services as:

- LXC daemon
- LXC containers
- Vault
- Haproxy/Nginx
- Jira & confluence
- Gitlab
- Chrony, OpenVPN ...

we use deployment stage

```
ansible-playbook -D -v -i inventory/prod/prod deploy.jalapeno.yml
```

To further limit execution scope you can do following

```
ansible-playbook -D -v -i inventory/prod/prod -t play:lxc-vpn deploy.jalapeno.yml
```

Deployment variables are defined inside *host_vars* `inventory/prod/host_vars/jalapeno`. Currently our inventory looks like this:

```
jalapeno ansible_ssh_host=92.240.236.99 ansible_python_interpreter=/usr/bin/python3 ansible_user=haad

lxc-vpn-cra ansible_ssh_host=172.16.253.5 ansible_user=root ansible_python_interpreter=/usr/bin/python3
lxc-vpn-pygmalios ansible_ssh_host=172.16.253.6 ansible_user=root ansible_python_interpreter=/usr/bin/python3
lxc-vpn-pixel ansible_ssh_host=172.16.253.8 ansible_user=root ansible_python_interpreter=/usr/bin/python3
lxc-vpn-ib ansible_ssh_host=172.16.253.9 ansible_user=root ansible_python_interpreter=/usr/bin/python3
lxc-vpn-freal ansible_ssh_host=172.16.253.2 ansible_user=root ansible_python_interpreter=/usr/bin/python3
lxc-vpn-telco ansible_ssh_host=172.16.253.7 ansible_user=root ansible_python_interpreter=/usr/bin/python3

lxc-vault ansible_ssh_host=172.16.253.20 ansible_user=root ansible_python_interpreter=/usr/bin/python3
lxc-prometheus ansible_ssh_host=172.16.253.21 ansible_user=root ansible_python_interpreter=/usr/bin/python3
lxc-padlock ansible_ssh_host=172.16.253.22 ansible_user=root ansible_python_interpreter=/usr/bin/python3
```

where all _lxc-vpn-*_ containers are used for VPN management. Each is provisioned from `plays/jalapeno/lxc-vpn.yml`

### Services

#### Vault

Vault is deployed as a service for sharing passwords/accounts. Currently runs at `http://172.16.253.20:8200/ui/` but will be placed behind haproxy with SSL later. User/Password for login is inside `jalapeno_secrets.yml`

## Operations

### Adding Route

```
ansible-playbook -D -v -i inventory/prod/prod addroute.jalapeno.yml
```

### Adding VPN Users

Add user public key to `wireguard_peers` variable at `inventory/prod/host_vars/jalapeno` run following playbook.

```
ansible-playbook -D -v -i inventory/prod/prod addvpn-users.jalapeno.yml
```

Copy generated config and send it to user for wireguard setup.