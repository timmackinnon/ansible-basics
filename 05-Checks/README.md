# Checks
Here are some examples of writing system checks with Ansible.

## Structure of a check
The Ansible [Assert module](http://docs.ansible.com/ansible/assert_module.html) is a simple way to write checks to validate system configuration. This combined with Ansible **facts** and the [Shell](http://docs.ansible.com/ansible/shell_module.html) and [Script](http://docs.ansible.com/ansible/script_module.html) modules provides a powerful toolset for introspecting systems.

### Example - Ansible Fact
```
tasks:
  - name: Ensure OS is {{ valid_os_name }}
    assert:
      that: "ansible_distribution == valid_os_name"
       msg: "OS is not valid - is {{ ansible_distribution }} but must be {{ valid_os_name }}"
```

### Example - Shell
```
tasks:
  - name: Check disk space of root filesystem
    shell: |
      df -BG / | awk '/^[^Filesystem]/ {print $2}' | sed 's/G$//'
    register: root_filesystem_size

  - name: Ensure root filesystem is at least {{ root_filesystem_min_size }}G
    assert:
      that: "root_filesystem_size.stdout | int >= root_filesystem_min_size"
      msg: "/ is {{ root_filesystem_size.stdout }}G in size - fails minimum {{ root_filesystem_min_size }}G check"
```

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

## Running the system-checks.yml playbook
```
(venv) $ ansible-playbook system-checks.yml

PLAY [System Checks] ***********************************************************

TASK [Gathering Facts] *********************************************************
ok: [testserver]

TASK [debug] *******************************************************************
ok: [testserver] => {
    "changed": false,
    "msg": "OS name is Ubuntu"
}

TASK [Ensure OS is Ubuntu] *****************************************************
ok: [testserver] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [debug] *******************************************************************
ok: [testserver] => {
    "changed": false,
    "msg": "OS arch is x86_64"
}

TASK [Ensure OS architecture is x86_64] ****************************************
ok: [testserver] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [debug] *******************************************************************
ok: [testserver] => {
    "changed": false,
    "msg": "OS version is 16.04"
}

TASK [Ensure OS version is one of [u'14.04', u'16.04']] ************************
ok: [testserver] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [Check disk space of root filesystem] *************************************
ok: [testserver]

TASK [debug] *******************************************************************
ok: [testserver] => {
    "changed": false,
    "msg": "/ is 10G in size"
}

TASK [Ensure root filesystem is at least 10G] **********************************
ok: [testserver] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [Install Docker] **********************************************************
ok: [testserver]

TASK [Check Docker version] ****************************************************
changed: [testserver]

TASK [Ensure docker version is 1.9.1] ******************************************
fatal: [testserver]: FAILED! => {
    "assertion": "installed_docker_version.stdout == valid_docker_version",
    "changed": false,
    "evaluated_to": false,
    "failed": true,
    "msg": "Docker 1.9.1 is not installed. Installed version is 1.5-1"
}

PLAY RECAP *********************************************************************
testserver                 : ok=12   changed=1    unreachable=0    failed=1
```

## Destroy your Vagrant VM and cleanup interim files
```
(venv) $ vagrant destroy -f
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
(venv) $ deactivate
$ rm -rf .vagrant venv *-console.log
```