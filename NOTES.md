
IaaS:
* `apt-get update/upgrade`
* `apt-get install git`
*` install node.js`
* `install npm`
* `npm install`
    * note vulnerabilitues
* `sudo ln -s /usr/bin/nodejs /usr/local/bin/node`
* `apt install redis-server`
* `npm start`
* open 3000 in gcloud)

CaaS

Create Docker File
* FROM node:carbon  *- who maintains this? Is it safe?*
* WORKDIR /usr/src/app
* COPY package.json ./
* COPY npm-shrinkwrap.json ./
* RUN npm install
* COPY . .
* CMD ["npm", "start"] *- entry point*

Build Docker Image

`docker build -t yourname/redis-app:1.0 .`

Write Deployment(s):

Need a Deployment and a Service for each of the Web and Redis components

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: redis-chat-redis
spec:
  selector:
    matchLabels:
      app: redis-chat-redis
      deployment: redis-chat
  replicas: 1
  template:
    metadata:
      labels:
        app: redis-chat-redis
        deployment: redis-chat
    spec:
      containers:
      - name: redis
        image: redis
        resources:
          requests:
            cpu: 200m
            memory: 200Mi
        ports:
        - containerPort: 6379
```

Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis-chat-redis
    deployment: redis-chat
  name: redis-chat-redis-service
spec:
  ports:
  - port: 6379
    name: redis
  selector:
    app: redis-chat-redis
```

Web Deployment:

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: redis-chat-web
spec:
  selector:
    matchLabels:
      app: redis-chat-web
      deployment: redis-chat
  replicas: 2  #why 2?
  template:
    metadata:
      labels:
        app: redis-chat-web
        deployment: redis-chat
    spec:
      containers:
      - name: redis-chat
        image: mcowger/redis-chat  # leave this
        resources:
          requests:
            cpu: 200m
            memory: 200Mi
        ports:
        - containerPort: 3000  #why 3000?
        env:
        - name: REDIS_HOST_NAME
          value: redis-chat-redis-service
```

Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: redis-chat-web # why these values
    deployment: redis-chat
  name: redis-chat-web-service
spec:
  ports:
  - port: 3000 
    name: redis
  type: NodePort #whats a nodeport?
  selector:
    app: redis-chat-web
```

PaaS

Create manifest

```yaml
applications:
- name: redis-chat
  memory: 512M
  random-route: true  #what does this do?
  buildpack: nodejs_buildpack
  services:
  - chat-db
```


