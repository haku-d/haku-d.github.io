---
layout: single
title:  "Deploy angular monorepo to AWS S3 by github workflow"
date:   2024-10-12 07:14:38 -0600
categories: angular aws github s3 monorepo
---

The guideline is created for `Nx monorepo` workspace only to build and deploy affected apps to AWS S3 by github workflow.

Let's consider the below monorepo structure
.
└── YourMonoRepo/
    ├── apps/
    │   ├── app1
    │   └── app2
    └── libs/
        ├── lib1
        └── lib2

### The workflow will be triggered if there is any commit is made on branch `develop`.

```
name: CI-Deploy

on:
  push:
    branches:
      - develop
```

### Setup build job

```
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      app1_exist: ${{ steps.check_build_output1.outputs.app1_exist }}
      app2_exist: ${{ steps.check_build_output2.outputs.app2_exist }}
    steps:
      - name: 🛎️ Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: 📥 Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20  # Specify your Node.js version
          cache: 'npm'
      - name: 📦 Install dependencies
        run: npm ci
      - name: 🏗️ Build only affected apps
        run: npx nx affected -t build --base=develop~1 --head=develop
      - name: 🔍 Check if app1 build output folder exists
        id: check_build_output1
        if: ${{ hashFiles('./dist/apps/app1') != '' }}
        run: echo "app1_exist=true" >> "$GITHUB_OUTPUT"
      - name: 🔍 Check if app1 build output folder exists
        id: check_build_output2
        if: ${{ hashFiles('./dist/apps/app2') != '' }}
        run: echo "app2_exist=true" >> "$GITHUB_OUTPUT"
      - name: ⬆️ Upload app1 artifact
        if: steps.check_build_output.outputs.app1_exist == 'true'
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/apps/app1
      - name: ⬆️ Upload app2 artifact
        if: steps.check_build_output.outputs.app2_exist == 'true'
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/apps/app2
```

#### Steps

1. Checkout the source code
2. Setup node.js env
3. Install dependencies
4. Build affected apps
5. Check if output folder of `app1` is existed
6. Check if output folder of `app2` is existed
7. Upload artifacts

---