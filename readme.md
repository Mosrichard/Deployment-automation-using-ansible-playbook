Website Deployment on NGINX using Ansible
Project Overview

This project demonstrates how to automate the deployment of a static web application on AWS EC2 instances using Ansible playbooks with NGINX.
The deployment process includes:

Installing NGINX and unzip package.

Removing old/default website files.

Downloading a website zip file from a URL.

Unzipping the files and copying them to /var/www/html/.

Restarting NGINX to serve the site.

The playbook ensures that the website can be deployed reliably and repeatedly with a single command.


Prerequisites

AWS Account – for creating EC2 instances.

Ansible Installed on your control node.

SSH access from the control node to target EC2 instances.

Public URL for website zip (e.g., organic-1.0.0.zip).


Architecture
       Control Node (Ansible) ---> Target Node(s) (EC2 Instances with NGINX)


Control Node: Runs Ansible playbook commands.

Target Node(s): EC2 instances where the static website will be deployed.


Steps
1. Launch EC2 Instance(s)

Go to AWS EC2 console and launch Ubuntu instance.

Assign security group with inbound rules:

SSH (port 22) from your control node.

HTTP (port 80) for NGINX.

Note the public IP of the instance(s).


2. Connect Control Node to Target EC2 using .ssh

Place your .pem file on the control node, e.g., ~/.ssh/mykey.pem.   if dont have just create mkdir -p ~/.ssh/

Set permissions for the key:

chmod 400 ~/.ssh/mykey.pem      

Add target node(s) to your inventory.ini:

[webservers]
ec2-1 ansible_host=<EC2_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=/path/to/mykey.pem


4. Playbook: playbook.yml
- name: Deploy static website with Ansible
  hosts: webservers
  become: yes
  tasks:

    - name: Install NGINX and unzip
      apt:
        name:
          - nginx
          - unzip
        state: present
        update_cache: yes

    - name: Remove default html files on Nginx
      file:
        path: /var/www/html/
        state: absent

    - name: Recreate deployment directory
      file:
        path: /var/www/html/
        state: directory
        owner: www-data
        group: www-data
        mode: '0755'

    - name: Download website zip
      get_url:
        url: https://example.com/organic-1.0.0.zip   # replace with your link
        dest: /your_controlnode_path/organic-1.0.0.zip
        mode: '0644'

    - name: Unzip website files
      unarchive:
        src: /your_controlnode_path/organic-1.0.0.zip
        dest: /your_controlnode_path/
        remote_src: yes
        extra_opts: ["-o"]

    - name: Copy website files to nginx html folder
      copy:
        src: /your_controlnode_path/organic-1.0.0/
        dest: /var/www/html/
        owner: www-data
        group: www-data
        mode: '0755'

    - name: Restart nginx
      service:
        name: nginx
        state: restarted


5. Run the Playbook
ansible-playbook -i inventory.ini playbook.yml


The playbook will:

1. Install NGINX and unzip if not present.

2. Remove old files in /var/www/html/.

3. Download and unzip the website.

4. Copy files to the web server folder.

5. Restart NGINX.
--------------------------------------------------

6. Verify Deployment

Open the EC2 public IP in a browser:

http://<EC2_PUBLIC_IP>/


You should see your static website live.


Notes

Ensure NGINX default HTML files are removed before copying your website.

unarchive requires unzip to be installed on target nodes.

You can deploy to multiple EC2 instances simultaneously by adding them in inventory.ini.