+++
title = "How to Run a Playbook Without Entering SSH Password Each Time"
description = ""
categories = ["DevOps"]
keywords = ["azizunsal", "azizunsal.github.io","ansible","server provisioning", "devops"]
+++

One of the boring things while working with Ansible is that it prompts each time the hosts’ SSH passwords. 

```
$ ansible-playbook my-playbook.yml
SSH password:
```

I kept entering passwords a very long time when creating/modifying playbooks.

Finally, I decided not to enter ssh passwords each time. Its very simple configuration indeed.

Here it is;
I removed `ask_pass=True` line in `ansible.cfg` file.
I added a new group variables section for the whole inventory, it can be added for a specific group though.
```
[all:vars]
ansible_user=<username>
ansible_ssh_pass=<ssh_pass>
```

Now it is time to run the playbook;
``` 
$ ansible-playbook my-playbook.yml
```

And the result is disappointing!

```
TASK [common : Installing python2 for Ansible]
fatal: [192.168.1.98]: FAILED! => {"msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}
fatal: [192.168.1.100]: FAILED! => {"msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}
fatal: [192.168.1.101]: FAILED! => {"msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}
```


Ansible says it needs `sshpass` to be able to run ssh commands without prompting passwords.

Ok, let's do the last step by installing `sshpass`;

On Ubuntu 
```
apt-get install sshpass
```
or if you’re running Ansible on OSX 
```
brew install https://raw.githubusercontent.com/kadwanev/bigboybrew/master/Library/Formula/sshpass.rb
```

Detailed installation instruction for `sshpass` can be found [here](https://gist.github.com/arunoda/7790979).


Let's run the playbook for the last time:

```
$ ansible-playbook my-playbook.yml
```

```

PLAY [all:!proxmox] 

TASK [common : include_tasks]
included: check-python2.yml for 192.168.1.98, 192.168.1.100, 192.168.1.101

TASK [common : Installing python2 for Ansible]
changed: [192.168.1.98]
changed: [192.168.1.100]
changed: [192.168.1.101]


TASK [common : Reload ansible_facts]
ok: [192.168.1.100]
ok: [192.168.1.101]
ok: [192.168.1.98]
```

Done.