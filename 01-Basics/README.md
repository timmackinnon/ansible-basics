# Basic example using VirtualEnv and Vagrant

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
(venv) $ vagrant init ubuntu/xenial64
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.

(venv) $ vagrant up
...
==> default: Loading metadata for box 'ubuntu/xenial64'
    default: URL: https://vagrantcloud.com/ubuntu/xenial64
==> default: Adding box 'ubuntu/xenial64' (<version>) for provider: virtualbox
    default: Downloading: https://vagrantcloud.com/ubuntu/boxes/xenial64/versions/<version>/providers/virtualbox.box
...
```

## Connect into the VM
```
(venv) $ vagrant ssh
...

vagrant@ubuntu-xenial:~$ exit
logout
Connection to 127.0.0.1 closed.
```
## Use Ansible to run a few commands in the VM
Examine the ssh config for the VM:
```
(venv) $ vagrant ssh-config
Host default
  HostName 127.0.0.1
  User vagrant
  Port 2222
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /some/path/ansible-examples/01-Basics/.vagrant/machines/default/virtualbox/private_key
  IdentitiesOnly yes
  LogLevel FATAL
```

Now, try and connect into the VM with raw ssh:
```
(venv) $ ssh vagrant@127.0.0.1 -p 2222 -i .vagrant/machines/default/virtualbox/private_key
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
ECDSA key fingerprint is <key>.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[127.0.0.1]:2222' (ECDSA) to the list of known hosts.
...

vagrant@ubuntu-xenial:~$ exit
logout
Connection to 127.0.0.1 closed.
```

Add VM info to Ansible _hosts_ file:
```
(venv) $ cat hosts
testserver ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222 ansible_ssh_user=vagrant ansible_ssh_private_key_file=.vagrant/machines/default/virtualbox/private_key ansible_python_interpreter=/usr/bin/python3
```

Ping host with Ansible:
```
(venv) $ ansible testserver -i hosts -m ping
testserver | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Move some details to the _ansible.cfg_ file and run a few commands:
```
(venv) $ more ansible.cfg
[defaults]
hostfile = hosts
remote_user = vagrant
private_key_file = .vagrant/machines/default/virtualbox/private_key
host_key_checking = false

(venv) $ more hosts
testserver ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222 ansible_python_interpreter=/usr/bin/python3

(venv) $ ansible testserver -i hosts -m ping
testserver | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

(venv) $ ansible testserver -i hosts -m command -a hostname
testserver | SUCCESS | rc=0 >>
ubuntu-xenial
```

Run a command that requires _sudo_ (-s) to run:
```
(venv) $ ansible testserver -i hosts -m command -s -a "tail -1 /var/log/syslog"
testserver | SUCCESS | rc=0 >>
<date> ubuntu-xenial python3[<pid>]: ansible-command Invoked with removes=None executable=None _uses_shell=False warn=True chdir=None _raw_params=tail -1 /var/log/syslog creates=None
```

## Destroy your Vagrant VM and cleanup interim files
```
(venv) $ vagrant destroy -f
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
(venv) $ deactivate
$ rm -rf Vagrantfile .vagrant ansible.cfg hosts venv *-console.log
```
