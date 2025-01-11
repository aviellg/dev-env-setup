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

# Function to check if a command needs to be run with sudo
run_with_sudo_if_needed() {
  local command="$@"
  
  if [ "$EUID" -ne 0 ]; then
    echo "Running with sudo: $command"
    sudo bash -c "$command"
  else
    echo "Running without sudo: $command"
    bash -c "$command"
  fi
}

# Function to perform the tasks
perform_tasks() {
  run_with_sudo_if_needed "apt-get update"
  run_with_sudo_if_needed "apt-get upgrade -y"
  run_with_sudo_if_needed "apt-get install -y ansible git curl"
  run_with_sudo_if_needed "ansible-pull -U https://github.com/aviellg/ansible-pull-setup.git"
}

perform_tasks

exit 0
```

   This script will install necessary packages and configure the `autoadmin` user.

### 3. Install Ansible using pipx
1. **Save the following bash script to a file (e.g., `install_ansible_with_pipx.sh`)**:
    ```bash
    #!/bin/bash

    # Remove Ansible if installed via apt
    echo "Removing Ansible installed via apt..."
    sudo apt remove -y ansible

    # Install pipx using apt
    echo "Installing pipx..."
    sudo apt update
    sudo apt install -y pipx

    # Ensure pipx is in the PATH
    echo "Ensuring pipx is in the PATH..."
    pipx ensurepath

    # Install ansible using pipx
    echo "Installing Ansible with pipx..."
    pipx install --include-deps ansible

    # Inject argcomplete into ansible
    echo "Injecting argcomplete into Ansible..."
    pipx inject --include-apps ansible argcomplete

    # Reload shell
    echo "Reloading the shell..."
    exec $SHELL

    echo "Installation completed!"
    ```
2. **Make the script executable**:
    ```sh
    chmod +x install_ansible_with_pipx.sh
    ```

3. **Run the script**:
    ```sh
    ./install_ansible_with_pipx.sh
    ```

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
