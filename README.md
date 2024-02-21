# 2024程序员容器化上云之旅-WSL2版

前后端分离Web应用Docker容器化与Kubernetes/K8s上云

## 1 编程故事

[程序员吾真本-知乎专栏-2024程序员容器化上云之旅](https://www.zhihu.com/column/deliberate-practice-in-devops)

## 2 在Windows11的WSL2的Ubuntu中如何运行Shopping List Web App

### 2.1 Docker Setup

- Install Docker desktop with WSL2 for Windows 11

### 2.2 back-end setup

#### 2.2.1 JDK Setup

- Install JDK 17 tem using sdkman (for Ubuntu)

#### 2.2.2 IDE Setup

- Install Intellij IDEA Ultimate Edition
- Install JPA Buddy plugin in Intellij IDEA

#### 2.2.3 Blank Back-end Project Setup - start.spring.io

- lombok
- spring web
- spring data jpa
- flyway migration
- postgresql driver

#### 2.2.4 How to run the back-end application during development

- Start postgres database and pgadmin using docker compose

```shell
# Start Docker Desktop
cd infrastructure
docker compose up postgres pgadmin
```

- Check the database using pgadmin

open docker desktop -> 
Containers -> 
infrastructure ->
pgadmin-1 -> 
click the link "5050:80" -> 
username/password: admin@gmail.com/admin@gmail.com -> 
Servers 

  - Register a server

  right-click Servers -> 
  Register -> 
  Server... -> 
  General -> 
  Name: shopping-list-web-app -> 
  Connection -> 
  Host name/address: postgres -> 
  Port: 5432 -> 
  Maintenance database: postgres -> 
  Username: postgres -> 
  Password: postgres -> 
  Save password? Yes

  - Connect to the server
  
  shopping-list-db -> 
  databases -> 
  shoppingList -> 
  schemas -> 
  public -> 
  tables -> 
  shopping_item -> 
  right-click -> 
  "View/Edit Data" -> 
  All Rows

- How to configure data source in intellij idea

  - Install JPA Buddy

  - JPA Structure (on lower left corner) -> 
    right-click Data connections -> 
    New -> 
    Detect Connections -> 
    Password: postgres -> 
    Test Connection -> 
    OK

- Start the web application

go to `back-end` folder

```shell
sdk use java 17.0.10-tem
./gradlew bootRun
```

- Check the API using swagger-ui

http://localhost:8081/swagger-ui.heml -> GET /api/v1/shopping-items


#### 2.2.5 How to check api documentation

http://localhost:8081/swagger-ui.html

#### 2.2.6 How to check API functions in a tool

- Install Insomnia (https://insomnia.rest/)
- Run database and db admin: ```docker compose up -d```
- Run the application: ```.\gradlew.bat bootRun```
- Open Insomnia -> 

  add a collection -> 
  add POST request ->
    createShoppingItem
      POST: localhost:8081/api/v1/shopping-items ->
      JSON: {"title": "milk", "purchased": false} ->
      Send
  add GET request ->
    getAll
      GET: localhost:8081/api/v1/shopping-items ->
      Send
    getById
      GET: localhost:8081/api/v1/shopping-items/1 ->
      Send
  add PUT request -> 
    updateShoppingItem
      PUT: localhost:8081/api/v1/shopping-items/1 ->
      JSON: {"title": "milk", "purchased": true} ->
      Send
  add DELETE request ->
    deleteShoppingItem
      DELETE: localhost:8081/api/v1/shopping-items/1 ->
      Send

### 2.3 front-end setup

go to `front-end` folder

- Install node.js and npm using nvm

https://github.com/nvm-sh/nvm

```shell
nvm -v
nvm ls-remote
nvm install iojs-v3.3.1
nvm ls
node -v # v3.3.1
npm -v # 2.14.3
```

- generate scaffold project using create-vue

https://github.com/vuejs/create-vue

```shell
npm create vue@latest
cd front-end
vue -V
npm install
npm run format
npm run dev
```

- Install vue-axios for using Axios conveniently to make HTTP requests to back-end server or API

https://www.npmjs.com/package/vue-axios

```shell
npm install --save axios vue-axios
npm list
```

- Install element-plus for high-quality components like buttons, tables, dialogs, etc.

https://element-plus.org/en-US/guide/installation.html#version

```shell
npm install element-plus --save
npm list
npm info element-plus
```

#### 2.3.1 Front-end IDE Setup

- Install WebStorm

#### 2.3.2 Project setup to install all dependencies
```
npm install
```

#### 2.3.3 How to run the vue.js app - Compiles and hot-reloads the front-end app for development
```
npm run dev
```

Visit the app from localhost or another machine

http://localhost:8080/

http://192.168.31.180:8080

#### 2.3.4 Compiles and minifies for production

```
npm run build
```

#### 2.3.5 Customize configuration

See [Configuration Reference](https://cli.vuejs.org/config/).


#### 2.3.6 How to run the back-end application in a container

```
# NOTE: You need to generate docker image on ubuntu or windows 10/11, not on apple m1, to generate an amd64 image, instead of arm64 image

# start postgres database and pgadmin using docker compose for running tests during gradle build
cd infrastructure
docker compose up postgres pgadmin

# generate back-end app image
cd back-end
./gradlew clean build
docker buildx build --build-arg JAR_FILE=build/libs/shoppinglist-0.0.1-SNAPSHOT.jar -t wubin28/shopping-list-api:v1.0.docker-compose.amd64 .

# start 3 containers: postgres, pgadmin, shopping-list-api
cd infrastructure
docker compose up -d postgres pgamdin shopping-list-api

# Check the API using Insomnia
```

#### 2.3.7 How to run the front-end application in a container

```
# NOTE: You need to generate docker image on ubuntu or windows 10/11, not on apple m1, to generate an amd64 image, instead of arm64 image

# generate front-end app image
cd front-end
docker buildx build  -t wubin28/shopping-list-front-end:v1.0.docker-compose.amd64 .

# start 4 containers: postgres, pgadmin, shopping-list-api, shopping-list-front-end
cd infrastructure
docker compose up -d

# Check the API using front end
http://localhost:8080/
```

### 2.4 How to push back-end and front-end images to hub.docker.com

```
docker login
docker push wubin28/shopping-list-api:v1.0.docker-compose.amd64
docker push wubin28/shopping-list-front-end:v1.0.docker-compose.amd64
```

### 2.5 How to pull back-end and front-end images from hub.docker.com

```
docker login
docker pull wubin28/shopping-list-api:v1.0.docker-compose.amd64
docker pull wubin28/shopping-list-front-end:v1.0.docker-compose.amd64
```

### 2.6 How to demo chaos engineering for dependency outage and delay

* Steady state hypothesis for dependency outage: The front end app should show a message "Could not connect to the back end app" when the back end app is down.

* Steady state hypothesis for dependency delay: The front end app should show a message "The response from the back end was delayed for over 3 seconds" when the back end app delays for over 3 seconds.

* Steady state hypothesis is not supported

```
# hypothesis not supported for dependency outage: see error in chrome console only
git co chaos-pumba-tc
cd infrastructure
docker compose down
docker compose up
http://localhost:8080/
docker container ls
pumba_linux_amd64 kill infrastructure-shopping-list-api-1

# hypothesis not supported for dependency delay: not see anything in web page and console
git co chaos-pumba-tc
cd infrastructure
docker compose down
docker compose up
http://localhost:8080/
docker container ls
pumba_linux_amd64 netem --duration 1m delay --time 4000 infrastructure-shopping-list-api-1
```

* Steady state hypothesis is supported

```
# hypothesis supported for dependency outage: see user-friendly message "Could not connect to the back end app"
git co chaos-pumba-dependency-outage
cd infrastructure
docker compose down
docker compose up
http://localhost:8080/
docker container ls
pumba_linux_amd64 kill infrastructure-shopping-list-api-1

# hypothesis supported for dependency delay: see user-friendly message "The response from the back end was delayed for over 3 seconds"
git co chaos-pumba-dependency-delay
cd infrastructure
docker compose down
docker compose up
http://localhost:8080/
docker container ls
pumba_linux_amd64 netem --duration 1m delay --time 4000 infrastructure-shopping-list-api-1
```
