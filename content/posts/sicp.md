---
title: "Sicp"
date: 2021-10-24T15:24:14+08:00
draft: false
---

## 使用整理

### emacs

#### 安装 geiser

> https://medium.com/@joshfeltonm/setting-up-emacs-for-sicp-from-scratch-daa6473885c5

```lisp
(require 'package)
(add-to-list 'package-archives '("melpa" . "https://melpa.org/packages") t)
(package-initialize)
(package-refresh-contents)

(setq geiser-mit-binary "/usr/local/bin/scheme")
(setq geiser-active-implementations '(mit))
```

安装后可在 scm 源码文件处：

```lisp
M-x run-geiser RET
C-x o
C-x C-e ;; Eval sexp before point
```

更多请参见：https://www.nongnu.org/geiser/geiser_5.html

#### 配置 company

> http://company-mode.github.io/

```lisp
(add-hook 'after-init-hook 'global-company-mode)
```

安装：

```li
M-x package-install RET company RET
```

## spacemacs

> https://develop.spacemacs.org/layers/+lang/scheme/README.html



