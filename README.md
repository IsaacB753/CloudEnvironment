# CloudEnvironment
This is my collection of resources used to build a virtual cloud environment in azure 

The Ansible folder contains playbooks and config files for setting up ansible and the ELK server. The filebeat folder contains playbooks and configuration files for setting up filebeat. 

How to configure the ELK stack for a cloud environment: 

Tip: Manage all the VMs through a jumpbox VM and build a container inside of that to store Keys and playbooks. 
1. Build virtual network located in a new region 
2. Add a network peering in azure to allow access across the different resource groups 
3. Build a new ELK virtual machine with at least 4GB of RAM. Same region as virtual network. Use the same keys to access it as the web VMs it will be managing. 
4. Edit the network security group to only allow SSH access from the ansible container inside the jumpbox VM responsible for managing all the servers 
5. Attempt to SSH into the web VMs from the ansible container
6. Edit the /etc/ansible/hosts file to have the private ips of the ELK vm and the WEB vms

#your hosts will be under different names and private IPs depending on preference and what azure assigns as IPs

Ex.
# /etc/ansible/hosts
 [webservers]
 10.0.0.4 ansible_python_interpreter=/usr/bin/python3
 10.0.0.5 ansible_python_interpreter=/usr/bin/python3
 10.0.0.6 ansible_python_interpreter=/usr/bin/python3

 [elk]
 10.1.0.4 ansible_python_interpreter=/usr/bin/python3

(If you end up getting stuck look for a starter hosts file on google)

7. Write or run a premade provisioner to set up the ELK server 
Ex. 
                               
---
- name: Configure Elk VM with Docker
  hosts: elk ###CHANGE THIS###
  remote_user: Sysadmin ###CHANGE THIS###
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761 ###THIS IMAGE MIGHT NOT ALWAYS BE VALID###	
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

8. Run: docker ps or docker container list -a. To check and see if the container is running 
9. Add your external IP from the ELK VM to this URL to see the data: http://[your.ELK-VM.External.IP]:5601/app/kibana 
