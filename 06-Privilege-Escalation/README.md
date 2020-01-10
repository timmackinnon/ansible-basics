# Privilege Escalation
This section shows some of the challenges around privilege escalation with Ansible.

## Set up a virtualenv and install ansible
```
$ virtualenv venv
New python executable in /some/path/venv/bin/python
Installing setuptools, pip, wheel...done.

$ . venv/bin/activate

(venv) $ pip install ansible==2.3.0.0
Collecting ansible==2.3.0.0
...
Successfully installed MarkupSafe-1.0 PyYAML-3.12 ansible-2.3.0.0 asn1crypto-0.24.0 bcrypt-3.1.4 cffi-1.11.4 cryptography-2.1.4 enum34-1.1.6 idna-2.6 ipaddress-1.0.19 jinja2-2.10 paramiko-2.4.0 pyasn1-0.4.2 pycparser-2.18 pycrypto-2.6.1 pynacl-1.2.1 six-1.11.0

(venv) $ ansible --version
ansible 2.3.0.0
...
```

## Start an Ubuntu VM
```
(venv) $ vagrant up
...
==> default: Importing base box 'ubuntu/xenial64'...
...
```
This VM will have two users pre-configured:
* _test1_ with the following sudo permissions:
  * `test1 ALL=(ALL) NOPASSWD: /usr/bin/timedatectl set-timezone`
* _test2_ with the following sudo permissions:
  * `test2 ALL=(ALL) NOPASSWD: ALL`

# Set up users and run a few tests
Push your key to the VM for both test users:
```
ansible-playbook --ask-pass --user test1 copykey.yml
ansible-playbook --ask-pass --user test2 copykey.yml
```
password for user _test1_ is `test1` and for _test2_ is `test2`.

## Running a few tests as user _test1_:
```
(venv) $ ssh test1@127.0.0.1 -p 2222
$ timedatectl
      Local time: <time>
  Universal time: <time>
        RTC time: <time>
       Time zone: Etc/UTC (UTC, +0000)
 Network time on: yes
NTP synchronized: no
 RTC in local TZ: no
$ timedatectl set-timezone America/New_York
-- will request password to escalate privilege

$ sudo timedatectl set-timezone America/New_York
-- will work based on sudo permissions

$ timedatectl
      Local time: <time>
  Universal time: <time>
        RTC time: <time>
       Time zone: America/New_York (EST, -0500)
 Network time on: yes
NTP synchronized: no
 RTC in local TZ: no
```

## Running a few playbooks as user _test1_ and _test2_
The `timezone_1.yml` playbook:
```
---
- name: Test timezone setting
  hosts: all
  become: true
  tasks:
    - name: Set timezone to America/New_York
      timezone: name=America/New_York
```
when run as follows:
```
ansible-playbook -e ansible_ssh_user=test1 timezone_1.yml
```
will perform the following set of operations:
* connect to target host as user _test1_
* _sudo_ to root - `become: true`
* set the timezone to _America/New_York_

```
(venv) $ ansible-playbook -e ansible_ssh_user=test1 timezone_1.yml

PLAY [Test timezone setting] ***************************************************

TASK [Gathering Facts] *********************************************************
ok: [testserver]

TASK [Set timezone to America/New_York] ****************************************
fatal: [testserver]: FAILED! => {"changed": false, "failed": true, "module_stderr": "Shared connection to 127.0.0.1 closed.\r\n", "module_stdout": "sudo: a password is required\r\n", "msg": "MODULE FAILURE", "rc": 1}

PLAY RECAP *********************************************************************
testserver                 : ok=1    changed=0    unreachable=0    failed=1
```
This fails because user _test1_ cannot _sudo_ to root without a password - user _test1_ has limited sudo access.

Trying the same command as user _test2_:
```
(venv) $ ansible-playbook -e ansible_ssh_user=test2 timezone_1.yml

PLAY [Test timezone setting] ***************************************************

TASK [Gathering Facts] *********************************************************
ok: [testserver]

TASK [Set timezone to America/New_York] ****************************************
changed: [testserver]

PLAY RECAP *********************************************************************
testserver                 : ok=2    changed=1    unreachable=0    failed=0
```
This will work due to the wide open _sudoers_ permissions for user _test2_.

The `timezone_2.yml`, which instead uses the _shell_ module to invoke the _timedatectl_ command directly:
```
- name: Test timezone setting
  hosts: all
  tasks:
    - name: Set timezone to America/New_York
      shell: sudo -H -n /usr/bin/timedatectl set-timezone America/New_York
      args:
        warn: false
```
when run as follows:
```
ansible-playbook -e ansible_ssh_user=test1 timezone_2.yml
```
will perform the following set of operations:
* connect to target host as user _test1_
* sudo to root for the single _shell_ execution and set the timezone to _America/New_York_

```
(venv) $ ansible-playbook -e ansible_ssh_user=test1 timezone_2.yml

PLAY [Test timezone setting] ***************************************************

TASK [Gathering Facts] *********************************************************
ok: [testserver]

TASK [Set timezone to America/New_York] ****************************************
changed: [testserver]

PLAY RECAP *********************************************************************
testserver                 : ok=2    changed=1    unreachable=0    failed=0
```
This will work as it takes advantage of traditional _sudoers_ permissions.

To summarize, Ansible modules cannot be used in the case where privilege escalation is required (`become: true`) but sudoers permissions are limited.

More details can be found [here](http://docs.ansible.com/ansible/become.html#can-t-limit-escalation-to-certain-commands).

## Destroy your Vagrant VM and cleanup interim files
```
(venv) $ vagrant destroy -f
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
(venv) $ deactivate
$ rm -rf .vagrant venv *-console.log
```
