---
title: ZSH Settings
# author: wrkholic84
date: 2020-02-14 00:00:00 +0900
categories: [Computer, Environments]
tags: [mac, zsh]
math: true
mermaid: true
# image:
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---
bash shell을 너무 오래 써오다보니, zsh로 조금 늦게 넘어오게 되었다. 난 bash가 좋은데..(익숙해서)

## Change shell to zsh
```console
The default interactive shell is now zsh.
To update your account to use zsh, please run `chsh -s /bin/zsh`.
for more details, please visit https://support.apple.com/kb/HT208050.
foo@bar:~$ chsh -s /bin/zsh
```

## OH MY ZSH 설치
```console
foo@bar:~$ sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## NPM 설정 복사
```console
-> ~ cat ~/.bash_profile >> ~/.zshrc
-> ~ source ~/.zshrc
```
npm 명령이 잘 실행되는지 확인.

## iTerm2 Colour Scheme 변경
원하는 걸로 찾아서 변경해봅시다.

## ZSH Plugin 설치
1. zsh-auto
```console
-> ~ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```
2. zsh-highlight
```console
-> ~ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

.zshrc 파일에서
```console
plugins=(
  git
  zsh-syntax-highlighting
  zsh-autosuggestions
)
```

## Powerline font 설치
[Download Meslo LG M Regular for Powerline](https://github.com/powerline/fonts/blob/master/Meslo%20Slashed/Meslo%20LG%20M%20Regular%20for%20Powerline.ttf
)

다운로드 후 설치.
사용하고자 하는 IDE 등에서 font-famliy에 `Meslo LG M Regular for Powerline` 입력

## ZSH theme 설치
.zshrc 파일에서
```console
ZSH_THEME="agnoster"
```
```console
-> ~ source ~/.zshrc
```

## VIM 꾸미기
하는김에..
```console
wrkholic84 > vi ~/.vimrc
set hlsearch
set ruler
set autoindent
set cindent
set nu
set ts=4
set shiftwidth=4
syntax on
```

끝.