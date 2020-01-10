# Multiple Hosts Playbook Example
This is a simple playbook example for configuring and starting an `nginx` web server using Ansible on 3 hosts

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

## Start 3 Ubuntu VMs
```
(venv) $ vagrant up
Bringing machine 'vagrant1' up with 'virtualbox' provider...
Bringing machine 'vagrant2' up with 'virtualbox' provider...
Bringing machine 'vagrant3' up with 'virtualbox' provider...
==> vagrant1: Importing base box 'ubuntu/xenial64'...
...
==> vagrant1: Forwarding ports...
    vagrant1: 80 (guest) => 8080 (host) ...
    vagrant1: 443 (guest) => 8443 (host) ...
    vagrant1: 22 (guest) => 2222 (host) ...
==> vagrant1: Booting VM...
...
==> vagrant2: Importing base box 'ubuntu/xenial64'...
...
==> vagrant2: Forwarding ports...
    vagrant2: 80 (guest) => 8081 (host) ...
    vagrant2: 443 (guest) => 8444 (host) ...
    vagrant2: 22 (guest) => 2200 (host) ...
==> vagrant2: Booting VM...
...
==> vagrant3: Importing base box 'ubuntu/xenial64'...
...
==> vagrant3: Forwarding ports...
    vagrant3: 80 (guest) => 8082 (host) ...
    vagrant3: 443 (guest) => 8445 (host) ...
    vagrant3: 22 (guest) => 2201 (host) ...
==> vagrant3: Booting VM...
...
```
__NOTE:__
* ports `80` and `443` (guest) are forwarded to ports `8080` and `8443` (host) respectively for VM 1
* ports `80` and `443` (guest) are forwarded to ports `8081` and `8444` (host) respectively for VM 2
* ports `80` and `443` (guest) are forwarded to ports `8082` and `8445` (host) respectively for VM 3

## Test connectivity with VMs
The contents of the `hosts` file are as follows:
```
[webservers]
testserver1 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222 ansible_python_interpreter=/usr/bin/python3
testserver2 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2200 ansible_python_interpreter=/usr/bin/python3
testserver3 ansible_ssh_host=127.0.0.1 ansible_ssh_port=2201 ansible_python_interpreter=/usr/bin/python3
```

A group called _[webservers]_ has been defined. This allows you to address a _host_ (testserver1) or a _group_ of _hosts_ when issuing commands:
```
(venv) $ ansible testserver1 -m ping
testserver1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
(venv) $ ansible webservers -m ping
testserver1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
testserver2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
testserver3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Generating the key and certificate for the nginx server
```
(venv) $ mkdir files
(venv) $ openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -subj /CN=localhost -keyout files/nginx.key -out files/nginx.crt
```

## Running the web-tls.yml playbook
```
(venv) $ ansible-playbook web-tls.yml
...

PLAY [Configure webserver with nginx and tls] **********************************

TASK [Gathering Facts] *********************************************************
ok: [testserver2]
ok: [testserver1]
ok: [testserver3]

TASK [Install nginx] ***********************************************************
changed: [testserver3]
changed: [testserver1]
changed: [testserver2]

TASK [Create directories for TLS certificates] *********************************
changed: [testserver1]
changed: [testserver3]
changed: [testserver2]

TASK [Copy TLS key] ************************************************************
changed: [testserver2]
changed: [testserver3]
changed: [testserver1]

TASK [Copy TLS certificate] ****************************************************
changed: [testserver2]
changed: [testserver1]
changed: [testserver3]

TASK [Copy nginx config file] **************************************************
changed: [testserver1]
changed: [testserver2]
changed: [testserver3]

TASK [Enable configuration] ****************************************************
ok: [testserver1]
ok: [testserver2]
ok: [testserver3]

TASK [Copy index.html] *********************************************************
changed: [testserver1]
changed: [testserver2]
changed: [testserver3]

RUNNING HANDLER [restart nginx] ************************************************
changed: [testserver1]
changed: [testserver2]
changed: [testserver3]

PLAY RECAP *********************************************************************
testserver1                   : ok=9    changed=7    unreachable=0    failed=0
testserver2                   : ok=9    changed=7    unreachable=0    failed=0
testserver3                   : ok=9    changed=7    unreachable=0    failed=0
```

And navigate to `https://localhost:8443`, `https://localhost:8444` and `https://localhost:8445` to see the end result.

## Destroy your Vagrant VMs and cleanup interim files
```
(venv) $ vagrant destroy -f
==> vagrant3: Forcing shutdown of VM...
==> vagrant3: Destroying VM and associated drives...
==> vagrant2: Forcing shutdown of VM...
==> vagrant2: Destroying VM and associated drives...
==> vagrant1: Forcing shutdown of VM...
==> vagrant1: Destroying VM and associated drives...
(venv) $ deactivate
$ rm -rf .vagrant files venv *-console.log
```
