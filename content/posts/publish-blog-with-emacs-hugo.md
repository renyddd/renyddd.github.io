+++
title = "结合 Hugo 与 Emacs 管理发布博客"
author = ["renyddd"]
description = "记录对 Emacs Hugo 的配置过程"
date = 2022-05-08T17:30:00+08:00
tags = ["emacs", "hugo"]
categories = ["Emacs", "Hugo"]
draft = false
+++

## 使用概览 {#使用概览}

此配置主要由两个包组成：

1.  [ox-hugo](https://github.com/kaushalmodi/ox-hugo)：将 org 文件导出为 markdown 格式的导出器，如将当前文档导出

    ```elisp
    C-c C-e H h
    ```

{{< figure src="http://pic.renyddd.top/ox-hugo.png" >}}

1.  [easy-hugo](https://github.com/masasam/emacs-easy-hugo)：通过其 major mode 完成与 hugo 二进制文件的交互；

    ```elisp
    M-x easy-hugo
    ```

{{< figure src="http://pic.renyddd.top/easy-hugo.png" >}}


## Emacs 配置文件 {#emacs-配置文件}

根据如上文档安装相关包，这里不在赘述；其中有一点是 tomelr package not found 问题，解决方法也如下所示：

```elisp
;; 解决下面的 tomelr 包下载错误，先指定 github 下载源
;; (error ("Could not find package tomelr. Updating recipe repositories: (org-elpa melpa gnu-elpa-mirror el-get ..."))
(straight-use-package '(tomelr
			 :type git :host github :repo "kaushalmodi/tomelr"
			 ))

(straight-use-package 'ox-hugo)

(with-eval-after-load 'ox
   (require 'ox-hugo))
 (setq org-hugo-base-dir "~/your/hugo/path"
	   org-hugo-default-section-directory "posts") ; 具体可参考 HUGO/content 下目录
```

这里欢迎查看我的全部 [Emacs 配置](https://github.com/renyddd/DotConfigurations/tree/main/.emacs.d)。
