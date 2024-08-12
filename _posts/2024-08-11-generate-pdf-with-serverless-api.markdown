---
layout: single
title:  "Generate pdf with Serverless API - AWS Lambda Function using Puppeteer"
date:   2024-08-11 07:14:38 -0600
categories: aws
---

![Cover](/images/posts/generate-pdf-with-serverless-api/cover.png)

I started with `PDFKit` and `jsPDF` to generate pdf but it became difficult with complex layouts. `jsPDF` supports generating a PDF from an HTML element or string but it does not work in node environment. 

Finally, I ended up using `puppeteer` to generate a PDF page from an HTML string. I created a simple [code](https://github.com/haku-d/html-2-pdf) using `Nestjs` framework, including a `POST` endpoint which will generate a PDF from HTML string input.

Recently, I found [Google Chrome for AWS Lambda as a layer](https://github.com/shelfio/chrome-aws-lambda-layer). That means I can setup a serverless api with `Lambda` running `puppeteer`. **THAT IS ALL WE NEED!**.

You can check out the set up with AWS SAM CLI on my github repo: [https://github.com/haku-d/html-2-pdf-lambda](https://github.com/haku-d/html-2-pdf-lambda)

---