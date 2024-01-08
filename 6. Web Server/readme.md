![alt text](?raw=true)
## Create Domains

Domain untuk App, Backend dan lainnya sudah saya buat menggunakan dns cloudflare

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/6.%20Web%20Server/images/6.1%20dns.png?raw=true)


## All Domain Are HTTPS

Lalu yang terakhir menginstallasi semua web menggunakan Certbot SSL Default yang sudah saya lakukan di task 1 provisioning bagian challange, perintahnya seperti dibawah ini

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
> perintah diatas dijalankan menggunakan ansible-playbook

dan berikut hasil dari certbot yang mana website sudah menggunakan https


Ini untuk beberapa website yang sudah saya lakukan wildcard certbot

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/6.%20Web%20Server/images/Screenshot%202023-10-23%20011511.png?raw=true)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/6.%20Web%20Server/images/Screenshot%202023-10-23%20011522.png?raw=true)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/6.%20Web%20Server/images/Screenshot%202023-10-23%20011533.png?raw=true)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/6.%20Web%20Server/images/Screenshot%202023-10-23%20011543.png?raw=true)
