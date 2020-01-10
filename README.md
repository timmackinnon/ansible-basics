# Ansible Primer
This repository holds some details around an introduction to __Ansible__ + some code examples. The bulk of this repository is based on the book `Ansible Up & Running` whose code examples live [here](https://github.com/ansiblebook/ansiblebook).

# Setting up your environment
For most of the examples here, you will need the following:
* Python
* VirtualEnv
* VirtualBox
* Vagrant

I don't go through the details of installing these tools but rather leave these up to the reader for their operating system of choice.

# What is Ansible?
The name __Ansible__ comes from the book _Ender's Game_ by _Orson Scott Card_. In that book, the _ansible_ was used to control a large number of remote ships at once, over vast distances. You can think of this as a metaphor for controlling remote servers.

Ansible is often described as a _configuration management_ tool, and is typically mentioned in the same conversation as _Chef_, _Puppet_, and _Salt_. Configuration Management typically entails describing a set of servers and their state and then using a tool to enforce that the servers are indeed in that state. Ansible exposes a _Domain_ _Specific_ _Language_ (DSL) that is used to describe the state of servers.

Ansible can be used for doing _deployment_ as well. Using a single tool for _management_ and _deployment_ can make life simpler for the individuals responsible for operations.

Ansible can also be used for _orchestration_ of deployment. This typically involves multiple servers where things have to happen in a specific order.

Finally Ansible can be used for _spinning_ _up_ new servers in the Cloud ... in environments like _Amazon's_ _EC2_ and _OpenStack_.

# How does Ansible work?
Typically users write _scripts_ to provision servers. In Ansible, a script is called a _playbook_. A _playbook_ decribes which _hosts_ to configure, and an ordered list of _tasks_ to perform on these hosts.

Ansible will make _ssh_ connections in parallel to a list of _hosts_ and execute the list of _tasks_ on these hosts simultaneously.

Ansible will:
* generate a Python script to perform the _task_ in question
* copy the script to all _hosts_
* execute the script on all _hosts_
* wait for the script to complete execution on all _hosts_

Ansible will then move to the next _task_ in the list of _tasks_ and repeat. It is important to note that:
* Ansible runs each task in parallel across all _hosts_
* Ansible waits until all hosts have completed a _task_ before moving to the next task
* Ansible runs the _tasks_ in the order that you specify them

# Why Ansible?
Ansible's syntax is built on top of `YAML`, which is a data format language that was designed to be easy for humans to read and write. In a way, `YAML` is to `JSON` what `Markdown` is to `HTML`.

To manage a server with Ansible, the server needs only Python and SSH.

Ansible is _agentless_ and works on a _push_ model (although it does support a _pull_ mode as well)

Ansible _scales_ _down_. Using Ansible to configure a single node is easy.

Ansible ships with a large array of pre-existing _modules_. You can use these _modules_ for:
* installing a package
* restarting a service
* copying files
* etc.

Ansible modules use a _declarative_ syntax and are _idempotent_.

Ansible uses thin layers of abstraction which allow users to get going with Ansible very quickly. To work with Ansible, you do need to know the basics of Linux system management. Items like:
* connecting to a host over SSH
* bash and the shell
* installing packages
* use of _sudo_
* working with files and permissions
* service management
* environment variables

As mentioned above, Ansible uses a __YAML__ file format but also uses __Jinja2__ for templating.

For more info, see the complete [Ansible Documentation](http://docs.ansible.com/ansible/)