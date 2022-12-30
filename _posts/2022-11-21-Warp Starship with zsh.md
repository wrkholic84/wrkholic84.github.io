---
title: "Warp Starship with zsh"
# author: wrkholic84
date: 2022-11-21 00:00:00 +0900
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
## Install Starship Terminal

brew를 통해 starship terminal을 설치한다.

```bash
brew install starship
```

1.1. starship terminal을 사용하기 위해서는 Nerd Font를 사용해야 한다. [https://www.nerdfonts.com/font-downloads](https://www.nerdfonts.com/font-downloads) 에서 적당한 폰트를 골라 설치하자. Hack Nerd Font를 추천.

다운로드 후 Hack Regular Nerd Font Complete.ttf 파일만 설치해주면 된다.

1.2. 명령 프롬프트에서 starship config를 실행하면 vi editor가 열린다. 아래 내용을 넣고 저장한다.

[https://raw.githubusercontent.com/ChristianLempa/dotfiles/main/.config/starship.toml](https://raw.githubusercontent.com/ChristianLempa/dotfiles/main/.config/starship.toml) 참조

```shell
# ~/.config/starship.toml

# Inserts a blank line between shell prompts
add_newline = false

# Change command timeout from 500 to 1000 ms
command_timeout = 1000

# Change the default prompt format
# ---
# old config ...
# format = """\
# [╭╴](238)$env_var \
# $all[╰─](238)$character"""

# new config
format = """$env_var $all"""

# Change the default prompt characters
[character]
# old config
# success_symbol = "[](238)"
# error_symbol = "[](238)"
success_symbol = ""
error_symbol = ""

# Shows an icon that should be included by zshrc script based on the distribution or os
[env_var.STARSHIP_DISTRO]
format = '[$env_value](white)'
variable = "STARSHIP_DISTRO"
disabled = false

# Shows the username
[username]
style_user = "white"
style_root = "white"
format = "[$user]($style) "
disabled = false
show_always = true

[hostname]
ssh_only = false
format = "on [$hostname](bold yellow) "
disabled = false

[directory]
truncation_length = 1
truncation_symbol = "…/"
home_symbol = " ~"
read_only_style = "197"
read_only = "  "
format = "at [$path]($style)[$read_only]($read_only_style) "

[git_branch]
symbol = " "
format = "via [$symbol$branch]($style) "
# truncation_length = 4
truncation_symbol = "…/"
style = "bold green"

[git_status]
format = '[\($all_status$ahead_behind\)]($style) '
style = "bold green"
conflicted = "🏳"
up_to_date = " "
untracked = " "
ahead = "⇡${count}"
diverged = "⇕⇡${ahead_count}⇣${behind_count}"
behind = "⇣${count}"
stashed = " "
modified = " "
staged = '[++\($count\)](green)'
renamed = "襁 "
deleted = " "

[kubernetes]
format = 'via [ﴱ $context\($namespace\)](bold purple) '
disabled = false

# (deactivated because of no space left)
# 
[terraform]
format = "via [ terraform $version]($style) 壟 [$workspace]($style) "
disabled = true

[vagrant]
format = "via [ vagrant $version]($style) "
disabled = true

[docker_context]
format = "via [ $context](bold blue) "
disabled = true

[helm]
format = "via [ $version](bold purple) "
disabled = true

[python]
symbol = " "
python_binary = "python3"
disabled = true

[nodejs]
format = "via [ $version](bold green) "
disabled = true

[ruby]
format = "via [ $version]($style) "
disabled = true
```

1.3. 프롬프트를 예쁘게 표시하기 위해 홈디렉터리 밑 다음 경로(.zsh/starship.zsh)에 아래 내용으로 파일을 만들어 준다. [https://raw.githubusercontent.com/ChristianLempa/dotfiles/main/.zsh/starship.zsh](https://raw.githubusercontent.com/ChristianLempa/dotfiles/main/.zsh/starship.zsh) 참조

```shell
# find out which distribution we are running on
LFILE="/etc/*-release"
MFILE="/System/Library/CoreServices/SystemVersion.plist"
if [[ -f $LFILE ]]; then
  _distro=$(awk '/^ID=/' /etc/*-release | awk -F'=' '{ print tolower($2) }')
elif [[ -f $MFILE ]]; then
  _distro="macos"
fi

# set an icon based on the distro
# make sure your font is compatible with https://github.com/lukas-w/font-logos
case $_distro in
    *kali*)                  ICON="ﴣ";;
    *arch*)                  ICON="";;
    *debian*)                ICON="";;
    *raspbian*)              ICON="";;
    *ubuntu*)                ICON="";;
    *elementary*)            ICON="";;
    *fedora*)                ICON="";;
    *coreos*)                ICON="";;
    *gentoo*)                ICON="";;
    *mageia*)                ICON="";;
    *centos*)                ICON="";;
    *opensuse*|*tumbleweed*) ICON="";;
    *sabayon*)               ICON="";;
    *slackware*)             ICON="";;
    *linuxmint*)             ICON="";;
    *alpine*)                ICON="";;
    *aosc*)                  ICON="";;
    *nixos*)                 ICON="";;
    *devuan*)                ICON="";;
    *manjaro*)               ICON="";;
    *rhel*)                  ICON="";;
    *macos*)                 ICON="";;
    *)                       ICON="";;
esac

export STARSHIP_DISTRO="$ICON"
```

그리고 아래 내용을 .zshrc 의 맨 밑에 넣어준다.

```shell
[[ -f ~/.zsh/starship.zsh ]] && source ~/.zsh/starship.zsh
eval "$(starship init zsh)"
```

## Install Warp

brew를 통해 warp를 설치한다.

```bash
brew install warp
```

warp 앱을 실행시키면 계정을 필요로 한다. 적당한 이메일로 가입

## Set Warp Preferences

3.1. 터미널 줄 간격 줄이기

Apperance > Blocks > Compact mode : On

3.2. 터미널 프롬프트 표시

Features > Honor User’s custom prompt(PS1) : On

3.3. 명령어 자동 완성

Open Completions Menu as you type : On

3.4. 폰트 변경

Appearnce > Text > Terminal font : Hack Nerd Font

## Set Font on Terminals

터미널을 사용하는 여러 프로그램에 앞서 설치한 Hack Nerd Font를 기본 폰트로 설정해준다.

4.1. VSCode

Command + , 로 VSCode 설정파일을 열고 font 를 검색.

Edit in settings.json 을 열어서, 아래 내용을 추가한다.

```json
"terminal.integrated.fontFamily": "'Hack Nerd Font'",
```