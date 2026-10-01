# CS320 Week 5 Ansible Web Server Lab

An Ansible lab that provisions an AlmaLinux virtual machine as an Apache web server and deploys a small static website. 
WSL running on Windows serves as the Ansible control node.

## Overview

The `site.yml` playbook automates the following tasks on the managed AlmaLinux host:

- Adds the Google Chrome repository and installs the lab's required packages.
- Installs Apache HTTP Server, MariaDB Server, Firefox, and Git.
- Copies the static website into HTTP server document root at `/var/www/html`.
- Removes Google Chrome after installation.
- Reboots the managed host, then await its return.
- Enables and restarts Apache.
- Confirms that Apache is active and that the website returns HTTP status `200`.

The website contains three pages:

- `index.html` - Project home page.
- `about.html` - Lab description.
- `contact.html` - Contact page.

## Technology Stack

- Ansible
- WSL on Windows
- AlmaLinux
- Apache HTTP Server (`httpd`)
- Static HTML
- Bootstrap 5.3.3 via CDN

## Prerequisites

- WSL with an Ansible control environment.
- Ansible installed.
- Network or SSH access to the AlmaLinux managed host.
- Working SSH private key with permission to connect to the managed host.
- `sudo` privileges on the managed host.

Confirm Ansible installation:

```bash
ansible --version
```

## Configuration

Update `inventory.ini` for your specific environment. The inventory identifies the managed host, the managed hosts IP address, SSH user, and private key:

```ini
[linux_vdi]
tp_cs320_al10 ansible_host=10.2.37.105 ansible_user=kjohnson ansible_connection=ssh ansible_ssh_private_key_file=/home/kristine_johnson/.ssh/id_ed25519
```

Do not commit private keys, passwords, or environment-specific credentials. Replace the sample host values when using this project on another machine.

Test to confirm inventory and SSH connectivity prior to running the playbook:

```bash
ansible all -i inventory.ini --list-hosts
ansible all -i inventory.ini -m ansible.builtin.ping
```

The first command above should list the configured managed host (not`localhost`).

## Run the Playbook

From the project directory, run:

```bash
ansible-playbook -i inventory.ini site.yml
```

Since the playbook uses `become: true`,  Ansible may prompt for the managed user's `sudo` password.


## Verify the Deployment

After the playbook completes, open the managed host's address in a browser:

```text
http://<managed-host-ip>
```

Confirm Chrome was succesfully removed:

```bash
ansible -i inventory.ini linux_vdi -b -m command -a "rpm -q google-chrome-stable"
```
Expected: not installed (This is expected and proves that Chrome was installed in the earlier task and removed later)


Verify Apache is enabled and active:

```bash
ansible -i inventory.ini linux_vdi -m command -a "systemctl is-enabled httpd"
```
Expected: Enabled

```bash
ansible -i inventory.ini linux_vdi -b -m command -a "systemctl is-active httpd"
```
Expected: active

## Repository Layout

```text
.
├── files/
│   ├── about.html
│   ├── contact.html
│   └── index.html
├── inventory.ini
├── site.yml
├── successful-playbook-run.txt
├── LICENSE
└── README.md
```

## License

See [LICENSE](LICENSE) for project licensing information.