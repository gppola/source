---
title: zsh/vim/tmux 配置
date: 2020-08-30 09:20:30
tags: Linux

---

## vim
```
set nocompatible              " be iMproved, required
set nu
set backspace=2
set ignorecase
set smartcase

syntax on

imap <C-n> <Down>
imap <C-p> <Up>
imap <C-@> <C-Space>

filetype off                  " required
" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'
Plugin 'tomtom/tcomment_vim'
Plugin 'itchyny/lightline.vim'
Plugin 'octol/vim-cpp-enhanced-highlight'
Plugin '907th/vim-auto-save'

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line

autocmd FileType cpp nmap <buffer> <F9> :w<bar>! g++ -W -std=c++0x -o %:t:r %:t && ./%:t:r <Cr>
autocmd FileType python nmap <buffer> <F9> :w<bar>! python %:t <Cr>
autocmd FileType cpp nmap <buffer> <F10> :w<bar>! g++ -W -std=c++0x -o %:t:r %:t && ./%:t:r < in<Cr>
let g:auto_save = 1  " enable AutoSave on Vim startup

```

## tmux
from https://github.com/gpakosz/.tmux
在.tmux.conf.local末尾添加
```
# move status line to top
#set -g status-position top
tmux_conf_theme_status_left='  #S | '
tmux_conf_theme_status_right='#{prefix}#{pairing}#{synchronized}, %R , %d %b |'
tmux_conf_theme_status_left_bg='#00afff,#00afff,#00afff'
set-option -g repeat-time 1
set-option -g status-position top
set-window-option -g mode-keys vi
```

## zsh

install zsh, oh-my-zsh

use custom theme `powerlevel9k/powerlevel9k`
https://github.com/Powerlevel9k/powerlevel9k/wiki/Install-Instructions#arch-linux

install plugin
````
git clone git://github.com/zsh-users/zsh-syntax-highlighting $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

加入插件 ~/.zshrc
```
# plugins=(git)
plugins=(
	zsh-syntax-highlighting
	zsh-autosuggestions
	autojump
)
```
