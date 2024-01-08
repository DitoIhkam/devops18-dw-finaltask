![alt text](?raw=true)
# Monitoring

## Monitor Resources for appserver & gateway 

Sebelum kita memonitoring kedua server tersebut, kita perlu memastikan node exporter di kedua server itu telah terinstall node exporter serta target di prometheus mengarah ke ketiga node exporter tersebut.

Appserver (gambar metric)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.1%20Metric%20app.png?raw=true)

Gateway (gambar metric)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.2%20Metric%20gate.png?raw=true)

Prometheus Targets

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.3%20Prom.png?raw=true)


## Create a fully working dashboard in grafana (Template/DIY)

Setelah kita sudah memastikan resource monitor berjalan dengan baik, selanjutnya kita akan membuat dashboard untuk memonitoring kedua server tersebut menggunakan template grafana.

Pertama login dan setting grafana agar target mengarah ke prometheus yang sudah kita set

(gambar setting prometheus)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.4%20Graf.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.5%20Graf%20Dashboard.png?raw=true)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.6%20Graf%20setting%20prom.png?raw=true)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.7%20prom%20settingggs.png?raw=true)

lalu kita buat dashboard menggunakan settingan import 1860 seperti dibawah
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.8%20import%201.png?raw=true)

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.9%20import%202.png?raw=true)





## Grafana Alerts

Setelah membuat dashboard untuk memonitoring, terakhir saya akan membuat alert yang mana nanti akan ada peringatan masuk ketika cpu, ram, dan memory usage melebihi batas yang telah ditentukan.

dibagian kiri atas, klik garis tiga dan pindah ke menu alerting, lalu pilih contact point, dan set integration ke discord serta masukkan webhook url discord. Apabila sudah dimasukkan webhooknya, bisa di test send notification bila perlu untuk mengetahui sudah terhubung atau belum ke discord

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.10%20edit%20contact%20point.png?raw=true)


Setelah itu, kita setting notification policy agar mengarah ke discord dan bukan ke email ataupun lainnya

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.11%20arah%20contact%20point.png?raw=true)

ini menu dimana kita menambahkan alert rules

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.12%20cara%20menambahkan%20alert%20rules.png?raw=true)

lalu kita ke menu alert rules untuk menyetel cpu, ram, dan disk dibawah ini

CPU usage ove 80% & RAM usage over 10%
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.13%20script%20promql.png?raw=true)

Disk usage over 80% & Hasil Discord
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/5.%20Monitoring/images/5.14%20script%20dan%20hasi%3B.png?raw=true)

