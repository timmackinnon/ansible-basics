# Simple Playbook Example
This is a simple playbook example for configuring and starting an `nginx` web server using Ansible.

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
==> default: Forwarding ports...
    default: 80 (guest) => 8080 (host) ...
    default: 443 (guest) => 8443 (host) ...
    default: 22 (guest) => 2222 (host) ...
...
```
NOTE: ports `80` and `443` (guest) are forwarded to ports `8080` and `8443` (host) respectively

## Test connectivity with VM
The contents of the `hosts` file are as follows:
```
[webservers]
testserver ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222 ansible_python_interpreter=/usr/bin/python3
```
A group called _[webservers]_ has been defined. This allows you to address a _host_ (testserver) or a _group_ of _hosts_ when issuing commands:
```
(venv) $ ansible testserver -m ping
testserver | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
(venv) $ ansible webservers -m ping
testserver | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Anatomy of web-tls.yml playbook
`YAML` files are supposed to start with three dashes `---` but if they do not, Ansible will not complain.

Comments in `YAML` files should be prefixed with `#`.

An Ansible _playbook_ is essentially a list of Python dictionaries that outine a _play_.

In the example included here (web-tls.yml) we define a:
* _name_ which is a comment that describes what the _play_ is about
```
- name: Configure webserver with nginx and tls
```
* a group of _hosts_ to target
```
  hosts: webservers
```
* whether to use _become_ (sudo to root) when executing the _tasks_
```
  become: true
```
* a group of _vars_ which can be referenced via the _{{ var }}_ pattern
```
  vars:
    key_file: /etc/nginx/ssl/nginx.key
    cert_file: /etc/nginx/ssl/nginx.crt
    conf_file: /etc/nginx/sites-available/default
    server_name: localhost
```
* a list of tasks each outlined with a _name_, a _key_ which is the name of the module and a _value_ which is the args to that module
```
  tasks:
    - name: Install nginx
      apt: name=nginx update_cache=yes cache_valid_time=3600
    - name: create directories for TLS certificates
      file: path=/etc/nginx/ssl state=directory
    - name: Copy TLS key
      copy: src=files/nginx.key dest={{ key_file }} owner=root mode=0600
      notify: restart nginx
    - name: Copy TLS certificate
      copy: src=files/nginx.crt dest={{ cert_file }}
      notify: restart nginx
    - name: Copy nginx config file
      template: src=templates/nginx.conf.j2 dest={{ conf_file }}
      notify: restart nginx
    - name: Enable configuration
      file: dest=/etc/nginx/sites-enabled/default src={{ conf_file }} state=link
      notify: restart nginx
    - name: Copy index.html
      template: src=templates/index.html.j2 dest=/usr/share/nginx/html/index.html mode=0644
```
* a _handler_ that can be notified and executed at the conclusion of a task if that task has changed the _state_ of the system. Handlers only run after all _tasks_ have been run and are only run once.
```
  handlers:
    - name: restart nginx
      service: name=nginx state=restarted
```

## Generating the key and certificate for the nginx server
```
(venv) $ mkdir files
(venv) $ openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -subj /CN=localhost -keyout files/nginx.key -out files/nginx.crt
```

## Running the web-tls.yml playbook
```
(venv) $ ansible-playbook web-tls.yml

PLAY [Configure webserver with nginx and tls] **********************************

TASK [Gathering Facts] *********************************************************
ok: [testserver]

TASK [Install nginx] ***********************************************************
changed: [testserver]

TASK [Create directories for TLS certificates] *********************************
changed: [testserver]

TASK [Copy TLS key] ************************************************************
changed: [testserver]

TASK [Copy TLS certificate] ****************************************************
changed: [testserver]

TASK [Copy nginx config file] **************************************************
changed: [testserver]

TASK [Enable configuration] ****************************************************
ok: [testserver]

TASK [Copy index.html] *********************************************************
changed: [testserver]

RUNNING HANDLER [restart nginx] ************************************************
changed: [testserver]

PLAY RECAP *********************************************************************
testserver                 : ok=9    changed=7    unreachable=0    failed=0
```

And navigate to `https://localhost:8443` to see the end result.

## Destroy your Vagrant VM and cleanup interim files
```
(venv) $ vagrant destroy -f
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
(venv) $ deactivate
$ rm -rf .vagrant files venv *-console.log
```