# Proxmox CT Development Setup

## Overview
This README provides a step-by-step guide to set up a development environment inside a Proxmox container (CT). The setup includes enabling nesting, configuring SSH access, running a bash script to install necessary packages, and ensuring Ansible is installed using pipx.

## Steps

### 1. Create the Container in Proxmox
**password\sshkey**
    - create ssh or password to connect to machine 
**Enable Nesting**:
    - Under the "Options" tab, set "Nesting" to "Yes".

### 2. Run Initial Setup Script
1. **Connect to the Container**:
    - Using SSH: Open your terminal and run:
      ```sh
      ssh root@<CT_IP_Address>
      ```
    - Using Proxmox Console: Login using the password set earlier.

2. **Execute the initial setup script**: from https://github.com/aviellg/ansible-pull-setup
```bash 
#!/bin/bash

# Function to run apt commands with sudo
run_with_sudo_if_needed() {
  local command="$@"

  if [[ "$command" == apt* ]]; then
    if [ "$EUID" -ne 0 ]; then
      echo "Running with sudo: $command"
      sudo bash -c "$command"
    else
      echo "Running without sudo: $command"
      bash -c "$command"
    fi
  else
    echo "Running without sudo: $command"
    bash -c "$command"
  fi
}

# Remove Ansible if installed via apt
echo "Removing Ansible installed via apt..."
run_with_sudo_if_needed "apt remove -y ansible"

# Install pipx, git, and curl using apt
echo "Installing pipx, git, and curl..."
run_with_sudo_if_needed "apt update"
run_with_sudo_if_needed "apt install -y pipx git curl"

# Ensure pipx is in the PATH
echo "Ensuring pipx is in the PATH..."
pipx ensurepath

# Install Ansible using pipx
echo "Installing Ansible with pipx..."
pipx install --include-deps ansible

# Inject argcomplete into Ansible
echo "Injecting argcomplete into Ansible..."
pipx inject --include-apps ansible argcomplete

# Run ansible-pull command without sudo
echo "Running ansible-pull command without sudo..."
ansible-pull -U https://github.com/aviellg/ansible-pull-setup.git

# Reload shell
echo "Reloading the shell..."
exec $SHELL

echo "Installation completed!"

```

   This script will install necessary packages and configure the `autoadmin` user.

### 4. Verify Setup
- Ensure necessary packages are installed.
- Verify the `autoadmin` user is set up correctly.
- Confirm Ansible has been installed using pipx.

## Notes
- Ensure the network settings are properly configured in Proxmox to allow the container to connect to the internet.
- Adjust resources as per the development environment requirements.

## Troubleshooting
- If the container does not start, check the resources and nesting settings.
- Ensure the SSH key is correctly placed in the container.

## Contact
For any issues or suggestions, please reach out to the repository owner or refer to the Proxmox documentation.
