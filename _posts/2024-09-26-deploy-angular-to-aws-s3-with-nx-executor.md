---
layout: single
title:  "Deploy angular application to aws s3 with nx executors"
date:   2024-09-26 10:24:38 -0600
categories: angular aws s3
---

I would love to share the package that I have used to build & deploy my angular application to AWS S3. The source code was inspired by `@jefiozie/ngx-aws-deploy`. Since my current application is built in an nx monorepo, I decided to create an nx executor along with the application. 

You can found all the source code in folder tools/builder and follow my instructions in this post to configure it on your project.

## 1. Create an nx executor

Run this command to create new library

`npx nx g @nx/angular:library builder --directory=tools/builder`

> Feel free to modify the library name (builder) and its directory(tools/builder)

## 2. Get the source code

Go to `https://github.com/haku-d/nx-aws-deploy` and copy all source files inside folder `tools/builder` to your project.

## 3. Integrate the executor into your application

Open `project.json` in your app folder, add new `deploy` target as below:

Replace `hello-bucket` by your desire bucket name
Replace `<your-cloud-front-distribution-id>` by your cloudfront distribution id

```
{
  "name": "hello",
  "$schema": "../node_modules/nx/schemas/project-schema.json",
  "projectType": "application",
  "prefix": "app",
  "sourceRoot": "hello/src",
  "tags": [],
  "targets": {
    // other target
    "deploy": {
      "executor": "@ng-mf/builder:deploy",
      "options": {
        "bucket": "hello-bucket",
        "cfDistributionId": "<your-cloud-front-distribution-id>",
        "globFileUploadParamsList": [
          {
            "glob": "*",
            "ACL": "public-read",
            "CacheControl": "max-age=3600"
          },
          {
            "glob": "*.html",
            "CacheControl": "max-age=300"
          }
        ]
      }
    }
  }
}
```

## 4: Deploy your application

```
npx cross-env NG_DEPLOY_AWS_ACCESS_KEY_ID=1234 NG_DEPLOY_AWS_SECRET_ACCESS_KEY=321ACCESS NG_DEPLOY_AWS_REGION=ca-central-1 nx run hello:deploy
```



