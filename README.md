# Attack Infrastructure

v1.0 - 'Zip, ship, deploy'

This repo bundles together some resources and files for an easy and quick container-based attack infrastructure deployment.
The screen is built with [tmuxify](https://github.com/tonchis/tmuxify).

It's basically a less robust, less fleshed out, but more lightweight version of [warhorse](https://github.com/warhorse/warhorse) containing only open-source tools because I don't like proprietary software (and not because I can't afford the proprietary ones ;-; ). This deployment is only really suited for single vps deployments.

If you are searching for something *more fleshed out* you might want to give [warhorse](https://github.com/warhorse/warhorse) or [Redcloud](https://github.com/khast3x/Redcloud) a try!

## Content

Contained within this setup are the following programs and containers:

### C2s

- [Sliver C2](https://github.com/BishopFox/sliver) / [container-sliver](https://github.com/cyb3rn00dl3s/container-sliver)
- [Empire C2](https://github.com/BC-SECURITY/Empire)
- [Metasploit](https://github.com/rapid7/metasploit-framework)

### Web/File Server

- [nginx](https://hub.docker.com/_/nginx/)
- SFTP (included in openssh-server)

### Phishing

- [Gophish](https://github.com/gophish/gophish)
- [evilginx2](https://github.com/hash3liZer/evilginx2/) / <https://hub.docker.com/r/heywoodlh/evilginx2>

## Warning

This "thing" is an OPSEC Nightmare (for now) and should not be used for any serious production environment. The containers are configured in an unsafe manner and everything is running as root. I am not responsible for any damages or OPSEC failures that occur when running this project.

## OPSEC

With Version 1.0 I introduced some (slight) OPSEC improvements. While the configuration does help, I am nowhere near to say that this project can be considered "safe".

### System Hardening

The current system hardening process is experimental and **will kill your pet**, be it cat, dog or iguana. It **may** or may not **be directly** or indirectly **responsible** for the complete and utter **destruction** of **your system, your data** and a randomly chosen town in Ohio, USA.

**Please read up on what you are doing before enabling any hardening measures, kthx**

The following hardening measures are currently available for configuration:

- User creation
  - Create a super user (super_user_name/super_user_pass)
  - Generate and download an ssh key
  - Add generated ssh key to authorized_keys
- Install doas and replace sudo if necessary
- SSH hardening
  - Disallow root login
  - Enable PublickeyAuthentication
  - Disable PasswordAuthentication
  - Set MaxAuthRetries to 3
- Replace malloc system-wide with [hardened_malloc](https://github.com/GrapheneOS/hardened_malloc/)
- Replace some system information to avoid identification
  - Set timezone to UTC
  - Set Machine ID to the generic Whonix ID
- Filesystem adjustments
  - Set a default umask of 0077
  - Disable Core dumps
  - Set Swappiness to minimum amount
  - Enable APT seccomp filter

Finally all hardening measures can be verified by looking at the ansible play files located in `./roles/$os/optional_hardening/`

## Requirements

On your local PC:

- Python 3
- sshpass
- [Ansible](https://www.ansible.com/)

On your VPS:

- Debian 11

## Repository Directory structure

```s
attack-infrastructure/
├── roles/
│   ├── debian/
│   │   ├── docker/
│   │   │   ├── images/ - Contains task files for docker images
│   │   │   └── main.yml - Installs docker prerequisites
│   │   ├── packages/ - Contains task files for apt requirements
│   │   ├── flair/ - Contains task files for misc customization
│   │   ├── optional_hardening/ - Contains task files for system hardening
│   │   └── full_install.yml - Debian installation playbook
│   └── user-setup.yml - Creates necessary directory structure
├── .tmuxify.layout - Example tmuxify layout
├── ansible_vars.yml - Ansible variables (target, username, password)
├── ansible.cfg - Config file
├── inventory.yml - Inventory file
└── README.MD
```

## Usage

0. Read the readme!
1. Clone the repository
   ```sh
   git clone https://github.com/cyb3rn00dl3s/attack-infrastructure
   ```
2. Change into the repository directory
   ```sh
   cd attack-infrastructure
   ```
3. Install the requirements (e.g. on Debian):
   ```sh
   sudo apt update && sudo apt install -y python3 python3-pip python3-venv sshpass
   ```
4. Setup a python venv:
    ```sh
    python3 -m venv venv && source venv/bin/activate
    ```
5. Install the python requirements:
   ```sh
   pip install -r requirements.txt
   ```
6. Fill out the variables in the ansible_vars.yml file
   1. Choose the features you want to install
   2. Tweak the system settings to your liking
   3. (Recommended) setup system hardening
7. (optional) edit the tmuxify layout in `.tmuxify.layout`
8. Run the full_install Ansible Playbook from the root of the repository:
    ```sh
    ansible-playbook roles/debian/full_install.yml
    ```
9.  Log on to your VPS, cd into /root/c2 (or /home/$superuser/c2) and run `tmuxify`
    1.  If you have enabled system hardening, you will need the new key file (id_$superuser_\$timestamp) to log onto the system.

**Warning** installing every tool that is included here will take up a LOT of disk space (~5-ish GB). Make sure that you have enough or modify the setup to suit your needs (recommended!).

## On-Server Directory Structure

```s
/root/ or /home/$superuser
└── c2/
    ├── configs/ - various configuration files
    ├── content/ - Files hosted via nginx
    ├── sliver/ - Dedicated sliver folder
    │   ├── configs/
    │   └── misc/
    ├── empire/ - Dedicated empire folder
    ├── msf4/ - Dedicated msf folder
    ├── payloads/ - general directory for payloads
    └── phishlets/ - Folder for phishlets (can be used for sliver, evilginx, etc)
```

## TODOs

- Figure out independent exfiltration / data transfer
- General OPSEC considerations (e.g. Ansible Vault, system hardening, etc.)
- Support other VPS OSses (Alpine, maybe CentOS)
- User-friendly setup/run script
- Integrate Packer/Crypter(PEzor+Veil) and Mailserver
- Moar hardenings
  - Set up kernel hardening
  - Implement application sandboxing (Firejail or bubblewrap?)
  - fail2ban for SSH
  - Web-Server hardening through Webbunker
  - Firewall Rules Presets
    - Only allow management interfaces on localhost
- Easier management with Portainer
