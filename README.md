# Site Reliability Engineering Home Lab Setup Log

## Project Goal
Practice/deploy tools that will help me in my SRE career. Objectives might vary based on what my time allows me to do.

## Hardware used
Dell Optiplex 7050 Micro (Intel Core i5-6500T @ 2.5 GHz, 8GB RAM, 256GB SSD)

# Day One:
## Initial Infrastructure & Prometheus Deployment

### Actions Taken & Progress

#### 1. Foundational Infrastructure Setup (Proxmox Host as Ansible Control Node)
* **Proxmox Host Preparation:** Designated a virtual machine (or the Proxmox host itself) to serve as the central Ansible control node for managing the lab environment.
* **Ansible Installation & Configuration:** Installed Ansible on the designated control node. Configured basic Ansible settings, including the inventory file (`/etc/ansible/hosts`) to define the managed virtual machines (`vm101`, `vm102`, `vm103`).
* **Secure Connectivity:** Set up SSH key-based authentication from the Ansible control node to all target virtual machines to enable secure, passwordless automation.
* **Connectivity Test:** Successfully verified Ansible's ability to communicate with and execute commands on all target VMs.

#### 2. Prometheus Server Deployment Attempt (Initial Ansible Playbook)
* **Playbook Development:** Began the automation of Prometheus server installation by creating an `ansible-playbook` (`install_prometheus.yml`). This playbook was designed to handle user/group creation, directory setup, binary download/extraction, configuration management, and systemd service integration.
* **Persistent Troubleshooting:** Encountered significant and persistent `YAML syntax errors` (specifically "mapping values are not allowed in this context" at consistent line/column positions) during playbook execution. Exhausted standard YAML debugging techniques, including strict indentation checks, hidden character removal (`cat -A`, `vim :set list`), and clean file recreation. This indicated a potential deeper environmental issue with the YAML parser on the host.

#### 3. Prometheus Server Deployment (Successful Shell Script Workaround)
* **Strategic Pivot:** Due to the unresolved Ansible YAML parsing issue, a decision was made to bypass the Ansible layer for the initial Prometheus deployment. A comprehensive shell script (`install_prometheus_script.sh`) was developed to replicate all the intended installation steps.
* **Script Debugging:** The initial execution of the shell script revealed specific runtime errors:
    * Warnings about the `prometheus` group already existing (from previous partial Ansible runs).
    * **Critical `chown: invalid user: 'prometheus:prometheus'` errors**, indicating the `prometheus` user did not exist on the system despite the group being present.
    * Further debugging resolved a `useradd` command ambiguity (`--g` vs. `-g`) related to primary group assignment.
* **Successful Deployment:** After these corrections, the `install_prometheus_script.sh` executed flawlessly.
* **Verification:** Confirmed Prometheus service status (`active (running)`), verified it was listening on `0.0.0.0:9090`, and successfully accessed the Prometheus Web UI in a browser.

#### 4. Initiating Node Exporter Setup
* **Concept Introduction:** Began exploring the next phase of monitoring by understanding Node Exporter – a lightweight agent essential for collecting system-level metrics from individual hosts.
* **Deployment Preparation:** Outlined the steps for installing Node Exporter on each of the virtual machines (`vm101`, `vm102`, `vm103`) using a dedicated shell script, and preparing the Prometheus configuration to scrape these new targets.

### Key Learnings & Challenges
* **YAML Strictness:** Reconfirmed the absolute precision required for YAML indentation and syntax; even subtle, invisible characters can cause parsing failures.
* **Debugging Layers:** Successfully navigated debugging across multiple layers: Ansible playbook syntax, shell script execution, and underlying Linux system commands (`useradd`, `chown`).
* **Problem-Solving Adaptability:** Demonstrated the ability to pivot strategies (from Ansible to shell script) when a primary tool faces inexplicable blockers, a crucial SRE skill.

### Next Steps
* Install Node Exporter on `vm101`, `vm102`, and `vm103`.
* Configure Prometheus to scrape metrics from all Node Exporter instances.
* Begin exploring basic PromQL queries and dashboarding in Prometheus UI.
