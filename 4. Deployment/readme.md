![alt text](?raw=true)
# Deployment


## Staging

Pada tahap ini saya akan membuat docker image sekecil mungkin untuk frontend dan baceknd. Berikut adalah beberapa file2 Dockerfile serta docker compose

### Front End

`Dockerfile`

```
# Stage 1: Build the application
FROM node:16-alpine AS builder
WORKDIR /app
COPY . .
RUN yarn install

# Stage 2: Create a smaller image
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app /app
EXPOSE 3000
CMD ["yarn", "start"]
```

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.1%20fe%20staging%20dockerfile.png?raw=true)



### Back End

`Dockerfile`

```
FROM golang:1.16-alpine
RUN mkdir /app
COPY . /app
WORKDIR /app
RUN go get ./ && go build && go mod download
EXPOSE 5000
CMD ["go", "run", "main.go"]
```

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.2%20be%20staging%20dockerfile.png?raw=true)


### Docker Compose

file `docker-compose.yml` berfungsi untuk membuild dan menjalankan frontend, backend serta database secara bersamaan

```
version: '3.8'
services:
  frontend:
    build: ./fe-dumbmerch2
    container_name: fe-container-staging
    image: kelompok2/fe-staging
    ports:
      - "3000:3000"
    depends_on:
      - postgres
    networks:
      - network
    restart: unless-stopped

  backend:
    build: ./be-dumbmerch2
    container_name: be-container-staging
    image: kelompok2/be-staging
    ports:
      - "5000:5000"
    depends_on:
      - postgres
    networks:
      - network
    restart: unless-stopped

  postgres:
    image: "postgres"
    container_name: db-container-production
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_USER: postgres
      POSTGRES_DB: postgres
    volumes:
      - "./data/postgresql:/var/lib/postgresql/data"
    networks:
      - network
    ports:
      - "5432:5432"
    restart: unless-stopped

networks:
  network:
```

lalu saya jalankan docker compose dengan perintah berikut 

`docker compose up -d`

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.3%20docker%20compose%20up%20staging.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.41%20docker%20ps%20dan%20images.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.42%20hasil%201.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.43%20hasil%202.png?raw=true)



## Production

Pada tahap ini saya akan membuat docker image siap deploy. Berikut adalah beberapa file2 Dockerfile serta docker compose

### Front End

`Dockerfile`
```
FROM node:16-alpine3.11 as production
WORKDIR /home/app
COPY . .
RUN npm install
RUN npm run build

FROM node:16-alpine3.11
WORKDIR /home/app
COPY --from=production /home/app /home/app
RUN npm install -g server
EXPOSE 3000
CMD ["npx","serve","build","-l","3000"]
```

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.5%20fe%20prod%20dockerfile.png?raw=true)


### Back End
untuk bagian backend saya menggunakan distroless

`Dockerfile`

```
FROM golang:1.18-alpine as distroless
WORKDIR /home/app
COPY . .
RUN CGO_ENABLED=0  go build

#distroless
FROM gcr.io/distroless/cc-debian11
WORKDIR /home/app
COPY --from=distroless /home/app /home/app
CMD ["/home/app/dumbmerch"]
```

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.6%20be%20prod%20dockerfile.png?raw=true)

### Docker Compose
`docker-compose.yml`

file docker-compose.yml berfungsi untuk membuild dan menjalankan frontend, backend serta database secara bersamaan

```
version: '3.8'
services:
  frontend:
    build: ./fe-dumbmerch2
    container_name: fe-container-prod
    image: kelompok2/fe-prod
    ports:
      - "3000:3000"
    depends_on:
      - postgres
    networks:
      - network
    restart: unless-stopped

  backend:
    build: ./be-dumbmerch2
    container_name: be-container-prod
    image: kelompok2/be-prod
    ports:
      - "5000:5000"
    depends_on:
      - postgres
    networks:
      - network
    restart: unless-stopped

  postgres:
    image: "postgres"
    container_name: db-container-production
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_USER: postgres
      POSTGRES_DB: postgres
    volumes:
      - "./data/postgresql:/var/lib/postgresql/data"
    networks:
      - network
    ports:
      - "5432:5432"
    restart: unless-stopped

networks:
  network:
```



lalu saya jalankan docker compose dengan perintah berikut

docker compose up -d

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.7%20docker%20compose%20up%20prod.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.8%20docker%20ps%20dan%20images.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.42%20hasil%201.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.43%20hasil%202.png?raw=true)



## CI/CD

Script for front end

```
name: CI/CD Pipeline

on:
  push:
    branches: [production]
  pull_request:
    branches: [production]
  workflow_dispatch:
  
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

#    - name: Git Pull
#      run: |
#        cd /home/ditoihkam/fe-dumbmerch2
#        git pull

    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}


    - name: SSH Login
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.143
        username: ditoihkam
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: 'echo "Logged in to SSH"'


    - name: Build and Push Docker Image
      run: |
        docker build -t kelompok2/fe-dumbmerch2 .
        docker push kelompok2/fe-dumbmerch2
    - name: SSH Deploy
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.143
        username: ditoihkam
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /home/ditoihkam/fe-dumbmerch2
          docker compose up -d
```

Scritp for backend

```
name: CI/CD Pipeline

on:
  push:
    branches: [production]
  pull_request:
    branches: [production]
  workflow_dispatch:
  
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

#    - name: Git Pull
#      run: |
#        cd /home/ditoihkam/fe-dumbmerch2
#        git pull

    - name: Login to Docker Hub
      uses: docker/login-action@v1
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}


    - name: SSH Login
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.143
        username: ditoihkam
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: 'echo "Logged in to SSH"'


    - name: Build and Push Docker Image
      run: |
        docker build -t kelompok2/be-dumbmerch2 .
        docker push kelompok2/be-dumbmerch2
    - name: SSH Deploy
      uses: appleboy/ssh-action@master
      with:
        host: 103.175.221.143
        username: ditoihkam
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /home/ditoihkam/be-dumbmerch2
          docker compose up -d
```



## Github Action Setup

Untuk mengatur agar github action bisa berjalan, kita perlu mengatur secrets didalam github repo nya. Pergi ke bagian setting, di bagian Secrets and Variable klik Action, lalu klik new repository secrets. Disini saya mengatur 3 secrets yang nantinya akan terhubung ke github action untuk cicd seperti di gambar

![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.91%20setting%20action.png?raw=true)
untuk token docker hub bisa diambil dari gambar ini
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/last.png?raw=true)


Masih didalam repository nya, kita pergi ke bagian tab action, lalu pilih `set up a workflow yourself`, lalu copy paste script cicd nya. setelah di commit, github action akan memproses cicd nya


![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.92%20setting%20action.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.93%20setting%20action.png?raw=true)
![alt text](https://github.com/DitoIhkam/devops18-dumbways-ihkam-audito/blob/main/4.%20Deployment/images/4.94%20setting%20action.png?raw=true)


