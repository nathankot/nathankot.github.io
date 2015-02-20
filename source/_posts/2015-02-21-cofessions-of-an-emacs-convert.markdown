---
layout: post
title: "7 Confessions of an Emacs convert coming from vim"
date: 2015-02-21 10:30:20 +1300
comments: true
categories: vim emacs
---

## Emacs needs a new icon

Seriously the logo for emacs sucks. If it hadn't, I might've tried Emacs a long
time ago. I imagine a lot of people are in the same boat here, there are
[many][replacement1] [replacement][replacement2] icons out there!

## Evil-mode is just as good as vim, if not better

If you come from vim, [evil-mode][evil] is a must have. If you're expecting
half-assed vim emulation then you will be disappointed. Evil-mode can do
everything vim can. Sometimes even more: remember [vim-repeat][vim-repeat]? Well
repeating evil-mode plugin commands just works, no more unexpected repeat behavior!

## Smoother, faster, more stable

Apart from startup time, everything just feels so much nicer compared to a vim
overloaded with plugins (I make the distinction because vanilla vim is obviously
going to be very crisp.) Startup time can be improved drastically by
[lazy-loading plugins][use-package] anyway.

## The emacs community is small, but that's okay

Vim has won the popularity contest - undisputed. No problem - whilst the
vim community needs masses of people maintaining and fixing it's plugins, most
emacs plugins are very stable due to the editor's better design choices and [language][elisp].

## Waiting on Neovim? Give emacs a try

That's why I did. After spending some time playing around with [Neovim][neovim]
I realized that it's far from stable, and the transition would be a pain in the
ass anyway. Emacs has already solved a lot of the problems Neovim is trying to
solve. And when Neovim _does_ become a headless text editor, Emacs can take
advantage of that as well ;)

## Great excuse to learn Lisp

I'm still not a _huge_ fan of the lisp family, but I can see it's
merits. Nevertheless I would pick elisp over vimscript any day. 

## I wish Vim was this good

Imagine the design choices of Emacs coupled with the relentless community of
Vim, damn. Perhaps Neovim can realize this?

_Want to try emacs? [Read this awesome pictogram first =D][how-to-learn].

[replacement1]: http://jasonm23.github.io/
[replacement2]: http://emacs.sexy/
[evil]: http://www.emacswiki.org/emacs/Evil
[vim-repeat]: https://github.com/tpope/vim-repeat
[elisp]: https://www.gnu.org/software/emacs/manual/html_node/elisp/
[use-package]: https://github.com/jwiegley/use-package
[neovim]: http://neovim.org
[how-to-learn]: http://emacs.sexy/img/How-to-Learn-Emacs-v2-Large.png
