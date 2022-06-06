## Dockerfile for Angular

You can use the Dockerfile to build an Angular app.

Follow the steps below to build an Angular app:

#### 1. Create a nginx configuration file

My default nginx configuration file looks as follows (it’s located in the same directory as Dockerfile):

```nginx.config
events{}
http {
    include /etc/nginx/mime.types;
    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;
        index index.html;
        location / {
            try_files $uri $uri/ /index.html;
        }
    }
}
```

In general, here we define the server on which application will be hosted, its port and default behaviour.

More detail about the configuration file can be found in the 
[nginx documentation](https://nginx.org/en/docs/http/ngx_http_core_module.html#include).

#### 2. Create a Dockerfile file

We’ll divide our Docker build into _two_ stages:
1. Compiling the source code into production ready output (builder stage)
2. Running compiled app in a nginx server docker image (runner stage)

Only compiled output from first stage will be moved to the second so small size of the container will be preserved.

```Dockerfile
FROM node:latest as builder
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm install

COPY . .
RUN npm run build

FROM nginx:latest as runner
COPY nginx.config /etc/nginx/nginx.conf
COPY --from=builder /app/dist /usr/share/nginx/html
```


#### 3. (Optional) Create a dockerignore file

To make building our Docker image even more efficient we can add to the project’s root additional file called `.dockerignore`.

In our case we don’t want to copy any files from _node_modules_ and _dist_ folders, because they’re not needed in compilation.

```ignore
node_modules
dist
```

#### 4. Build the image

Run the following command to build the image:

```bash
docker build -t <image_name> .
```

Then, you can check the image with the following command: `docker images`.

#### 5. Run the image to create a container and run the app

To run the image, you can use the following command:

```bash
docker run --name <container_name> -d -p <port>:80 <image_name>
```

With it first you give a name to the container (`--name <container_name>`), 
then make sure that it will run in the background (`-d`), 
next you map container port to your local (`-p <port>:80`) 
and finally you pick a base Docker image to be that you’ve just created - `<image_name>`.

To check if new container is running in terminal type: `docker ps`. 
Or you can enter a web browser and go to http://localhost:<port>/.