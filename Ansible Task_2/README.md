# Integration of Docker 🐳 and Ansible

 ❗Ansible Task❗

Description:

Write an Ansible PlayBook that does the following operations in the managed nodes:

🔹 Configure Docker

🔹 Start and enable Docker services

🔹 Pull the httpd server image from the Docker Hub

🔹 Run the httpd container and expose it to the public

🔹 Copy the html code in /var/www/html directory and start the web server


![alt text](https://www.ansible.com/hubfs/2016_Images/Blog_Headers/Ansible-Docker-Blog-2.png)


## Pre-Configuration Required :

##### RHEL 8 :

Install RHEL 8 or any other Linux Distribution. In my case I am Using RHEL8 .

![alt text](https://media-exp1.licdn.com/dms/image/C5612AQGLeFC6i8sHgA/article-inline_image-shrink_1000_1488/0?e=1612396800&v=beta&t=ByK69l8cqQGRdZPYVu_Se7slVYM2GIo8JO3vMf2YsRQ )




## Ansible Installation :

- ##### Install Python3

```bash
yum install python3 -y
```

- yum epel-release configuration to download software from Internet.
Download the epel-relese from link :
[click here](https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm)
or can use wget command--

```bash 

wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

```

##### Install epel-release from file downloaded


```bash

rpm -ivh ./epel-release-latest-8.noarch.rpm

```

##### Install Ansible

```bash 

pip3 install ansible


```

##### Check the Ansible version

```bash

ansible --version

```

## Ansible Configuration


##### Create a file for inventory

```bash
vim /etc/myhosts.txt

```
##### Write the inventories or the managed nodes you have

```bash 
192.168.43.64 ansible_ssh_user=root ansible_ssh_pass=<password>

```
###### NOTE:
Above in the inventory we have to give the IP address of the Managed Node, in my case that is the IP address. For you find the IP address using ifconfig command in managed node. 


192.168.43.64 is my inventory IP

ansible_ssh_user : User you want to login in managed node

ansible_ssh_pass: provide the password of the above user

We can add a group of inventory IP addresses like a group of managed nodes for Front-end and some are for Database , may be Application Server or Mail server


##### Create a directory /etc/ansible

```bash 

mkdir /etc/ansible

```

##### Create a file /etc/ansible/ansible.cfg


```bash 

vim /etc/ansible/ansible.cfg

```
and write the following lines in the above file


```bash 

[defaults]
inventory = /etc/myhosts.txt
host_key_checking= False

```

Included the file /etc/myhosts.txt in the configuration file of ansible.

When we do ssh from controller node to managed node. we have to accept the public key from manged node.we are doing this with program there is no one to give permission so we are disabling the host_key_checking.


##### Install sshpass

```bash 

yum install sshpass -y

```
Now, we are good to go we can configure any thing managed nodes.


## Docker In Managed Node :

- ##### Docker repository

Adds the docker repository to RHEL. baseurl provide the url for the software.


```yml 

- yum_repository:

      description: DOCKER YUM repo
      name: docker
      baseurl: https://download.docker.com/linux/centos/7/x86_64/stable/
      gpgcheck: no
      state: present

```




```yml 



```


- ##### Install Docker

```yml 

- package:
      name: docker-ce-18.09.1-3.el7.x86_64
      skip_broken: yes
      state: present

Start Docker service
  - service:
      name: docker
      state: started


```

  
- ##### Python Docker SDK

```yml 

 command: pip3 install docker

```

# Complete playbook


```yml

- hosts: all
  tasks:

  - file:
      state: directory
      path: "/dvd"

  - mount:
      src: "/dev/cdrom"
      path: "/dvd"
      fstype: "iso9660"
      state: mounted

  - yum_repository:
      baseurl: "/dvd1/AppStream"
      name: "mydvd1"
      description: "This is dvd1ffff"
      gpgcheck: no

  - yum_repository:
      name: "mydvd2"
      description: "This is dvd1sss"
      baseurl: "/dvd1/BaseOS"
      gpgcheck: no

  - yum_repository:
      baseurl: "https://download.docker.com/linux/centos/7/x86_64/stable/"
      name: "docker-yum"
      description: "Docker Repo"
      state: present
      gpgcheck: no

  - package:
      name: "python3"
      state: present 
 
  - package:
      name: "docker-ce-18.09.1-3.el7.x86_64"
      state: present

  - service:
      name: "docker"
      state: started
      enabled: yes

  - pip:
      name: "docker-py"

  - community.general.docker_image:
      name: "httpd"
      source: pull

  - file:
      path: "/webpages"
      state: directory

  - copy:
      src: "/root/index.html" 
      dest: "/webpages"
 
  - community.general.docker_container:
      name: "myos"
      image: "httpd"
      exposed_ports: "80"
      ports: "8998:80"
      volumes: "/webpages:/usr/local/apache2/htdocs/"



```

Run this Play Book

```bash 
ansible-playbook docker.yml
```

docker.yml is the playbook name.


## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)
