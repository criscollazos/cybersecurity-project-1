# Instructions - Azure Cloud Environment
## 1) Setting up the Cloud Environment
### Create Resource Group
- Create a resource group and a region, I chose US West 2 region:
  - I named my Resource Group **RedTeam**.
![Image](https://www.dropbox.com/s/6edm3zi55hqthgz/SyMsK791w_B1qxe0QeP.png?dl=1)

### Create Vnet (Virtual Network)
- Create a new Virtual Network.
- Make sure to select the resource group we created previously as well as the same region:
  - Also, use the default IP and subnet settings.
![Image](https://www.dropbox.com/s/fsw2ro55gyw6qve/SyMsK791w_BkmC207xD.png?dl=1)
### Create a Network Security Group (NSG)
- Create a Network Security Group that is part of the Resource Group RedTeam.
- I named my NSG "**MyFirewall**".
- Make sure the NSG is in the same region as everything else we've created thus far. 
- Go to the Inbound Security Rules section in the NSG and click the add button to add a rule.
  - Create one rule blocks all traffic coming in with a high priority number.
![Image](https://www.dropbox.com/s/kjs1osead40gzom/SyMsK791w_r1p3QxExP.png?dl=1)

![Image](https://www.dropbox.com/s/o706fn07z9z6dcd/SyMsK791w_H1zzEeVgD.png?dl=1)
## 2) Create the Virtual Machines
### Create Jump Box VM
- Log in to Azure account.
- Click on the virtual machines box and then click new.
- Under resource group, select the group **RedTeam**.
- I named the VM **Jump-Box-Provisioner**.
- Use a Ubuntu server with at least 1GB of memory.
- Use a public SSH key from your local computer and give it a username you will remember.
  - Use "ssh-keygen" to create a public key if you don't have one.
  - My username is **azadmin**.
![Image](https://www.dropbox.com/s/0a8p1jf5zc1qf9i/SyMsK791w_rykqOAVev.png?dl=1)

### Network Security Group Rules
- This is an overview of all of the inbound rules for the MyFirewall NSG:
 
![Image](https://www.dropbox.com/s/tpmytsy7inhs8ek/SyMsK791w_SyZ_BeSgP.png?dl=1)

### Set up Docker.io on the Jump Box VM
- SSH into your Jump-Box VM, turn on your machine on Azure before that: `ssh azadmin@[public IP]`
- Once logged in, implement the following:
  - `sudo apt install docker.io`
  - `sudo docker pull cyberxsecurity/ansible`
  - Launch the ansible container: `docker run -ti cyberxsecurity/ansible:latest bash` to make sure it works
  - type `exit`
### Config and Hosts File
- `cd` into the `/etc/ansible/` directory and `nano ansible.cfg` file
  - scroll to the remote_user section and update to include **sysadmin** instead of root. Save and exit.
- `nano /etc/ansible/hosts` file
  - Uncomment the [webservers] header
  - Under the header, add the internal IP address of the 3 VMs:
    - `10.0.0.12 ansible_python_interpreter=/usr/bin/python3`
    - `10.0.0.13 ansible_python_interpreter=/usr/bin/python3`
    - `10.0.0.14 ansible_python_interpreter=/usr/bin/python3`

### Create 3 Virtual Machines
- We will create 3 additional virtual machines that will be web servers.
- We will name them **Web-1**, **Web-2**, and **Web-3**.
- Follow this criteria:
  - Allow no public IP address.
  - Create new availability set, I called mine **WEBSET**, you will set the 3 VMs to this.
  - Connect your VMs to the **RedNet** VNet and to the **MyFirewall** NSG.
- Use a public SSH key from the Jump-Box VM docker container and give it a username you will remember.
  - Use "ssh-keygen" to create a public key if you don't have one.
  - My username is **sysadmin**.
![Image](https://www.dropbox.com/s/hz9i8extb66ppgx/SyMsK791w_BySqoyBlw.png?dl=1)
- To make sure it works:
  - SSH into the Jump Box VM
  - Start and attach your docker container: `sudo docker start dvwa && sudo docker attack dvwa`
  - Once in the container, SSH into each VM to make sure they work: 
    - `ssh sysadmin@10.0.0.12`
    - `ssh sysadmin@10.0.0.13`
    - `ssh sysadmin@10.0.0.14`
## 3) Load Balancer
- Create a new Load Balancer in Azure.
- Select static IP address and select same Resource Group and region.
- Select create new public IP address.
- I named my Load Balancer: **Red-Team-LB**
![Image](https://www.dropbox.com/s/jurugzdeu64c255/SyMsK791w_H1W3Abrxw.png?dl=1)
- Once the Load Balancer is created, add a new Health Probe (use the default settings):

![Image](https://www.dropbox.com/s/x1h3cqz9ohgkvoa/SyMsK791w_S1Sg1Grgw.png?dl=1)
- Create a new Backend Pool and add the 3 VMs to it.
![Image](https://www.dropbox.com/s/ao73okv62tdg1ao/SyMsK791w_HJJrkzrxP.png?dl=1)

## 4) Logging into Jump Box Provisioner
- Log in to Azure and turn on your Jump Box Provisioner virtual machine.
- Open your personal computer terminal on (for me, it's the Mac)
```bash
ssh azadmin@[public ip]
```
- The public ip will be different every time. You get the public ip once you turn on your Jump Box VM:
```bash
cristina@Mac-mini ~ % ssh azadmin@40.65.111.84
The authenticity of host '40.65.111.84 (40.65.111.84)' can't be established.
ECDSA key fingerprint is SHA256:BDujdGea+JFQWQQjodS9RCPpiHFh1nLwGnuZXChxmag.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '40.65.111.84' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.3.0-1032-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Jul 10 01:34:11 UTC 2020

  System load:  0.77              Processes:           116
  Usage of /:   7.7% of 28.90GB   Users logged in:     0
  Memory usage: 34%               IP address for eth0: 10.0.0.7
  Swap usage:   0%

 * "If you've been waiting for the perfect Kubernetes dev solution for
   macOS, the wait is over. Learn how to install Microk8s on macOS."

   https://www.techrepublic.com/article/how-to-install-microk8s-on-macos/

11 packages can be updated.
0 updates are security updates.


Last login: Wed Jul  8 03:03:00 2020 from 76.171.145.84
azadmin@Jump-Box-Provisioner:~$
```
## 5) Starting Docker
- Check to see which containers you have:
```bash
azadmin@Jump-Box-Provisioner:~$ sudo docker container list -a
CONTAINER ID        IMAGE                           COMMAND             CREATED             STATUS                    PORTS               NAMES
b1dea65e61f4        cyberxsecurity/ansible:latest   "bash"              46 hours ago        Exited (0) 46 hours ago                       dvwa
```
- For me, my container is: **dvwa**
- To start your container, use the following commands:
```bash
azadmin@Jump-Box-Provisioner:~$ sudo docker start dvwa
serene_bardeen
azadmin@Jump-Box-Provisioner:~$ sudo docker attach dvwa
root@b1dea65e61f4:~#
```
- You can use the command: whoami to make sure you did it right.
## 6) Install ELK Stack
### Create a new VNet (Virtual Network)
- Create a new VNet that is in the same resource group (RedTeam) but in a different region.
- I created one and named it EastNet:
![Image](https://www.dropbox.com/s/9nmfvgco49ilva3/SyMsK791w_SyNS6xhJP.png?dl=1)

- Create a peer connection between the two VNets (RedNet and EastNet)
- In my EastNet page, I clicked on **Peerings** and added a new Peering with the following settings:
  - I created a new connection from EastNet to RedNet and called it: **Elk-to-Red**, connecting to my **RedNet** VNet.
  - I created another connection from RedNet to EastNet and called it: **Red-to-Elk**.
![Image](https://www.dropbox.com/s/yb9xtjmo7gvl85v/SyMsK791w_Bkz-RenJD.png?dl=1)
### Create a new Virtual Machine
- Create a new Ubuntu VM with a 4GB minimum RAM size.
- The IP address will be public.
- The time zone should be the same as your new VNet (**EastNet**), for me it was **East US 2**.
- The VNet will be **EastNet**, and the resource group will be the one we have for the other VMs, which is **RedTeam**.
- I used the public key from the ansible container and my username as **sysadmin**.
- I named my VM: **ELK**.
![Image](https://www.dropbox.com/s/31ivo96dytk19c3/SyMsK791w_Hy2WE-2kD.png?dl=1)

![Image](https://www.dropbox.com/s/2yh1hq3wvwsxiqv/SyMsK791w_rJ7Y4-31v.png?dl=1)

![Image](https://www.dropbox.com/s/ad2n1dje920bde4/SyMsK791w_B112NWnyP.png?dl=1)
- Once the VM is created, I ssh'd into the VM to make sure it works:
```bash
root@b1dea65e61f4:/etc/ansible# ssh sysadmin@10.1.0.4
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.3.0-1032-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Jul 15 04:27:46 UTC 2020

  System load:  0.14               Processes:              129
  Usage of /:   16.0% of 28.90GB   Users logged in:        0
  Memory usage: 73%                IP address for eth0:    10.1.0.4
  Swap usage:   0%                 IP address for docker0: 172.17.0.1


2 packages can be updated.
2 updates are security updates.


Last login: Wed Jul 15 02:51:33 2020 from 10.0.0.7
sysadmin@ELK:~$ 
```
### Download and Configure the Container
- We will add the new **VM IP address** to the **hosts** file in the ansible container ("**dvwa**" - name of container):
  - I went the **hosts** file:
```bash
root@b1dea65e61f4:~# cd /etc/ansible/
root@b1dea65e61f4:/etc/ansible# ll
total 52
drwxr-xr-x 1 root root  4096 Jul 15 02:48 ./
drwxr-xr-x 1 root root  4096 Jul  8 04:03 ../
-rw-r--r-- 1 root root 19988 Jul 10 03:55 ansible.cfg
-rw-r--r-- 1 root root  1217 Jul 15 02:09 hosts
-rw-r--r-- 1 root root  1290 Jul 15 02:48 install-elk.yml
-rw-r--r-- 1 root root   723 Jul 11 18:19 pentest.yml
drwxr-xr-x 2 root root  4096 Dec  4  2019 roles/
```
- Edited the **hosts** file and added the elk IP address under the [**elk**] group:
```bash
[webservers]
## alpha.example.org
## beta.example.org
10.0.0.12 ansible_python_interpreter=/usr/bin/python3
10.0.0.13 ansible_python_interpreter=/usr/bin/python3
10.0.0.14 ansible_python_interpreter=/usr/bin/python3
## 192.168.1.110

[elk]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3
```
- Created a playbook that will **configure** the ELK server, named it **"install-elk.yml"**:
```yaml
root@b1dea65e61f4:/etc/ansible# cat install-elk.yml 
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: sysadmin
  become: true
  tasks:
    # Use apt module
    - name: Install containerd
      apt:
        update_cache: yes
        name: containerd
        state: present
    - name: Install docker.io
      apt:
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
      #start docker
    - name: start docker
      service:
        name: docker
        state: started
        enabled: yes
      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144
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
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
```
### Launched and Exposed the Container
- Ran the yaml playbook, made sure it worked:
```bash
root@b1dea65e61f4:/etc/ansible# ansible-playbook install-elk.yml
```
- SSH'd into the ELK server and ran "docker ps":
```bash
sysadmin@ELK:~$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                              NAMES
6b5479738d75        sebp/elk:761        "/usr/local/bin/star…"   2 hours ago         Up 2 hours          0.0.0.0:5044->5044/tcp, 0.0.0.0:5601->5601/tcp, 0.0.0.0:9200->9200/tcp, 9300/tcp   elk
```
- As we can see from above, the container was successfully created, with the image "sebp/elk:761".
### Identity and Access Management
- We are going to restrict access to the ELK VM through the **ELK Network Security Group**:
![Image](https://www.dropbox.com/s/pw0evnc2sav0c30/SyMsK791w_HJ0b9Z2kw.png?dl=1)

- Once in the NSG for ELK, we are going to add an inbound rule that will allow access from our computer to the ELK server on **port 5601**:
![Image](https://www.dropbox.com/s/e8jpnca6l78zklm/SyMsK791w_B10B9b2Jw.png?dl=1)
- Next, we will add another security rule that will **restrict all access** to the ELK server, with a higher priority number:
![Image](https://www.dropbox.com/s/55mw84fuu6a94lx/SyMsK791w_ryoo5b2JP.png?dl=1)
- Finally, we will verify that we can log into the server by accessing on our browser, **[ELK-public-IP]:5601/app/kibana**:
  - Note: the public IP will always change every time we restart it.
![Image](https://www.dropbox.com/s/d4jpjo0hjv5xhi4/SyMsK791w_S1KEjZn1w.png?dl=1)
## 7) Install Filebeat
- We can use Filebeat to collect, parse, and visualize ELK logs in a single command. This will help us better track our organizational goals.
### Install Filebeat on the DVWA container
- First, we will start the virtual machines (including the ELK server) on Azure.
- Then, we will access the kibana page and make sure it works.
- We will start the dvwa container within the jump box vm:
```bash
azadmin@Jump-Box-Provisioner:~$ sudo docker start dvwa
dvwa
azadmin@Jump-Box-Provisioner:~$ sudo docker attach dvwa
root@b1dea65e61f4:~# 
```
- I jumped back to the kibana page and found the DEB page for creating a system log and will use this guide to create our filebeat playbook.
![Image](https://www.dropbox.com/s/3sc9zs6qmfsze16/SyMsK791w_HyKR9olev.png?dl=1)

### Create Filebeat configuration file
- I created a folder called files in /etc/ansible of the container:
```bash
root@b1dea65e61f4:~# cd /etc/ansible/
root@b1dea65e61f4:/etc/ansible# mkdir files
root@b1dea65e61f4:/etc/ansible# ll
total 56
drwxr-xr-x 1 root root  4096 Jul 17 01:53 ./
drwxr-xr-x 1 root root  4096 Jul  8 04:03 ../
-rw-r--r-- 1 root root 19988 Jul 10 03:55 ansible.cfg
drwxr-xr-x 2 root root  4096 Jul 17 01:53 files/
-rw-r--r-- 1 root root  1217 Jul 15 02:09 hosts
-rw-r--r-- 1 root root  1290 Jul 15 02:48 install-elk.yml
-rw-r--r-- 1 root root   723 Jul 11 18:19 pentest.yml
drwxr-xr-x 2 root root  4096 Dec  4  2019 roles/
```
- Once the folder is created, I will download the filebeat-config.yml:
```bash
root@b1dea65e61f4:/etc/ansible# curl https://gist.githubusercontent.com/slape/5cc350109583af6cbe577bbcc0710c93/raw/eca603b72586fbe148c11f9c87bf96a63cb25760/Filebeat > /etc/ansible/files/filebeat-config.yml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 73112  100 73112    0     0   190k      0 --:--:-- --:--:-- --:--:--  192k
```
- I opened the filebeat-confi.yml file and added the ELK server private IP address in two areas:
```bash
output.elasticsearch:
  hosts: ["10.1.0.4:9200"]
  username: "elastic"
  password: "changeme"
  
setup.kibana:
  host: "10.1.0.4:5601"
```
### Create Filebeat Installation Playbook
- I created the filebeat-playbook.yml under /etc/ansible/roles.
- Once created, using the DEB page, I added the needed commands:
```bash
root@b1dea65e61f4:/etc/ansible/roles# cat filebeat-playbook.yml
---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start
```
- Once the filebeat-playbook.yml is done, I will run it:
```bash
root@b1dea65e61f4:/etc/ansible/roles# ansible-playbook filebeat-playbook.yml 

PLAY [installing and launching filebeat] ************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [10.0.0.14]
ok: [10.0.0.12]
ok: [10.0.0.13]

TASK [download filebeat deb] ************************************************************************************************************************
[WARNING]: Consider using the get_url or uri module rather than running 'curl'.  If you need to use command because get_url or uri is insufficient
you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.

changed: [10.0.0.14]
changed: [10.0.0.12]
changed: [10.0.0.13]

TASK [install filebeat deb] *************************************************************************************************************************
changed: [10.0.0.14]
changed: [10.0.0.13]
changed: [10.0.0.12]

TASK [drop in filebeat.yml] *************************************************************************************************************************
changed: [10.0.0.12]
changed: [10.0.0.14]
changed: [10.0.0.13]

TASK [enable and configure system module] ***********************************************************************************************************
changed: [10.0.0.12]
changed: [10.0.0.13]
changed: [10.0.0.14]

TASK [setup filebeat] *******************************************************************************************************************************
changed: [10.0.0.12]
changed: [10.0.0.13]
changed: [10.0.0.14]

TASK [start filebeat service] ***********************************************************************************************************************
[WARNING]: Consider using the service module rather than running 'service'.  If you need to use command because service is insufficient you can add
'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.

changed: [10.0.0.12]
changed: [10.0.0.13]
changed: [10.0.0.14]

PLAY RECAP ******************************************************************************************************************************************
10.0.0.12                  : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.13                  : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.14                  : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
### Verify Installation and Playbook
- Back on the kibana page, I will check the data on the instruction page:
![Image](https://www.dropbox.com/s/pxto7o540jl5pll/SyMsK791w_rJCXEoexD.png?dl=1)
- Finally, I checked the dashboard to make sure things are running:
![Image](https://www.dropbox.com/s/im3giqimpvsw5fh/SyMsK791w_Bkqxroexv.png?dl=1)

![Image](https://www.dropbox.com/s/iwcl24qww3xje3h/SyMsK791w_Sy1fSjlgw.png?dl=1)
### Create a play to install Metricbeat
- Using the same steps for Filebeat, I created a config file for Metricbeat and added ELK's private IP in two areas:
```bash
output.elasticsearch:
  hosts: ["10.1.0.4:9200"]
  username: "elastic"
  password: "changeme"
  
setup.kibana:
  host: "10.1.0.4:5601"
```
- Under the roles folder, I will create the metricbeat-playbook.yml:
```bash
---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metric beat
    command: metricbeat setup

    # Use command module
  - name: start metric beat
    command: service metricbeat start
```
- Once both playbooks were created, I ran the metribeat-playbook.yml:
```bash
root@b1dea65e61f4:/etc/ansible/roles# ansible-playbook metricbeat-playbook.yml 

PLAY [Install metric beat] **************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************
ok: [10.0.0.14]
ok: [10.0.0.12]
ok: [10.0.0.13]

TASK [Download metricbeat] **************************************************************************************************************************
[WARNING]: Consider using the get_url or uri module rather than running 'curl'.  If you need to use command because get_url or uri is insufficient
you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.

changed: [10.0.0.12]
changed: [10.0.0.13]
changed: [10.0.0.14]

TASK [install metricbeat] ***************************************************************************************************************************
changed: [10.0.0.12]
changed: [10.0.0.13]
changed: [10.0.0.14]

TASK [drop in metricbeat config] ********************************************************************************************************************
changed: [10.0.0.12]
changed: [10.0.0.14]
changed: [10.0.0.13]

TASK [enable and configure docker module for metric beat] *******************************************************************************************
changed: [10.0.0.12]
changed: [10.0.0.13]
changed: [10.0.0.14]

TASK [setup metric beat] ****************************************************************************************************************************
changed: [10.0.0.13]
changed: [10.0.0.14]
changed: [10.0.0.12]

TASK [start metric beat] ****************************************************************************************************************************
[WARNING]: Consider using the service module rather than running 'service'.  If you need to use command because service is insufficient you can add
'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.

changed: [10.0.0.12]
changed: [10.0.0.13]
changed: [10.0.0.14]

PLAY RECAP ******************************************************************************************************************************************
10.0.0.12                  : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.13                  : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
10.0.0.14                  : ok=7    changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
- Now that the playbook ran successfully, I will check the data on the kibana page:
![Image](https://www.dropbox.com/s/q9dh7ct8ufr5ds1/SyMsK791w_BkzWKsxxD.png?dl=1)
- Finally, we will check the dashboard, to make sure it works:

![Image](https://www.dropbox.com/s/hp4bpdak2re3bjn/SyMsK791w_ryWkFjelP.png?dl=1)

![Image](https://www.dropbox.com/s/j1yw4xk1qaqr5se/SyMsK791w_rkYttiglw.png?dl=1)
