---
title: "How to Run a Playbook Without SSH Password"
date: 2018-10-05
lastmod: 2018-12-09T20:08:20+03:00
categories: ["devops", "tips"]
keywords: ["ansible", "ansible playbook", "server provisioning", "devops", "ssh", "sshpass"]
description: "Play Ansible playbook without ssh password"
tags: ["ansible", "ansible playbook", "devops"]
---

One of the boring things while working with Ansible is that it prompts each time the hosts’ SSH passwords. 

```bash
$ ansible-playbook my-playbook.yml
SSH password:
```

I kept entering passwords a very long time when creating/modifying playbooks.

Finally, I decided not to enter ssh passwords each time. Its very simple configuration indeed.
<!--more-->
Here it is;

Ansible Config
---

I removed `ask_pass=True` line in `ansible.cfg` file.
I added a new group variables section for the whole inventory, it can be added for a specific group though.

```ini
[all:vars]
ansible_user=<username>
ansible_ssh_pass=<ssh_pass>
```

Now it is time to run the playbook;

```bash
ansible-playbook my-playbook.yml
```

And the result is disappointing!

```bash
TASK [common : Installing python2 for Ansible]
fatal: [192.168.1.98]: FAILED! => {"msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}
fatal: [192.168.1.100]: FAILED! => {"msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}
fatal: [192.168.1.101]: FAILED! => {"msg": "to use the 'ssh' connection type with passwords, you must install the sshpass program"}
```

Sshpass
---

Ansible says it needs `sshpass` to be able to run ssh commands without prompting passwords.

Ok, let's do the last step by installing `sshpass`;

On Ubuntu

```bash
apt-get install sshpass
```

or if you’re running Ansible on OSX

```bash
brew install https://raw.githubusercontent.com/kadwanev/bigboybrew/master/Library/Formula/sshpass.rb
```

Detailed installation instruction for `sshpass` can be found [here](https://gist.github.com/arunoda/7790979).

The Result
---

Let's run the playbook for the last time:

```sh
ansible-playbook my-playbook.yml
```

```bash
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
