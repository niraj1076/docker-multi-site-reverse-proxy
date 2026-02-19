
# Docker Multi-Site Reverse Proxy (AWS EC2)

This project demonstrates how to host multiple websites on a single server using Docker and Nginx reverse proxy.

The setup was deployed on an AWS EC2 Ubuntu instance.

---

##  Project Objective

Host two separate websites on the same server using:

- Docker containers
- Docker Compose
- Nginx reverse proxy
- Path-based routing

---

## 🏗 Architecture

User Browser  
      ↓  
EC2 Port 80  
      ↓  
Reverse Proxy Container (Nginx)  
      ↓  
Docker Internal Network  
      ↓  
Site1 Container  
Site2 Container  

Only the reverse proxy exposes port 80 to the outside world.  
Backend containers remain internal for better isolation and security.

---

## 📂 Project Structure

```

multi-site/
│
├── docker-compose.yml
│
├── site1/
│   └── index.html
│
├── site2/
│   └── index.html
│
└── nginx/
└── default.conf

```

---

## ⚙ Step 1 — Launch EC2 Instance

- Ubuntu 22.04
- t2.micro
- Open port 80 in Security Group

---

## ⚙ Step 2 — Install Docker & Docker Compose

```

sudo apt update
sudo apt install docker.io docker-compose-plugin -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER

```

Logout and login again.

Verify:

```

docker --version
docker compose version

```

---

## ⚙ Step 3 — Create Website Files

### site1/index.html

```

<h1>This is Website 1</h1>
```

### site2/index.html

```
<h1>This is Website 2</h1>
```

---

## ⚙ Step 4 — Create Reverse Proxy Config

### nginx/default.conf

```
server {
    listen 80;

    location /site1/ {
        proxy_pass http://site1/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location = /site1 {
        return 301 /site1/;
    }

    location /site2/ {
        proxy_pass http://site2/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location = /site2 {
        return 301 /site2/;
    }
}
```

---

## ⚙ Step 5 — Docker Compose File

### docker-compose.yml

```
version: '3.8'

services:

  site1:
    image: nginx
    container_name: site1
    volumes:
      - ./site1:/usr/share/nginx/html
    restart: always

  site2:
    image: nginx
    container_name: site2
    volumes:
      - ./site2:/usr/share/nginx/html
    restart: always

  reverse_proxy:
    image: nginx
    container_name: reverse_proxy
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - site1
      - site2
    restart: always
```

---

## ⚙ Step 6 — Stop Host Nginx (If Installed)

If Nginx is running on host, stop it:

```
sudo systemctl stop nginx
sudo systemctl disable nginx
```

---

## ⚙ Step 7 — Run the Application

```
docker compose down
docker compose up -d
```

Verify containers:

```
docker ps
```

You should see:

```
0.0.0.0:80->80/tcp
```

---

## 🌐 Access Websites

```
http://<your-ec2-public-ip>/site1/
http://<your-ec2-public-ip>/site2/
```

---

## 🧠 Key DevOps Concepts Demonstrated

* Docker containerization
* Docker internal networking
* Reverse proxy architecture
* Port publishing
* Service isolation
* Path-based routing
* Troubleshooting 502 errors
* Port conflict resolution

---

## 🛠 Troubleshooting

### 502 Bad Gateway

Check:

```
docker logs reverse_proxy
```

Test backend connectivity:

```
docker exec -it reverse_proxy sh
curl http://site1
curl http://site2
```

### Port Already In Use

Stop host nginx:

```
sudo systemctl stop nginx
```

---


