# Configuration Management with Ansible

An Ansible project for configuring a Ubuntu-based Linux server, installing Nginx, and serving a static website.

## Table of Contents

- [About](#about)
- [Features](#features)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Installation & Usage](#installation--usage)
- [Example Output](#example-output)
- [How It Works](#how-it-works)
- [Error Handling](#error-handling)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Author](#author)
- [License](#license)

## About

This repository contains a reusable Ansible playbook for provisioning a Linux server. The playbook targets the `droplet_do` host group and applies a sequence of roles to prepare the server, configure Nginx, deploy website files, and configure SSH access.

**Project Reference:** [roadmap.sh/projects/configuration-management](https://roadmap.sh/projects/configuration-management)

## Features

- Updates the APT package cache and upgrades installed packages.
- Installs common server utilities, including Fail2ban, CURL, SSH, and archive tools.
- Installs and enables Nginx as a system service.
- Configures Nginx with a Jinja2 template and serves files from `/var/www/roadmap/`.
- Copies and extracts the static website archive on the remote server.
- Adds a public SSH key to the root user's `authorized_keys` file.
- Reloads Nginx automatically when its configuration changes.

## Project Structure

```text
.
├── inventory.ini                  # Target hosts and connection settings
├── setup.yaml                     # Main Ansible playbook
├── roles/
│   ├── app/
│   │   ├── files/website.tar.gz   # Static website assets
│   │   └── tasks/main.yaml        # Uploads and extracts the website archive
│   ├── base/tasks/main.yaml       # Updates the system and installs utilities
│   ├── nginx/
│   │   ├── defaults/main.yaml     # Nginx variables
│   │   ├── handlers/main.yaml     # Reloads Nginx after configuration changes
│   │   ├── tasks/main.yaml        # Installs and configures Nginx
│   │   └── templates/nginx.conf.j2
│   └── ssh/tasks/main.yaml        # Installs the configured public key
└── README.md
```

## Requirements

- Ansible Core installed on the control machine.
- A reachable Debian-based Linux server with SSH access.
- Python available on the managed server.
- The `ansible.posix` collection for the `authorized_key` module:

  ```bash
  ansible-galaxy collection install ansible.posix
  ```

- A private SSH key on the control machine and its matching public key.
- A `website.tar.gz` archive in `roles/app/files/`, unless the application role is changed to use another source.

## Installation & Usage

1. Clone the repository and enter the project directory:

```bash
git clone https://github.com/JescAude18/configuration-management.git
cd configuration-management
```

2. Update `inventory.ini` with the managed server's address, SSH user, and private key path. Do not commit private keys or other credentials.

3. Ensure the website archive exists at `roles/app/files/website.tar.gz`. The archive should contain the static files that Nginx will serve.

4. Check connectivity:

```bash
ansible all -i inventory.ini -m ansible.builtin.ping
```

5. Run the playbook:

```bash
ansible-playbook setup.yaml -i inventory.ini
```

The playbook uses `become: true`, so the connecting user must be allowed to use privilege escalation.

6. Open the server address in a browser or verify the response from the command line:

```bash
curl http://SERVER_ADDRESS
```

## Example Output

An abbreviated successful run looks like this:

```text
PLAY [base] ****************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [droplet_do]

TASK [base : Update cache] *************************************************************************************
changed: [droplet_do]

TASK [base : Update server] ****************************************************************************************************************
changed: [droplet_do]

TASK [base : Install utilities] ****************************************************************************************************************
ok: [droplet_do]

TASK [base : Remove dependencies that are no longer required and purge their configuration files] ***************************************************************************************************************
ok: [droplet_do]

TASK [base : Run "apt-get clean"] ***************************************************************************************************************
ok: [droplet_do]

TASK [nginx : Install nginx] ***************************************************************************************************************
ok: [droplet_do]

TASK [nginx : Create root directory] ***************************************************************************************************************
ok: [droplet_do]

TASK [nginx : Copy nginx.conf template to server] ***************************************************************************************************************
ok: [droplet_do]

TASK [nginx : Enable nginx.conf by symlink] ***************************************************************************************************************
ok: [droplet_do]

TASK [nginx : Disable default nginx configuration] ***************************************************************************************************************
changed: [droplet_do]

TASK [nginx : Handle nginx service] ***************************************************************************************************************
ok: [droplet_do]

TASK [app : Upload static HTML website to server] ***************************************************************************************************************
ok: [droplet_do]

TASK [app : Unzip the tarball] ***************************************************************************************************************
ok: [droplet_do]

TASK [ssh : Add a given public key to the server] ***************************************************************************************************************
ok: [droplet_do]

PLAY RECAP ****************************************************************************************************
droplet_do                 : ok=15   changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

The exact task counts depend on the server's initial state.

## How It Works

`setup.yaml` targets `droplet_do` and applies the roles in this order:

1. **base** updates the operating system, installs required utilities, removes unused packages, and cleans the APT cache.
2. **nginx** installs Nginx, creates the web root, renders the virtual host configuration, disables the default site, and starts Nginx at boot.
3. **app** uploads `website.tar.gz` to `/var/www/roadmap/` and extracts it there.
4. **ssh** adds the configured public key to the root account's authorized keys.

The default Nginx variables listen on port `80`, use `/var/www/roadmap/` as the document root, and serve `index.html`. These values can be overridden in inventory, group variables, or host variables.

## Error Handling

- **SSH connection failure:** verify the host address, SSH user, private key path, firewall rules, and key permissions.
- **Privilege escalation failure:** make sure the remote user has passwordless or otherwise available `sudo` access when using `become: true`.
- **Missing `website.tar.gz`:** place the archive in `roles/app/files/` before running the playbook.
- **APT or package errors:** confirm that the server has network access and that its package sources are available.
- **Nginx configuration errors:** validate the rendered configuration on the server with `sudo nginx -t`, then inspect `systemctl status nginx`.
- **SSH key task errors:** verify that the `ansible.posix` collection is installed and that the referenced public key file exists.

## Roadmap

- Replace hard-coded connection and key paths with variables or encrypted Ansible Vault values.
- Add automated Nginx configuration validation before reloading the service.
- Add a handler to restart or reload services only when required.
- Add Molecule or CI tests for role idempotence.
- Add configurable firewall rules and HTTPS support with TLS certificates.

## Contributing

1. Create a feature branch.
2. Make focused changes that follow the existing role structure.
3. Run the playbook against a disposable test server or validate it with a local CI setup.
4. Open a pull request describing the change and its verification steps.

## Author

**Created by**: Jessica MOUSSOUGAN

**Email**: [jessicamoussougan@gmail.com](mailto:jessicamoussougan@gmail.com)

**GitHub**: [@JescAude18](https://github.com/JescAude18)

## License

No license yet.

This project is currently for personal training and learning.
