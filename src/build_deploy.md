# 搭建博客的过程记录

之所以会搭建这个博客，是希望自己学习一下 mdbook，gh-page 以及 git Action 的相关用法。

目前的目标是在一个 github Organization 中读取所有仓库的内容（具体会是一个个文献翻译仓库），并组织成一个总的文献翻译在线页面。

##### 一些参考的资料
+ 直接参考的是 [链接](https://navy-to-haijun.github.io/haijun-note/studying/%E5%9F%BA%E4%BA%8Emdbook%E5%92%8Cgithub-pages%E9%83%A8%E7%BD%B2%E4%B8%AA%E4%BA%BA%E7%BD%91%E7%AB%99.html) 

还有一些看过的
+ mdbook 使用: https://www.aye10032.com/2023/09/12/2023-09-12-mdbook/
+ github ci 介绍: https://zhuanlan.zhihu.com/p/250534172
+ stater-workflows 一些比较简单的脚本例子: https://github.com/actions/starter-workflows/tree/main
    + 其中 mdbook 的自动部署 Action 脚本:https://github.com/actions/starter-workflows/blob/main/pages/mdbook.yml
+ 使用 python 编写的脚本，用于生成 mdbook 的 SUMMARY.md 目录文件: https://github.com/lzzsG/mdBook-tools/tree/main/mdBook-directory-summary-generator


##### 希望爬取 github group 中的仓库
+ 学习了一下操作系统训练营 rCore 排行榜的自动更新
    + 更新的过程会使用 GitHub API 和 classroom 
    + 需要获取组织中 Owner 成员的 Token，赋值给排行榜仓库 action 的环境变量 AUTH_TOKEN

一个可以工作的 workflow 脚本，用于读取组织中所有仓库的名称及各个仓库的具体内容并输出出来。 [链接](https://github.com/cyman-paper-translation/main-page/blob/main/.github/workflows/build.yml)

```
name: Read Organization Repos

on:
  push:
    branches:
      - main  

jobs:
  read-repos:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      
      - name: Install Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'

      - name: List Repositories
        id: list-repos
        run: |
          AUTH_TOKEN="${{ secrets.AUTH_TOKEN }}"
          ORGANIZATION="cyman-paper-translation"
          
          # 获取组织中的仓库列表
          REPOS=$(curl -s -H "Authorization: token $AUTH_TOKEN" \
                   "https://api.github.com/orgs/$ORGANIZATION/repos?per_page=100" | \
                   jq -r '.[].full_name')
          
          for REPO in $REPOS; do
            echo "Repository: $REPO"
            # 读取仓库内容
            CONTENT=$(curl -s -H "Authorization: token $AUTH_TOKEN" \
                      "https://api.github.com/repos/$REPO/contents")
            echo "$CONTENT"
          done
```

[执行结果](https://github.com/cyman-paper-translation/main-page/actions/runs/8751284950/job/24016562491#step:4:1)