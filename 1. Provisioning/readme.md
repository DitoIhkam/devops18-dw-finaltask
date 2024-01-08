![alt text](?raw=true)

# Provisioning Biznet GIO NEO Lite Server

Pertama, untuk menyediakan server Biznet, saya akan membuat secara langsung ke website `portal.biznetgio.com` dan login menggunakan akun nya. Saya akan membuat 2 server dengan spesifikasi sebagai berikut :

- Appserver - 2 vCPU, 2GB RAM
- Gateway - 1 vCPU, 1GB RAM
- 60GB Storage each server

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/1.%20Provisioning/images/1.1%20BIZNET.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/1.%20Provisioning/images/1.2%20BIZNET%20CONF.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/1.%20Provisioning/images/1.3%20BIZNET%20PAYMNT.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/1.%20Provisioning/images/1.4%20BIZNET.png?raw=true)


# Server Configuration using Ansible

Setelah server di biznet telah dibuat sesuai konfigurasi, selanjutnya kita akan membuat konfigurasi menggunakan ansible

Sebelum menjalankan perintah ansible-playbook untuk bisa menjalankan ansible, yang pertama kita install ansible dan setelah itu buat ansible.cfg agar ansible mengerti settingan yang harus dijalankan sesuai settingan kita.

`ansible.cfg`

```
[defaults]
inventory = ~/ftask2/source/Inventory
private_key_file = /home/ditoihkam/.ssh/id_rsa
host_key_checking = false
interpreter_python =  auto_silent
```

`Inventory`

```
[appserver]
103.175.221.143

[gateway]
103.175.217.129

[all:vars]
ansible_user=ditoihkam
ansible_ssh_private_key_file=/home/ditoihkam/.ssh/id_rsa
```
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/1.%20Provisioning/images/1.5%20ansible%20cfg%20dan%20invent.png?raw=true)

1.  Saya akan melakukan installasi dan menjalankan beberapa kebutuhan software/aplikasi seperti :

### Semua Server
- Copy ssh key dan public ke target virtual machine
--Disable Password--
- Install & Run
    - Docker engine 
    - Node exporter

### Appserver

- Clone repo fe & be dumbmerch
- Install & Run
	- Prometheus
	- Grafana

### Gateway
- Install NGINX
- copy reverse proxy
- SSL certificate

khusus dibagian prometheus harus kita buat konfigurasinya sebelum menjalankan dan menginstall software/aplikasi tersebut dengan file 'prometheus.yml'

berikut kodenya

```
scrape_configs:
  - job_name: Grafana
    scrape_interval: 5s
    static_configs:
      - targets:
        - nodeapp.ditoihkamf.studentdumbways.my.id
        - nodegate.ditoihkamf.studentdumbways.my.id
```

> kode diatas tidak perlu dijalankan menggunakan ansible-playbook

Berikut untuk script nya dan ketika menjalankan ansible-playbook.

`attach ssh`
```
- hosts: all
  become: true
  tasks:
    - name: Copy SSH public key to remote host
      authorized_key:
        user: ditoihkam
        key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
        state: present

    - name: Copy SSH key
      copy:
        src: /home/ditoihkam/.ssh/id_rsa
        dest: /home/ditoihkam/.ssh/id_rsa
      when: not ansible_check_mode
```



`Install docker`
```
- hosts: all
  become: true
  vars:
    user: ditoihkam
  tasks:
    - name: Install Docker Dependencies
      apt:
        update_cache: yes
        name:
          - lsb-release
          - ca-certificates
          - curl
          - gnupg

    - name: Install GPG Key for Docker
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg

    - name: Add Docker APT Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu focal stable

    - name: Install Docker Engine
      apt:
        update_cache: yes
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io

    - name: Install Docker Compose
      apt:
        name: docker-compose
        state: latest
        update_cache: yes

    - name: Install Python Dependencies
      apt:
        name: python3-pip
        state: latest
        update_cache: yes

    - name: Install Docker SDK for Python
      pip:
        name: docker
        state: latest
        executable: pip3

    - name: Add user to the docker group
      user:
        name: "{{ user }}"
        groups: docker
        append: yes
        state: present

    - name: Start Docker service
      service:
        name: docker
        state: started

    - name: Enable Docker on boot
      service:
        name: docker
        enabled: yes
```

`install & run node-exporter`

```
- hosts: all
  become: true
  vars:
    user: ditoihkam
  tasks:

    - name: Pull the bitnami/node-exporter Docker image
      docker_image:
        name: bitnami/node-exporter
        source: pull

    - name: Run the Node Exporter container
      docker_container:
        name: node-exp
        image: bitnami/node-exporter
        state: started
        restart_policy: unless-stopped
        published_ports:
          - "9100:9100"
```

`clone repo`
```
---
- hosts: appserver
  gather_facts: false
  tasks:
    - name: Clone fe-dumbmerch Repository
      git:
        repo: https://github.com/demo-dumbways/fe-dumbmerch
        dest: /home/ditoihkam/fe-dumbmerch
      when: not ansible_check_mode

    - name: Clone be-dumbmerch Repository
      git:
        repo: https://github.com/demo-dumbways/be-dumbmerch
        dest: /home/ditoihkam/be-dumbmerch
      when: not ansible_check_mode
```

