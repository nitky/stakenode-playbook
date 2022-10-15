# stakenode-playbook

Easy server hardening and monitoring for Ethereum Solo Staking.

## 🚅 QuickStart

Clone or download the ansible content on your node server where you want it to run:

```bash
git clone https://github.com/nitky/stakenode-playbook
```

Install the Ansible:

> ⚠️ **Ubuntu 20.04 LTS users should configure the PPA to install Ansible 2.10 later.**
> ```bash
> sudo apt update
> sudo apt install software-properties-common
> sudo apt-add-repository --yes --update ppa:ansible/ansible
> ```

```bash
sudo apt update
sudo apt install ansible
```

Run the playbook:

> ⚠️ **For the node with public IP, change the play of "Apply ufw" in the playbook as needed.** (For example, delete `netdata` rules or stop using `ufw`.)

```bash
cd stakenode-playbook
ansible-galaxy install -r ./requirements.yml
ansible-playbook playbooks/stakenode.yml -i inventory/local.yml -K
```

Check the server performance using Netdata:

```bash
# Open http://[node_internal_ip]:19999 in a browser.
# If you are using Avahi, http://[server_name].local:19999 is also fine. 
```

Try to run `eth-docker` following [the official documentation](https://eth-docker.net/docs/About/Overview/):

> ⚠️ **The docker group grants privileges equivalent to the root user.** - *[Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)* 
>
>In other words, if a security issue exists that allows an attacker to become a non-privileged user, it can cause the same problems as a root user compromise. So, I do not recommend running the following command:
>```bash
> # sudo ./ethd install
>```

```bash
cd ~/eth-docker
sudo ./ethd config
sudo ./ethd start
sudo ./ethd logs -f consensus
sudo ./ethd logs -f execution
```

Generate keys and import them to the client:

```bash 
# Generate both validator(signing) and withdrawal keys:
sudo ./ethd cmd run --rm deposit-cli-new --eth1_withdrawal_address YOURHARDWAREWALLETADDRESS --uid $(id -u)
# Test your mnemonic seed phrase:
# Ensure the deposit_data JSON files are the same.
sudo ./ethd cmd run --rm deposit-cli-existing --folder seed_check --eth1_withdrawal_address YOURHARDWAREWALLETADDRESS --uid $(id -u)
diff -s .eth/validator_keys/deposit_data*.json .eth/seed_check/deposit_data*.json
sudo rm .eth/seed_check/*
# Import keys:
sudo ./ethd keys import
sudo ./ethd keys list
```

> ⚠️ If you want to delete the validator public key in the client, run the following commands:
>```bash
>sudo ./ethd keys delete YOURVALIDATORPUBLICKEY
>sudo ./ethd keys list
>```


Check the dashboard using Grafana:

```bash
# Open http://[node_internal_ip]:3000 in a browser.
# If you are using Avahi, http://[server_name].local:3000 is also fine. 
# - username: admin
# - password: admin
```

## ✨ Features

Easy server hardening & monitoring.

- devsec.hardening
- acct
- auditd
- clamav
- fail2ban
- netdata
- sysstat

## 📌 Requirements

Tested with Ubuntu 22.04 LTS.

- Ubuntu Server 20.04 LTS later
- Python 3.8 later
- Ansible 2.10 later


## 🧪 Evaluation of lynis security scan

Use the following commands to evaluate:

```bash
git clone https://github.com/CISOfy/lynis
sudo chown -R root:root lynis
cd lynis 
sudo ./lynis audit system
```

### Vanilla Ubuntu

```
================================================================================

  Lynis security scan details:

  Hardening index : 64 [############        ]
  Tests performed : 256
  Plugins enabled : 2

  Components:
  - Firewall               [V]
  - Malware scanner        [X]

  Scan mode:
  Normal [V]  Forensics [ ]  Integration [ ]  Pentest [ ]

  Lynis modules:
  - Compliance status      [?]
  - Security audit         [V]
  - Vulnerability scan     [V]

  Files:
  - Test and debug information      : /var/log/lynis.log
  - Report data                     : /var/log/lynis-report.dat

================================================================================
```

### Hardened Ubuntu

```
================================================================================

  Lynis security scan details:

  Hardening index : 86 [#################   ]
  Tests performed : 268
  Plugins enabled : 2

  Components:
  - Firewall               [V]
  - Malware scanner        [V]

  Scan mode:
  Normal [V]  Forensics [ ]  Integration [ ]  Pentest [ ]

  Lynis modules:
  - Compliance status      [?]
  - Security audit         [V]
  - Vulnerability scan     [V]

  Files:
  - Test and debug information      : /var/log/lynis.log
  - Report data                     : /var/log/lynis-report.dat

================================================================================
```

## 📖 References

Special thanks to the following:

- https://github.com/dev-sec/ansible-collection-hardening
- https://github.com/CISOfy/lynis
- https://github.com/eth-educators/eth-docker
- https://github.com/Neo23x0/auditd
- https://github.com/netdata/community
- https://github.com/weareinteractive/ansible-apt
