## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Diagram](https://github.com/gabg1/cyberstudies/blob/1b954f0d346a2220ff6da098eeeadb92bff322b4/Diagrams/Diagramend.png?raw=true)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. 
Alternatively, select portions of the deployment file may be used to install only certain pieces of it, such as Filebeat.

Deployment file: 

  - [install-elk.yml](https://github.com/gabg1/cyberstudies/blob/43d3b2c0d88cf29713b89d2e328ddd7452f79337/Ansible/install-elk.yml)

```
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: azdmin
  become: true
  tasks:

     # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

        # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: 262144
        state: present
        reload: yes

# Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module (It will default to pip3)
    - name: Install Docker module
      pip:
        name: docker
        state: present       

# Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          -  "5601:5601"
          -  "9200:9200"
          -  "5044:5044"

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```        

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file system and system metrics.
Filebeat watches for log information about the file system, including which files have changed and when.
Metricbeat collect metrics from the operating system and from services running on the server.

The configuration details of each machine may be found below.

| Name     | Function    | IP Address  | Operating System        |
|----------|:------------|------------:|------------------------:|
| Jump Box | Gateway     | 10.0.0.4    | Linux Ubuntu 20.04.3 LTS|
| Web-1    | Web Server  | 10.0.0.7    | Linux Ubuntu 20.04.3 LTS|
| Web-2    | Web Server  | 10.0.0.6    | Linux Ubuntu 20.04.3 LTS|
| ELK VM   | Monitoring  | 10.1.0.5    | Linux Ubuntu 20.04.3 LTS|

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 98.36.200.158

Machines within the network can only be accessed by the Jump Box with IP 10.0.0.4.


A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses   |
|----------|---------------------|------------------------|
| Jump Box | Yes                 | 98.36.200.158          |
| Web-1    | No                  | 10.0.0.4               |
| Web-2    | No                  | 10.0.0.4               |
| ELK VM   | No                  | 10.0.0.4               |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because reduces the risk of human error, standardizes the configuration making it easy to replicate and fast to re-deploy. 

The playbook implements the following tasks:
- Increase virtual memory: Configures the target VM (the machine being configured) to use more memory
- Install docker.io: The Docker engine, used for running containers
- Install python3-pip:  Package used to install Python software
- Install Docker module: Python client for Docker. Required by Ansbile to control the state of Docker containers
- Download and launch a docker elk container
- Enable service docker on boot

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![docker ps](https://github.com/gabg1/cyberstudies/blob/43d3b2c0d88cf29713b89d2e328ddd7452f79337/Diagrams/Day%201%20Screenshot/docker_ps.png?raw=true)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
IP 10.0.0.7(Web-1) IP 10.0.0.6(Web-2)

We have installed the following Beats on these machines:
Filebeat, Metricbeat

These Beats allow us to collect the following information from each machine:
- Filebeat: Monitor the Apache server and MySQL database logs generated by DVWA
- Metricbeat: Monitor metrics from the DVWA machines.  



### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the [install-elk.yml](https://github.com/gabg1/cyberstudies/blob/43d3b2c0d88cf29713b89d2e328ddd7452f79337/Ansible/install-elk.yml) file to your machine and move it to `/etc/ansible` location.
```
$ cd /etc/ansible/ 
$ curl https://github.com/gabg1/cyberstudies/blob/e29f13df5a00ead1f4cbe55d6289f2084d5dac1e/Ansible/install-elk.yml > /etc/ansible/install-elk.yml
```

- On `/etc/ansible/` edit your `hosts` file to include the Web Servers VMs[webservers] and the VM where you ELK stack will run[elk]. 

Example:

$ nano /etc/ansible/hosts

```
[webservers]
10.0.0.7 ansible_python_interpreter=/usr/bin/python3
10.0.0.6 ansible_python_interpreter=/usr/bin/python3

[elk]
10.1.0.5 ansible_python_interpreter=/usr/bin/python3
```
 
- Run the playbook:
```
$ ansible-playbook install-elk.yml 
```
- After running the playbook, navigate to  `http://[your.ELK-VM.External.IP]:5601/app/kibana` to confirm that installation worked as expected. If successful, you should see a page similiar to the one shown below: 
![Kibana](https://github.com/gabg1/cyberstudies/blob/e29f13df5a00ead1f4cbe55d6289f2084d5dac1e/Diagrams/Day%201%20Screenshot/kibana_dash.png?raw=true)

Additional playbooks can be found here:
- Playbook to install Filebeat, download: [Filebeat](https://github.com/gabg1/cyberstudies/blob/e29f13df5a00ead1f4cbe55d6289f2084d5dac1e/Ansible/filebeat-playbook.yml)

- Playbook to install Metricbeat, download: [Metricbeat](https://github.com/gabg1/cyberstudies/blob/e29f13df5a00ead1f4cbe55d6289f2084d5dac1e/Ansible/metricbeat-playbook.yml)