`prometheus`

```
- name: Deploy Prometheus top on Docker
  hosts: appserver
  become: yes

  tasks:
    - name: Pull Prometheus Docker image
      docker_image:
        name: bitnami/prometheus
        source: pull

    - name: Create a directory if it doesn't exist
      ansible.builtin.file:
        path: /home/ditoihkam/prometheus
        state: directory
        mode: '0755'

    - name: Copy prometheus config
      copy:
        src: /home/ditoihkam/ftask2/source/prometheus.yml
        dest: /home/ditoihkam/prometheus/prometheus.yml

    - name: Run Prometheus container
      docker_container:
        name: Prometheus-container
        image: bitnami/prometheus
        state: started
        volumes:
          - /home/ditoihkam/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
        restart_policy: unless-stopped
        published_ports:
          - "9090:9090"
```

`grafana`
```
- name: Deploy Grafana top on Docker
  hosts: appserver
  become: yes

  tasks:
    - name: Pull Grafana Docker image
      docker_image:
        name: grafana/grafana
        source: pull

    - name: Run Grafana container
      docker_container:
        name: Grafana-container
        image: grafana/grafana
        state: started
        restart_policy: unless-stopped
        published_ports:
          - "3001:3000"
```

`reverse proxy`

```
server {
    server_name nodeapp.ditoihkamf.studentdumbways.my.id;

    location / {
             proxy_pass http://103.175.221.143:9100;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
    }
}



server {
    server_name nodegate.ditoihkamf.studentdumbways.my.id;

    location / {
             proxy_pass http://103.175.217.129:9100;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
    }
}



server {
    server_name prom.ditoihkamf.studentdumbways.my.id;

    location / {
             proxy_pass http://103.175.221.143:9090;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
    }
}



server {
    server_name graf.ditoihkamf.studentdumbways.my.id;

    location / {
             proxy_pass http://103.175.221.143:3001;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
    }
}



server {
    server_name ditoihkamf.studentdumbways.my.id;

    location / {
             proxy_pass http://103.175.221.143:3000;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
    }
}


server {
    server_name api.ditoihkamf.studentdumbways.my.id;

    location / {
             proxy_pass http://103.175.221.143:5000;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```


`install nginx`
```
---
- become: true
  gather_facts: false
  hosts: gateway
  tasks:
    - name: Installing nginx
      apt:
        name: nginx
        state: latest
        update_cache: yes
    - name: Start nginx
      service:
        name: nginx
        state: started
    - name: Copy reverse-proxy.conf
      copy:
        src: /home/ditoihkam/ftask2/source/reverse-proxy.conf
        dest: /etc/nginx/sites-enabled
    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
```







`ssl certificate`
```
---
- become: true
  gather_facts: false
  hosts: gateway
  tasks:

    - name: Install Certbot with Snap
      snap:
        name: certbot
        classic: yes
        state: present
      become: yes


    - name: Obtain and install SSL certificate with Certbot Nginx Node Exporter Gateway
      command: certbot --nginx -d nodegate.ditoihkamf.studentdumbways.my.id --non-interactive --agree-tos --email ditoihkam@gmail.com
      become: yes


    - name: Obtain and install SSL certificate with Certbot Nginx Node Exporter AppServer
      command: certbot --nginx -d nodeapp.ditoihkamf.studentdumbways.my.id --non-interactive --agree-tos --email ditoihkam@gmail.com
      become: yes

    - name: Obtain and install SSL certificate with Certbot Nginx Node Exporter AppServer
      command: certbot --nginx -d graf.ditoihkamf.studentdumbways.my.id --non-interactive --agree-tos --email ditoihkam@gmail.com
      become: yes

    - name: Obtain and install SSL certificate with Certbot Nginx Node Exporter AppServer
      command: certbot --nginx -d prom.ditoihkamf.studentdumbways.my.id --non-interactive --agree-tos --email ditoihkam@gmail.com
      become: yes

    - name: Obtain and install SSL certificate with Certbot Nginx Node Exporter AppServer
      command: certbot --nginx -d ditoihkamf.studentdumbways.my.id --non-interactive --agree-tos --email ditoihkam@gmail.com
      become: yes

    - name: Obtain and install SSL certificate with Certbot Nginx Node Exporter AppServer
      command: certbot --nginx -d api.ditoihkamf.studentdumbways.my.id --non-interactive --agree-tos --email ditoihkam@gmail.com
      become: yes
```
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/1.%20Provisioning/images/1.6%20ANSIBLE%20PLAYBOOK.png?raw=true)


jangan lupa untuk menyetel ip dns ip di cloudflare kearah server gateway semua seperti ini.

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/1.%20Provisioning/images/1.11%20dns.png?raw=true)


Lalu ini hasil untuk beberapa akses :

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/1.%20Provisioning/images/1.7%20Metric%20app.png?raw=true)


![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/1.%20Provisioning/images/1.9%20prom.png?raw=true)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/1.%20Provisioning/images/1.10%20graf.png?raw=true)


