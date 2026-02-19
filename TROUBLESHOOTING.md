# 📄 TROUBLESHOOTING.md


# Troubleshooting & Errors Faced – Docker Multi-Site Reverse Proxy Project

During the implementation of hosting two websites using Docker and Nginx reverse proxy on AWS EC2, I encountered several real-world issues. Below is a complete record of the errors and how they were resolved.

---

## 1️⃣ 502 Bad Gateway Error

### ❌ Error
Browser displayed:
502 Bad Gateway

### 🔎 Root Cause
The reverse proxy container was unable to properly forward traffic to backend containers.

Possible causes:
- Incorrect Nginx configuration
- Missing trailing slash in proxy_pass
- Path routing mismatch

### 🔍 Debugging Steps

Checked reverse proxy logs:
```

docker logs reverse_proxy

```

Tested backend connectivity from inside reverse proxy container:
```

docker exec -it reverse_proxy sh
curl [http://site1](http://site1)
curl [http://site2](http://site2)

```

Confirmed backend containers were reachable.

### ✅ Solution

Updated Nginx configuration:

```

location /site1/ {
proxy_pass [http://site1/](http://site1/);
}

location = /site1 {
return 301 /site1/;
}

```

Restarted containers cleanly:
```

docker compose down
docker compose up -d --build

```

---

## 2️⃣ Port 80 Already In Use

### ❌ Error
```

failed to bind host port 0.0.0.0:80: address already in use

```

### 🔎 Root Cause
Host machine had Nginx service already running on port 80.

Verified using:
```

sudo ss -tulnp | grep :80

```

### ✅ Solution
Stopped and disabled host Nginx:
```

sudo systemctl stop nginx
sudo systemctl disable nginx

```

Then restarted Docker containers successfully.

---

## 3️⃣ Docker Compose 'ContainerConfig' Error

### ❌ Error
```

ERROR: for reverse_proxy 'ContainerConfig'
docker-compose==1.29.2

```

### 🔎 Root Cause
Old Docker Compose v1 was incompatible with current Docker Engine version.

### ✅ Solution
Removed old docker-compose and installed Docker Compose v2:

```

sudo apt remove docker-compose -y
sudo apt install docker-compose-plugin -y

```

Used:
```

docker compose up -d

```

(Note: Docker Compose v2 uses space instead of hyphen.)

---

## 4️⃣ Containers Running But Not Accessible Externally

### ❌ Problem
Containers were running but websites were not accessible from browser.

### 🔎 Root Cause
Reverse proxy container was missing port mapping:

```

ports:

* "80:80"

```

Without this, container was not exposed to EC2 host port.

### ✅ Solution
Added port mapping in docker-compose.yml and restarted stack.

---

## 5️⃣ Backend Path Routing Issue

### ❌ Problem
Accessing:
```

/site1

```
But backend container only serves:
```

/

```

### 🔎 Root Cause
Improper handling of trailing slash in Nginx.

### ✅ Solution
Added redirect rule:

```

location = /site1 {
return 301 /site1/;
}

```

Ensured proxy_pass had trailing slash:
```

proxy_pass [http://site1/](http://site1/);

```

---

# 🎯 Key Learnings From Debugging

- How reverse proxy forwards traffic internally
- Importance of Docker internal DNS (service names)
- Difference between host networking and container networking
- Proper troubleshooting of 502 errors
- Port conflict resolution on Linux
- Docker Compose v1 vs v2 compatibility issues
- Importance of trailing slash behavior in Nginx

---
