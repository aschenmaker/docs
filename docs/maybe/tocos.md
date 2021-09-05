# 使用github action 同时部署到 Github Pages 和 腾讯云COS

在`.github/workflows`创建Action文件
## 部署到Github Pages
创建pulish_gh.yml
```yml
name: Publish docs via GitHub Pages
on:
  push:
    branches:
      - master

jobs:
  build:
    name: Deploy docs
    runs-on: ubuntu-latest
    steps:
      - name: Checkout main
        uses: actions/checkout@v2

      - name: Deploy docs
        uses: mhausenblas/mkdocs-deploy-gh-pages@master
        env:
          GITHUB_TOKEN: ${{ secrets.GH_TOKEN }} # 需要获取github-token
          CONFIG_FILE: mkdocs.yml
          EXTRA_PACKAGES: build-base
          CUSTOM_DOMAIN: docs.cqs.es # 如果自有域名则可设置
```

## 部署到Tencent COS
好处是快，但是需要开通对象存储服务，以及需要已经备案的域名。
同样创建 `pulish_cos.yml`
```yml
name: Deploy to tencent osss
on:
  push:
    branches:
      - master

jobs:
  build:
    name: Deploy docs
    runs-on: ubuntu-latest
    steps:
      - name: Checkout main
        uses: actions/checkout@v2

      - name: Deploy to tencent osss
        uses: aschenmaker/mkdocs-deploy-tencentcos@master
        env:
          CONFIG_FILE: mkdocs.yml
          SECRET_ID: ${{ secrets.SECRET_ID }}
          SECRET_KEY: ${{ secrets.SECRET_KEY }}
          BUCKET: ${{ secrets.BUCKET }}
          REGION: ap-nanjing
```
需要腾讯云对应的API SECRET，最好通过创建子用户设置到github仓库的secret中。
细节查看: 访问[设置Github action](https://github.com/marketplace/actions/deploy-mkdocs-to-tencent-cos)

