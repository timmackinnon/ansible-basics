# Variables and Facts
This section describes some details around using variables and facts in Ansible.

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

## Variables

### Variables in Playbooks
Variables can be defined in-line in playbooks:
```
vars:
  var1: value1
  var2: value2
  var3: value3
```
or can be defined in files and referenced with a *vars_files* tag:
```
vars_files:
 - myvars.yml
```
where _myvars.yml_ would look like:
```
var1: value1
var2: value2
var3: value3
```

### Viewing the value of a variable
To view the value of a variable, you can use the _debug_ module:
```
 - debug: var=var1
```

### Registering variables
It will often be necessary to _register_ a variable in the context of a _playbook_ execution:
```
- name: Capture output of id command
  command: id -un
  register: login
```
To view the contents of the registered variable, you can use the _debug_ module:
```
- name: Capture output of id command
  command: id -un
  register: login
- debug: var=login
```
Running the `login.yml` and the `login-debug.yml` playbooks, you can see the following output:
```
(venv) $ ansible-playbook login.yml

PLAY [Collects the id of the current user] *************************************

TASK [Gathering Facts] *********************************************************
ok: [testserver]

TASK [capture output of id command] ********************************************
changed: [testserver]

PLAY RECAP *********************************************************************
testserver                 : ok=2    changed=1    unreachable=0    failed=0

(venv) $ ansible-playbook login-debug.yml

PLAY [Collects the id of the current user] *************************************

TASK [Gathering Facts] *********************************************************
ok: [testserver]

TASK [capture output of id command] ********************************************
changed: [testserver]

TASK [debug] *******************************************************************************************************************************************
ok: [testserver] => {
    "changed": false,
    "login": {
        "changed": true,
        "cmd": [
            "id",
            "-un"
        ],
        "delta": "<value>",
        "end": "<date>",
        "rc": 0,
        "start": "<date>",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "vagrant",
        "stdout_lines": [
            "vagrant"
        ]
    }
}

PLAY RECAP *********************************************************************
testserver                 : ok=3    changed=1    unreachable=0    failed=0
```
Looking at the `JSON` output from the debug module, you can see:
* _changed_ key is used to determine whether a state change has occurred
* _cmd_ key contains the invoked command
* _rc_ key contains the return code from the executed command
* _stderr_ key contains any text written to stderr
* _stdout_ key contains any text writtent to stdout
* *stdout_lines* contains any text written to stdout split by newlines

### Using registered variables
Here is an example of how to use the _debug_ module to log a particular _key_ from a variable:
```
(venv) $ ansible-playbook login-use-var.yml

PLAY [Collects the id of the current user] *************************************

TASK [setup] *******************************************************************
ok: [testserver]

TASK [capture output of id command] ********************************************
changed: [testserver]

TASK [debug] *******************************************************************
ok: [testserver] => {
    "msg": "Logged in as user vagrant"
}

PLAY RECAP *********************************************************************
testserver                 : ok=3    changed=1    unreachable=0    failed=0
```

## Facts
When Ansible gathers `facts`, it connects to a host and queries the host for all kinds of details about the host, such as:
* CPU architecture
* OS
* IP addresses
* etc.

For a semi-complete list of _facts_, you can have a look [here](https://github.com/lorin/ansible-quickref/blob/master/facts.rst).

A simple example to collect the OS variant:
```
(venv) $ ansible-playbook fact-os.yml

PLAY [Collects the id of the current user] *************************************

TASK [Gathering Facts] *******************************************************************************************************************************************
ok: [testserver]

TASK [debug] *******************************************************************************************************************************************
ok: [testserver] => {
    "ansible_distribution": "Ubuntu",
    "changed": false
}

PLAY RECAP *********************************************************************
testserver                 : ok=2    changed=0    unreachable=0    failed=0
```

A simple Ansible command to collect all `facts` for a system:
```
ansible testserver -m setup
```

A simple Ansible command to filter particular facts for a system:
```
ansible testserver -m setup -a 'filter=ansible_enp*'
```

Examples of how to retrieve the _IPv4_ and _IPv6_ addresses:
```
(venv) $ ansible testserver -m setup -a 'filter=ansible_all_ipv4_addresses'
testserver | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "<IPv4 address>"
        ]
    },
    "changed": false
}
(venv) $ ansible testserver -m setup -a 'filter=ansible_all_ipv6_addresses'
testserver | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv6_addresses": [
            "<IPv6 address>"
        ]
    },
    "changed": false
}
```

## Destroy your Vagrant VM and cleanup interim files
```
(venv) $ vagrant destroy -f
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
(venv) $ deactivate
$ rm -rf .vagrant venv *-console.log
```