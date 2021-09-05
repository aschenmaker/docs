# 或许你也需要这样的站点

> 十分钟快速搭建，个人知识管理网站

## 配置mkdoc
https://blog.keybrl.com/professional-2018-05-19-mkdocs-blog/

## 配置github action，自动部署
https://www.ixiqin.com/2021/01/how-to-use-a-lot-action-automatically-deploy-mkdocs/

## 附件
我的配置文件，供你参考
```yml
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

