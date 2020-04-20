+++
title = "Export Org-mode in Chinese to PDF with custom latex class"
author = ["chi"]
date = 2020-04-20T18:50:00+08:00
lastmod = 2020-04-20T18:57:14+08:00
tags = ["emacs", "latex", "pdf"]
categories = ["实用功"]
draft = false
toc = true
+++

因为疫情需要准备些远程面试题，在 Org-mode 里写好了之后想导出 PDF，先导出为 markdown，再用 pandoc 转为 PDF 搞定，然而还是想一步到位，节省时间。

目前在用的 emacs 配置基于 [redguardtoo/emacs.d](https://github.com/redguardtoo/emacs.d) 修改而来。

配置的默认特性有：

-   使用 [ElegantLaTeX/ElegantPaper](https://github.com/ElegantLaTeX/ElegantPaper) 作为默认的导出模板;
-   使用 [gpoore/minted](https://github.com/gpoore/minted) 进行代码高亮;
-   使用 ctex 默认的字体设置;

下载 [elegantpaper.cls](https://github.com/ElegantLaTeX/ElegantPaper/blob/master/elegantpaper.cls) 放到 \`org\` 文档同级目录内。

在 `~/.custom.el` 里添加配置：

```lisp
(with-eval-after-load 'ox-latex
 ;; http://orgmode.org/worg/org-faq.html#using-xelatex-for-pdf-export
 ;; latexmk runs pdflatex/xelatex (whatever is specified) multiple times
 ;; automatically to resolve the cross-references.
 (setq org-latex-pdf-process '("latexmk -xelatex -quiet -shell-escape -f %f"))
 (add-to-list 'org-latex-classes
               '("elegantpaper"
                 "\\documentclass[lang=cn]{elegantpaper}
                 [NO-DEFAULT-PACKAGES]
                 [PACKAGES]
                 [EXTRA]"
                 ("\\section{%s}" . "\\section*{%s}")
                 ("\\subsection{%s}" . "\\subsection*{%s}")
                 ("\\subsubsection{%s}" . "\\subsubsection*{%s}")
                 ("\\paragraph{%s}" . "\\paragraph*{%s}")
                 ("\\subparagraph{%s}" . "\\subparagraph*{%s}")))
  (setq org-latex-listings 'minted)
  (add-to-list 'org-latex-packages-alist '("" "minted")))
```

在 \`org\` 文档的头部添加参数：

```org
#+LATEX_COMPILER: xelatex
#+LATEX_CLASS: elegantpaper
#+OPTIONS: prop:t
```

安装 `minted` 的依赖:

```bash
brew install pygments
```

之后将光标移动到要导出的 Heading, `C-c C-e C-s l p` 即可。

****参考****

-   <http://orgmode.org/worg/org-faq.html#using-xelatex-for-pdf-export>
-   <https://orgmode.org/manual/Export-Settings.html#Export-settings>