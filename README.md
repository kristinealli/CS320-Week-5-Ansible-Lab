# CS320 Week 5 Ansible Web Server Lab

An Ansible lab that provisions an AlmaLinux virtual machine as a basic Apache web server and deploys a small static website. The Ansible control node is WSL running on Windows.

## Overview

The `site.yml` playbook automates the following tasks on the managed AlmaLinux host:

- Adds the Google Chrome repository and installs the lab's required packages.
- Installs Apache HTTP Server, MariaDB Server, Firefox, and Git.
- Copies the static website into Apache's document root at `/var/www/html`.
- Removes Google Chrome after installation.
- Reboots the managed host and waits for it to return.
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

Before running the playbook, confirm that you have:

- WSL with an Ansible control environment.
- Ansible installed and available on your `PATH`.
- Network or SSH access to the AlmaLinux managed host.
- A working SSH private key with permission to connect to the managed host.
- `sudo` privileges on the managed host.

Check the Ansible installation with:

```bash
ansible --version
```

## Configuration

Update `inventory.ini` for your environment. The inventory identifies the managed host, its IP address, SSH user, and private key:

```ini
[linux_vdi]
tp_cs320_al10 ansible_host=10.2.37.105 ansible_user=kjohnson ansible_connection=ssh ansible_ssh_private_key_file=/home/kristine_johnson/.ssh/id_ed25519
```

Do not commit private keys, passwords, or environment-specific credentials. Replace the sample host values when using this project on another machine.

Test inventory discovery and SSH connectivity before running the playbook:

```bash
ansible all -i inventory.ini --list-hosts
ansible all -i inventory.ini -m ansible.builtin.ping
```

The first command should list the configured managed host rather than `localhost`.

## Run the Playbook

From the project directory, run:

```bash
ansible-playbook -i inventory.ini site.yml
```

The playbook uses `become: true`, so Ansible may prompt for the managed user's `sudo` password. If the SSH key or inventory file is stored elsewhere, use its path explicitly.

## Run the Deployment Verification

After the deployment playbook completes, run the verification playbook:

```bash
ansible-playbook -i inventory.ini verify.yml
```

This checks the required packages, confirms Google Chrome was removed, verifies all three website files, checks the reboot and server reachability, confirms Apache is enabled and running, tests each page over HTTP, and validates the clean playbook recap. It writes the concise results to `verification-results.txt` without duplicating the full Ansible readout.

## Verify the Deployment

After the playbook completes, open the managed host's address in a browser:

```text
http://<managed-host-ip>
```

Verify Apache is enabled and active:

```bash
ansible -i inventory.ini linux_vdi -m command -a "systemctl is-enabled httpd"
```
Expected: Enabled

Verify Apache is enabled and active:

```bash
ansible -i inventory.ini linux_vdi -b -m command -a "systemctl is-active httpd"
```

```bash
ansible -i inventory.ini -m ansible.builtin.shell -a "systemctl is-active httpd" --become
curl http://<managed-host-ip>
```

A successful run should finish with `faild=0`, and Apache should report `active`.

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