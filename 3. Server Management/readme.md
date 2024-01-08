
# Server Management
## Server only login with ssh & Password login disable

Saya disini mengatur agar ketika login tidak perlu menggunakan password serta mengatur agar login hanya bisa dilakukan menggunakan ssh key saja. Masing-masing server untuk pengaturannya berada dilokasi yang berada digambar dan mengganti menjadi `PasswordAuthentication no` dengan menggunakan sudo

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/3.%20Server%20Management/images/3.1%20login%20disable.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/3.%20Server%20Management/images/3.2%20login%20only%20with%20ssh.png?raw=true)


## Create SSH Config

saya membuat sistem one gateway dengan membuat file config berlokasi di `home/ditoihkam/.ssh/config` agar semua akses ssh melalui perantara gateway, serta agar ketika login tidak mengetikan username dan ip. Cukup mengetikan kalimat yang telah kita atur, dalam hal ini saya atur `appserver` yang akan mengarah ke username ditoihkam dengan ip tertera, begitu juga untuk `gateway`

```
host gateway
    HostName 103.175.217.129
    User ditoihkam

host appserver
    Hostname 103.175.221.143
    User ditoihkam
    ProxyCommand ssh gateway -W %h:%p
    IdentityFile /home/ditoihkam/.ssh/id_rsa
```

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/3.%20Server%20Management/images/3.3%20ssh%20config%20%26%20login%20appserer%20test.png?raw=true)


## UFW enable only with port allowed

Saya membuat Uncomplicated Firewall (UFW) agar hanya bisa mengakses port yang saya izinkan saja. Disini saya menggunakan ansible dengan script dibawah

```
- become: true
  gather_facts: false
  hosts: gateway
  tasks:
    - name: install firewall
      apt:
        name: ufw
        update_cache: yes
        state: latest
    - name: enable ufw
      community.general.ufw:
        state: enabled
        policy: allow
    - name: rules
      community.general.ufw:
        rule: allow
        proto: tcp
        port: "{{ item }}"
      with_items:
      - 22
      - 80
      - 443
      - 3000
      - 3001
      - 5000
      - 3306
      - 9090
      - 9100
    - name: enable
      community.general.ufw:
        state: reloaded
        policy: allow
```


![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/3.%20Server%20Management/images/3.4%20ufw%20port%20allowed%20only.png?raw=true)
