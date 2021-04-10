---
layout: post
title: "My Neovim Development Setup"
description: "An update to my last neovim guide"
date: 2018-04-12
tags: [neovim, vim, deoplete, dein, cquery, language, server, python, c, c++]
comments: true
---

It's been a while since I wrote about my Neovim setup. Since my last post, my
`nvim` config has grown to be a little more sophisticated, and I finally worked
out autocompletion and linting for all of the languages I work with.

Here's what my editor looks like:

![neovim rust](/images/nvim_1.png)

I have posted my full neovim configuration on
[Github](https://github.com/afnanenayet/nvim-dotfiles)

## Split up your `init.vim`

I had a horribly long `init.vim` file before. It gets clunky to manage and long
files are ugly. It's highly recommended that you split up your `init.vim` files
into more manageable chunks. The way you can do this is by sourcing each chunk
in your main init.vim file. Suppose we have a file for our deoplete settings,
and another file for our language client settings. Our `init.vim` file could
look something like this:

```vim
source $HOME/.config/nvim/config/deoplete.vim
source $HOME/.config/nvim/config/lc.vim
```

And in `config/lc.vim` and `config/deoplete.vim`, you could have whatever
settings pertaining to each plugin that you want. I personally have my config
files split up in the following manner:

```
config
├── 01.plugins.vim
├── 02.init.vim
├── 03.powerline.vim
├── 04.deoplete.vim
├── 05.language_client.vim
└── 06.keybindings.vim
```

My `init.vim` file looks like this:

```vim
for f in split(glob('~/.config/nvim/config/*.vim'), '\n')
    exe 'source' f
endfor
```

_Credit: Thanks to Igor Epstein for pointing out I can use a regex to grab
multiple files in a directory and clean up my init.vim config._

Note that I had to put the `plugins.vim` file first because it contains
everything pertaining to my package manager, dein, and dein has some scripts it
needs to run first in Neovim. The order that you put these scripts in does
matter, as they effectively will be set up in order.  Everything in
`plugins.vim` will be run before everything in the other files in my case.
`init.vim` files can get messy, and this offers a way to clean up your setup
and separate your neovim/vim config into logical units.

## Managing packages

There are a lot of package managers for Neovim/vim out there. People have
different preferences, I have no strong feelings one way or the other. I
personally use [dein](https://github.com/Shougo/dein.vim), which has served me
pretty well. It uses all of the great async features in Vim 8/Neovim and is
pretty easy to use.

My `plugins.vim` file has all of the settings pertaining to dein, as well as
the dein startup scripts that it needs to run when vim/nvim boots up.

After setting it up, installing plugins is as easy as:

```vim
call dein#install("github-username/repo-name")
```

Note that it doesn't automatically update plugins by default. I manually update
my plugins. You can just call `:call dein#update()` in Neovim, and dein will
upgrade all of your plugins for you.

## Autocompletion and Linting

I used to use [deoplete](https://github.com/Shougo/deoplete.nvim) for my
autocompletion needs. I ended up switching to
[ncm2](https://github.com/ncm2/ncm2) because I wanted to try out something new.
So far, I've had an identical experience with both engines, so I think you will
be fine regardless of which language client you use. I've heard excellent
things about [coc-nvim](https://github.com/neoclide/coc.nvim) (which stands for
"conqueror of completion." It aims to provide a fully fledged visual studio
code-esque editing experience for neovim. I personally think it's a really
heavy plugin, and whenever I feel that neovim doesn't suit my needs, I use
VSCode directly. However, if you don't want to use VSCode, and you want the
same quality of extensions, coc is the way to go, since VSCode has some quirks
and mistakes in its implementation of the language client spec, and many
language servers and plugins exclusively test against VSCode.

### Configuring Deoplete

_I don't use deoplete anymore, but I have left this part of my article up just
in case someone might find it useful. If this becomes out of date and ceases to
be useful, please let me know._

Deoplete is powered by completion sources, and it has a few default ones that
really annoyed me. For example, buffer based completion was fairly unhelpful
for me. I don't want autocompletion in a markdown or text file, and I only want
to have completion from meaningful symbols. You can set the completion sources
for deoplete, and also have completion enabled on a per-buffer basis.

I have deoplete completion disabled by default

```vim
" disable autocomplete by default
let b:deoplete_disable_auto_complete=1
let g:deoplete_disable_auto_complete=1
```

I also set sources to be empty by default

```vim
let g:deoplete#sources = {}
```

And lastly, I don't want any autocompletion when I'm writing strings and
comments.

```vim
" Disable the candidates in Comment/String syntaxes.
call deoplete#custom#source('_',
            \ 'disabled_syntaxes', ['Comment', 'String'])
```

You can set sources per language like this

```vim
call deoplete#custom#option('sources', {
    \ 'python': ['LanguageClient'],
\})
```

and replace `python` which whatever language(s) you want.

_Note: thanks to Steve Vermeulen for notifying me about the new convention for
setting language server settings due to updates from deoplete._

### Completion sources and language servers

If you want to use a language client, I would suggest using the
[LanguageClient-neovim](https://github.com/autozimu/LanguageClient-neovim)
plugin. Note that this source is called `LanguageClient`, so if you want to use
a language server for a language, you need to set it up for the LanguageClient
plugin, then set the source for that language to be `LanguageClient`. One thing
that tripped me up when trying to set up sources for deoplete was that I didn't
find good documentation for what each plugin is named. For example,
`deoplete-rust` uses `racer`, but the name of the source is `rust`, but
`deoplete-jedi` has a source called `jedi`.

Here are some sources/language servers you can use (I've included the names of
the sources for entries that aren't language servers).

- python:
  - [deoplete-jedi](https://github.com/zchee/deoplete-jedi) (source) `jedi`
  - [pyls](https://github.com/palantir/python-language-server) (language server)
- rust:
  - [deoplete-rust](https://github.com/sebastianmarkow/deoplete-rust) (source) `rust`
  - [rls](https://github.com/rust-lang-nursery/rls/) (language server)
- C/C++:
  - [deoplete-clangx](https://github.com/Shougo/deoplete-clangx) (source) `clangx`
  - [ccls](https://github.com/MaskRay/ccls) (language server) note
    that I've found this to be the most effective solution for large projects.
    `ccls` is a fork of `cquery`, which is no longer being actively developed.
  - [clangd](https://clang.llvm.org/extra/clangd.html) (language server)

I only use language servers for development (except when editing vim files).

## Other plugins

Some of my other favorite plugins are:

- [NERD tree](https://github.com/scrooloose/nerdtree): a nice hierarchical file
- browser
- [fzy](https://github.com/jhawthorn/fzy): a fast fuzzy file finder I use as to
- search for files and for text within files. You can use this as a replacement
  for the `Ctrl+P` plugin
- [vim airline](https://github.com/vim-airline/vim-airline/): a lean powerline
  plugin for vim
