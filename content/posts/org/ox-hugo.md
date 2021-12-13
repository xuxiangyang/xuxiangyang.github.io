+++
title = "使用Org写博客"
author = ["徐向阳"]
date = 2021-08-10T19:24:00+08:00
tags = ["hugo"]
categories = ["org"]
draft = false
+++

## 为什么使用Org而不直接使用Github Pages Markdown {#为什么使用org而不直接使用github-pages-markdown}

1.  我是Emacs用户，使用Spacemacs，日常开发工作主要使用这个工具，并且Spacemacs对于Org支持度很好
2.  [Org](https://orgmode.org/org.html) 基本语法表达能力不弱于标准 [Markdown](https://www.markdownguide.org/basic-syntax/)。
3.  [Org Capture](https://orgmode.org/org.html#Capture-and-Attachments) + [Agenda Views ](https://orgmode.org/manual/Agenda-Views.html#Agenda-Views)可以实现我想在任意的Emacs窗口随时管理我的 TODO、笔记等，自然的我希望博客也可以在这套体系下，关于这套多提。关于这套多提几句，它主要可以实现我如下模型
    -   在任意的Emacs任意界面随时记录和查看TODO，随意记录后Emacs会自动帮你汇总，这样可以最大限度的降低工作中的打扰
    -   Capture 可以自定义Template，可以满足我在不同场景下预制结构
    -   用Capture和直接自己去编辑各个org文件是非常不同的感受，你永远不用操心写到一半的东西要保存到哪里，如果找到你写了一半的东西
4.  我希望能有更多的控制，并且这些控制能充分利用前人经验，Github Page只是暂时的一个方案，它满足目前需求（毕竟我还没有写博客，我希望自己能定期有至少一次的深入思考），但灵活性就会有所下降，我不希望未来整个网站和Github必须绑死。


## 如何选取 Org 博客方案 {#如何选取-org-博客方案}

我参考了 [Blogs and Wikis with Org](https://orgmode.org/worg/org-blog-wiki.html) 最终选择了 `ox-hugo` ，主要考虑了如下几点：

1.  有丰富主题可供选择，我希望找一个简单的但是还算是有过设计的主题
2.  使用的人要多，而且仍要在维护，这样踩坑就少
3.  文档丰富，使用起来简单

基于以上几点过滤完，其实剩下的考虑就是在 `Jekyll` 和 `Hugo` 间了，因为以前听说过Hugo，所以就随意选择了Hugo方向了。通过尝试原生 org 感觉起来支持度不能满足我需求，并且没有如何天然配合Capture使用，所以 `ox-hugo` 就成了最终方案。当然，事实证明 `ox-hugo` 就恰恰是我想要的


## 如何搭建起整个博客 {#如何搭建起整个博客}

1.  在Spacemacs中添加 `org` layer，并且启用 `org-enable-hugo-support` 之后重启Spacemacs

    ```emacs-lisp
    (defun dotspacemacs/layers ()
      (setq-default
       dotspacemacs-configuration-layers
       '(
         (org :variables
              org-enable-github-support t
              )
         )
       )
    )
    ```
2.  安装 [Hugo](https://gohugo.io/)，已Mac为例

    ```bash
    brew install hugo
    ```
3.  使用Hugo创建博客地址，假设创建在 `~/blog` 目录下

    ```bash
    hugo new site blog
    ```
4.  [可选] 根据上一步命令行输出可以配置 `Theme` 等信息
5.  为 Org Capture 增加 Blog 模板
    1.  [ox-hugo](https://ox-hugo.scripter.co/) 默认支持2中模式 `One post per Org subtree` 和 `One post per Org file` ，我选择了 `One post per Org subtree` 这样我的配置不需要各个文件都去配置，并且这样配置的 Capture 模板更简单些。Org本身的Subtree设计能够让我专注在某一篇文章中，并且暂时我的文章数肯定不会多到在同一个文件中出现什么性能问题，所以这个模式目前是合理的。
    2.  我的 org 目录在 `~/org` 下，我希望 blog 的 Org 文件和其他任务、笔记等隔离，创建了 `~/org/blog.org` 文件来专门做博客，添加如下开头

        ```org
        #+HUGO_BASE_DIR: ~/blog
        #+HUGO_SECTION: posts
        ```
    3.  为Captrue增加Blog模板

        ```emacs-lisp
        (defun dotspacemacs/user-config ()
          (with-eval-after-load 'org-capture
            (defun org-hugo-new-subtree-post-capture-template ()
              "Returns `org-capture' template string for new Hugo post.
        See `org-capture-templates' for more information."
              (let* ((title (read-from-minibuffer "Title: ")) ;Prompt to enter the post title
                     (fname (org-hugo-slug title)))
                (mapconcat #'identity
                           `(
                             ,(concat "* TODO " title)
                             ":PROPERTIES:"
                             ,(concat ":EXPORT_FILE_NAME: " fname)
                             ":END:"
                             "%?\n")          ;Place the cursor here finally
                           "\n")))
            (add-to-list 'org-capture-templates
                         '("b"                ;`org-capture' binding + h
                           "Blog"
                           entry
                           ;; It is assumed that below file is present in `org-directory'
                           ;; and that it has a "Blog Ideas" heading. It can even be a
                           ;; symlink pointing to the actual location of notes.org!
                           (file "blog.org")
                           (function org-hugo-new-subtree-post-capture-template)))
            )
          )
        ```
6.  尝试使用 Org Capture 写一篇文章，最终有一篇文章的Org文件内容如下

    ```text
    #+HUGO_BASE_DIR: ~/blog
    #+HUGO_SECTION: posts
    ​* TODO Test
      :PROPERTIES:
      :EXPORT_FILE_NAME: test
      :EXPORT_HUGO_BUNDLE: org
      :EXPORT_HUGO_TAGS: test
      :EXPORT_HUGO_CATEGORIES: org
      :END:
    ** 二级标题
    ```

    里面有一些propertie的意思如下

    -   `#+HUGO_BASE_DIR` 这个是对所有文章都生效的配置，指出 `blog` 这个 Hugo 项目的Path
    -   `EXPORT_FILE_NAME` 这个是文章mardown的文件名
    -   `EXPORT_HUGO_BUNDLE` 这个是文章所在的子目录，我希望不同 CATEGORIE 的文章可以按子目录划分
    -   `EXPORT_HUGO_TAGS` 文章标签，可以支持多个
    -   `EXPORT_HUGO_CATEGORIES` 文章类型，也支持多个，但是我希望一个文章属于一个类型，有多个标签

    上面的配置实际效果将是导出的 markdown 将存储在 `~/blog/content/posts/org/test.md` 上，最终文章的URL将是 `{HUGO_HOST}/posts/org/test`
7.  写作完成后要把 `Test` 处的 `TODO` ，通过 `C-c C-c` 标记为 `DONE` ，否则会认为这个文章为 `草稿` 状态，Hugo 不显示
8.  在 Org 文件中 `Test` 文章 SubTree 执行导出 `, e e H H`
9.  进入 blog 目录，执行 `hugo server` 就可以看到效果了
10. 进行到上述步骤 Org 和 Hugo之间的交互就通了，下面讲展示如何使用 GitHub Pages 显示 Hugo 编译出的内容，参考的是 [Hugo Host on Github](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
    1.  在Github 创建 你用户名的 repository，已我自己举例，我创建了一个 [xuxiangyang.github.io](https://github.com/xuxiangyang/xuxiangyang.github.io) 的项目，这个项目需要是Public（否则开通Page需要额外付费）
    2.  进入Blog项目，添加 `.github/workflows/gh-pages.yml` 文件，用Github Action 自动生成博客

        ```yaml
        name: github pages

        on:
          push:
            branches:
        ​      - main  # Set a branch to deploy
          pull_request:

        jobs:
          deploy:
            runs-on: ubuntu-20.04
            steps:
        ​      - uses: actions/checkout@v2
                with:
                  submodules: true  # Fetch Hugo themes (true OR recursive)
                  fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

              - name: Setup Hugo
                uses: peaceiris/actions-hugo@v2
                with:
                  hugo-version: 'latest'
                  # extended: true

              - name: Build
                run: hugo --minify

              - name: Deploy
                uses: peaceiris/actions-gh-pages@v3
                if: github.ref == 'refs/heads/main'
                with:
                  github_token: ${{ secrets.GITHUB_TOKEN }}
                  publish_dir: ./public
        ```
    3.  为 Blog 项目添加Git托管

        ```bash
        git init
        git add .
        git commit -m 'commit with first test post'
        git branch -M main
        git remote add origin git@github.com:xuxiangyang/test.git
        git push -u origin main
        ```
    4.  在Github看这个项目的Actions，等待Action运行成功，这个Action的原理是利用 `main` 分支的 Hugo 源码，编译成HTML静态文件，合并到=gh-pages=分支
    5.  在Github上 Settings -> Pages，修改 Source Branch为 `gh-pages`
    6.  等待一段时间应该就可以了