---
layout: single
title:  "Deploy Angular application using Docker"
date:   2024-08-13 10:24:38 -0600
categories: angular
---

While Angular comes with a number of third-party builders which implement deployment capabilities to different platforms such as Firebase, Vercel, Netlify, AWS S3, and Github pages. But this guide will walk you through how to deploy your angular application to a remote server using docker.

## Step 1: Get your angular application ready
Before you begin, make sure your angular application is ready for production.

## Step 2: Create the Dockerfile

```docker
FROM node:20-alpine as dist

WORKDIR /app

# Install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy application source code
COPY . .

# Build the application
ARG NODE_ENV=production
RUN npm run build --configuration=${NODE_ENV}

FROM nginx:alpine-slim

# Copy custom NGINX configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

WORKDIR /usr/share/nginx/html

COPY --from=dist app/dist/my-app/browser .

# Rename the main HTML file if necessary
RUN mv index.csr.html index.html
```

## Step 3: Create custom nginx configuration file 

```nginx
server {
  listen 80;
  sendfile on;
  default_type application/octet-stream;

  gzip on;
  gzip_http_version 1.1;
  gzip_disable      "MSIE [1-6]\.";
  gzip_min_length   256;
  gzip_vary         on;
  gzip_proxied      expired no-cache no-store private auth;
  gzip_types        text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;
  gzip_comp_level   9;

  root /usr/share/nginx/html;

  location / {
    try_files $uri /index.html =404;
  }

  location ~* \.(?:jpg|jpeg|gif|png)$ {
    try_files $uri /assets/No-Image-Placeholder.png;
  }
}
```

You might be found same guideline on the internet or by asking ChatGPT. But it did not tell you to create custom nginx configuration. Let's take a look at this case: once your application was deployed successfully and running on port **4200**. You open it on your favourite browser, click on any router link to access other pages, such as /profile. The full url will be `http://localhost:4200/profile`. 

If you used default nginx configuration for your application and you refresh the browser while opening `http://localhost:4200/profile`, nginx will show 404 page. But with this configuration, the nginx server will return the application's host page (index.html) when asked for a file that it does not have. All routes must fall back to index.html

## Step 4: Build the Docker Image

To build the Docker image, navigate to the directory containing your Dockerfile and run:

```
docker build -t my-angular-app .
```

This command will package your Angular application into a Docker image.

Step 5: Run the Docker Container
After the image is built, you can run it with the following command:

```
docker run -p 80:80 my-angular-app
```

This command starts an NGINX server inside a Docker container, serving your Angular application on `http://localhost`.

Thanks for your reading