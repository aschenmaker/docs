# 使用Github action 同时部署到 Github Pages 和 腾讯云COS


在`.github/workflows`创建Action文件
## 部署到Github Pages
创建`pulish_gh.yml`
```yaml
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

!!! tip "解决访问速度慢的问题" 
    Github Pages方便易用，但是访问速度非常慢。
    通常的解决方法是

    - 通过CDN加速源站的方式，使用GitHub提供的IP对网站进行加速。
    - 直接将静态文件推送腾讯云COS或者阿里云OSS上，使用静态网站访问。


好处是快，但是需要开通对象存储服务，以及需要已经备案的域名。

同样创建 `pulish_cos.yml`
```yaml
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
        uses: aschenmaker/mkdocs-deploy-tencentcos@v0.1.0-release
        env:
          CONFIG_FILE: mkdocs.yml
          SECRET_ID: ${{ secrets.SECRET_ID }}
          SECRET_KEY: ${{ secrets.SECRET_KEY }}
          BUCKET: ${{ secrets.BUCKET }}
          REGION: ap-nanjing
```


!!! note "配置该GitHub Action"
    需要腾讯云对应的API SECRET，为了确保安全性，最好通过创建子用户生成API SECRET，再设置到github仓库的secret中。

    具体[设置Github action](https://github.com/marketplace/actions/deploy-mkdocs-to-tencent-cos)
