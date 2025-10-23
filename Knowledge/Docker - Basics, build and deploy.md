## 1. What is Docker?

Docker lets you **package an app and all its dependencies** into a lightweight container image that runs consistently across environments.

A _container_ is an **isolated runtime instance** of an image.  
An _image_ is a **snapshot** of your app + its environment.

---

## 2. Basic Dockerfile

Dockerfile is a file called `dockerfile` you create wherever in your repo. Then when you run docker commands like `docker build` it uses this file as the reference for what it's supposed to do. The file name needs to be exactly the same for the commands to work.

A simple example for a web app:

```dockerfile
# Use a lightweight runtime base 
FROM node:18-alpine  

# Set working directory 
WORKDIR /app  

# Copy source and install deps 
COPY package*.json ./ R
UN npm install  

# Copy remaining files 
COPY . .  

# Build and expose 
RUN npm run build 
EXPOSE 3000  

# Default start command 
CMD ["npm", "run", "start"]
```

### Key Dockerfile Concepts

| Instruction | Meaning                                       |
| ----------- | --------------------------------------------- |
| `FROM`      | Base image (defines OS + runtime)             |
| `WORKDIR`   | Current directory inside container            |
| `COPY`      | Copy files from host → container              |
| `RUN`       | Execute shell commands while building         |
| `EXPOSE`    | Document which port the container listens on  |
| `CMD`       | Default runtime command when container starts |

---

## 3. Building and Running

```shell
# Build the image
docker build -t myapp .

# Run it interactively
docker run -p 8080:3000 myapp
```

- `-t myapp` → tags your image
- `-p 8080:3000` → maps local port 8080 → container’s port 3000

Check containers:

```shell
docker ps
docker stop <id>
docker logs <id>
```


---

## 4. Docker Compose — Multi-Container Setup

Compose allows you to define **multiple containers (services)** that work together (e.g. app + db + proxy).

It's always called `docker-compose.yaml`

**Example `docker-compose.yml`:**
```yaml
version: "3.9"

services:
  app:
    build: .
    container_name: myapp
    expose:
      - 3000
    environment:
      - NODE_ENV=production

  nginx-proxy:
    image: nginx:alpine
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - app
```

Instantiate everything:
`docker-compose up -d`

### How do you configure the containers?

Since docker containers are static - immutable images, how do you configure these images for deployment - as in the default username, database connection strings, so many things need to be set / configured for a deployment based on what the image is.

You get two ways to do that - 
1. **Environment variables -** you can set the ENV variables for the docker containers, which will be used by the app inside the container.
2. **Config files -** You can create persistent storage and mount them to wherever you want in the container. Anything container write to it, or you put to it is available to both host and the container. 

The specifics depend on the container image and the author. They would provide steps to how you can configure these. You can follow it to construct your docker compose file and have a customized deployment.

---

## 5. Reverse Proxy Setup (Nginx)

Your **nginx-proxy** acts as the entry point — routing HTTP(S) traffic to your app container.

Example config (`nginx/conf.d/default.conf`):

```nginx
server {
  listen 80;
  server_name example.com;

  location / {
    proxy_pass http://app:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

### Key Forward Headers

| Header              | Purpose                        |
| ------------------- | ------------------------------ |
| `Host`              | Preserve original domain       |
| `X-Forwarded-For`   | Client IP address              |
| `X-Forwarded-Proto` | Indicates HTTP or HTTPS scheme |

These headers let the backend know the **true client request origin** when behind a proxy.

>**Note:** You can use nginx-proxy image to get a ready made deployment. Look at budgetbud's source code for details

---

## 6. Folder Layout (Typical)

```
project/
│
├── app/                # Source code
│   ├── package.json
│   └── Dockerfile
│
├── nginx/
│   └── conf.d/
│       └── default.conf
│
├── docker-compose.yml
└── certs/              # SSL (optional)
```

---

## 7. Common Commands

|Command|Description|
|---|---|
|`docker build -t name .`|Build image|
|`docker run -d -p 80:80 name`|Run detached|
|`docker-compose up -d`|Start stack|
|`docker-compose down`|Stop all|
|`docker exec -it name sh`|Open shell inside container|
|`docker system prune`|Cleanup dangling images/containers|

---

## 8. Deployment (Generic Flow)

1. **Build image**
```
docker build -t myapp:latest .
```
2. **Push to registry (optional)**

```
docker tag myapp myregistry.com/myapp:latest 
docker push myregistry.com/myapp:latest
```

3. **Deploy to server**
	- Pull image on remote machine    
	- Run `docker-compose up -d`
	- Optionally integrate with systemd or CI/CD to auto-redeploy

---

## 9. Tips

- Prefer **Alpine** base images for smaller size.
- Use `.dockerignore` to skip unnecessary files (like `node_modules/`).
- Multi-stage builds help keep final image minimal.
- Always test locally before pushing to production.

---

## 10. Minimal Multi-Stage Example

```dockerfile
# Stage 1 - build
FROM node:18-alpine AS builder
WORKDIR /src
COPY . .
RUN npm ci && npm run build

# Stage 2 - serve
FROM nginx:alpine
COPY --from=builder /src/dist /usr/share/nginx/html
EXPOSE 80

```

>**Note:** Each `FROM` statement resets the container image - hence you can use dockerfile as a single place to generate your build output and final image. This ensures each container is built in the exact same way, regardless of the workstation building it.

