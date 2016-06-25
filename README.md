helm-git-files.el --- helm for git files
========================================

![M-x helm-git-files](https://gist.githubusercontent.com/tarao/57d4a6c2faf79b0782a3ed43c753bc24/raw/helm-git-files.png "M-x helm-git-files")

## Features

- List all the files in a git repository which the file of the current buffer belongs to
  - List untracked files and modified files in separate sources
- List files in the submodules
- Cache the list as long as the repository is not modified
- Avoid interrupting user inputs; external git commands are invoked asynchronously

## Install

### Manual Install

1. Install dependent package [helm](https://github.com/emacs-helm/helm)
2. Locate [helm-git-files.el](https://raw.github.com/tarao/helm-git-files-el/master/helm-git-files.el) in your load path

### Install by el-get

1. Setup [el-get](https://github.com/dimitri/el-get)
2. Put the following code in your Emacs init file (`~/.emacs` or `~/.emacs.d/init.el`)

```lisp
(el-get-bundle tarao/helm-git-files-el
  :depends helm)
```

## Usage

`M-x helm-git-files` will list the files in a git repository.  Note
that `M-x helm-git-files` will fail when the file of the current
buffer is not in a git repository.

## Customization

### Hack the helm sources

The helm sources to get files in a git repository are
`helm-git-files:modified-source`,
`helm-git-files:untracked-source` and `helm-git-files:all-source`.

The list of helm sources for submodules can be retrieved by function
`helm-git-files:submodule-sources`. The function takes one argument,
which is a list of symbols of source type, `modified`, `untracked` or
`all`. For example, `(helm-git-files:submodule-sources '(untracked
all)` returns helm sources for untracked files and all files in the
git repository of the submodules.

The following example defines a custom helm function to list files
from several sources, including ones from `helm-git-files.el`.

```lisp
(defun tarao/helm-for-files ()
  (interactive)
  (require 'helm-git-files)
  (unless helm-source-buffers-list
    (setq helm-source-buffers-list
          (helm-make-source "Buffers" 'helm-source-buffers)))
  (let* ((git-sources (and (helm-git-files:git-p)
                           `(helm-git-files:modified-source
                             helm-git-files:untracked-source
                             helm-git-files:all-source
                             ,@(helm-git-files:submodule-sources 'all))))
         (basic-sources '(helm-source-buffers-list))
         (other-sources '(helm-source-recentf
                          helm-source-bookmarks
                          helm-source-files-in-current-dir
                          helm-source-file-cache
                          helm-source-locate))
         (sources `(,@basic-sources
                    ,@git-sources
                    ,@other-sources)))
    (helm :sources sources
          :ff-transformer-show-only-basename t
          :buffer "*helm for files*")))
```

## Comparison to other packages

### [`helm-ls-git.el`](https://github.com/emacs-helm/helm-ls-git)

It provides some extra features such as a helm source of `git status`
but
[it lacks the ability to list files in submodules](https://github.com/emacs-helm/helm-ls-git/issues/19).

### [`helm-projectile.el`](https://github.com/bbatsov/helm-projectile)

It can list files not only in a git repository but any other kind of
VCS that [Projectile](https://github.com/bbatsov/projectile) supports.
The list includes files in submodules but the helm source is not
separated.  The invocation of `git ls-files` seems to be synchronous
since it is hidden inside the Projectile code.  This may cause a major
performance issue if you have a project with a large number of
submodules.

## History

The original version of this package was [`anything-git-files.el`](https://github.com/tarao/anything-git-files-el).
