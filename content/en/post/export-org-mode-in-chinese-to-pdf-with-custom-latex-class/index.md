+++
title = "Export Org-mode in Chinese to PDF with custom latex class"
date = 2020-04-20T18:50:00+08:00
lastmod = 2020-05-17T18:48:33+08:00
tags = ["emacs", "latex", "pdf"]
categories = ["Post"]
draft = false
toc = true
+++

A simple configuration to export org-mode doc in chinese to pdf with custom latex class, based on [redguardtoo/emacs.d](https://github.com/redguardtoo/emacs.d).

<!--more-->

Features are:

-   use [ElegantLaTeX/ElegantPaper](https://github.com/ElegantLaTeX/ElegantPaper) as default template;
-   use [gpoore/minted](https://github.com/gpoore/minted) to highlight code;
-   use `ctex` package's default font settings;

Download [elegantpaper.cls](https://github.com/ElegantLaTeX/ElegantPaper/blob/master/elegantpaper.cls) to the same directory contanis `org` files。

Add configs below to your  `~/.custom.el` ：

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

Add to the top of your `org` file：

```org
#+LATEX_COMPILER: xelatex
#+LATEX_CLASS: elegantpaper
#+OPTIONS: prop:t
```

Install dependencies of `minted` :

```bash
brew install pygments
```

Move cursor to the subtree to be exported, and press key `C-c C-e C-s l p` .

****Ref****

-   <http://orgmode.org/worg/org-faq.html#using-xelatex-for-pdf-export>
-   <https://orgmode.org/manual/Export-Settings.html#Export-settings>