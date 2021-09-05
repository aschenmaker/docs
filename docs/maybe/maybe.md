# 或许你也需要这样的站点

!!! abstract "十分钟快速搭建"
    1. 配置mkdocs
    2. 持续集成&部署
        3. 部署到服务器
        1. 部署到GitHub Pages
        2. 部署到对象存储
        

## 配置mkdoc
参考这篇文章，写的很详细，具体的内容我也不写了。

介绍了如何使用以及配置主题。

https://blog.keybrl.com/professional-2018-05-19-mkdocs-blog/

## 自动部署
参考这一篇文章，提供了自动部署到GitHub Pages方法。

https://www.ixiqin.com/2021/01/how-to-use-a-lot-action-automatically-deploy-mkdocs/


## 附件
到这里你已经拥有了这样一个网站，并且可以通过Github Action自动部署。

同时你也可以参考我的配置来设置你喜欢的样式，添加你需要的功能。

见以下配置文件[^1]

```yaml
site_name: Qiu's doc space
site_author: Qiu(aschenmaker)
site_url: https://docs.cqs.es
# Repository
repo_name: 'aschenmaker/docs'
repo_url: 'https://github.com/aschenmaker/docs'
edit_uri: 'blob/master/docs/'

# Copyright
copyright: 'Copyright &copy; 2021 - 2022 aschenmaker'


theme:
  name: 'material'
  language: 'zh'
  palette:
    - scheme: default
      primary: indigo
      accent: indigo
      toggle:
        icon: material/toggle-switch-off-outline
        name: Switch to dark mode
    - scheme: slate
      primary: blue
      accent: blue
      toggle:
        icon: material/toggle-switch
        name: Switch to light mode
  icon:
    logo: 'material/cloud'
  features:
    - navigation.sections
    - navigation.tabs
    - navigation.top
    - navigation.tracking
    - tabs
  font:
    text: 'Fira Sans'
    code: 'Fira Mono'
remote_branch: gh-pages
remote_name: origin

extra:
  social:
    - icon: fontawesome/brands/github
      link: https://github.com/aschenmaker
  search:
    lang: 
      - zh-CN
    tokenizer: '[\s\-\.]+'
  disqus: 'costcost'
  copyright: 'CC BY-NC-SA 4.0'

extra_javascript:
  - 'https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-MML-AM_CHTML'

markdown_extensions:
  - admonition                                   # 注解块支持
  - pymdownx.arithmatex                          # 数学公式的TeX语法支持
  - pymdownx.betterem:
      smart_enable: all
  - pymdownx.caret
  - pymdownx.critic
  - pymdownx.details
  - pymdownx.emoji:                              # 表情支持
      emoji_generator: !!python/name:pymdownx.emoji.to_svg
  - pymdownx.inlinehilite
  - pymdownx.magiclink
  - pymdownx.mark
  - pymdownx.smartsymbols
  - pymdownx.superfences
  - pymdownx.tasklist:                           # 任务清单支持
      custom_checkbox: true
  - pymdownx.tilde
  - meta  
```


[^1]: 可能不是最新的如果需要则查看我的github仓库，更多功能访问主题对应的文档可以获得。