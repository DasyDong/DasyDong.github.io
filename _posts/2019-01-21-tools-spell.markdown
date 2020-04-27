---
title: "Tools: go项目spell拼写检查"
layout: post
date: 2019-01-21 22:07
image: /img/images/markdown.jpg
headerImage: false
tag:
- python
- go
- tools
category: blog
author: dong
description: Markdown summary with different options
---

- [Github](#github)
- [spell-check-go](#spell-check-go)
- [Prerequisites](#prerequisites)
- [How to Use?](#how-to-use)
- [Results](#results)
- [TODO](#todo)

## Github
[Github](https://github.com/DasyDong/spell-check-go)

## spell-check-go
The project is used to check en-US word typo or spelled incorrectly for Go repository based [pyenchant](https://github.com/pyenchant/pyenchant)

eg:

Stawrt > False

StarttAction > False

StarttAction > False


## Prerequisites
python 3

## How to Use?
Git clone this repo[spell-check](https://github.com/DasyDong/spell-check-go.git)

```
virtualenv -p python3 ~/venv3 && source ~/venv3/bin/activate

pip3 install -r requirement.txt

python spell_check.py project
```

## Results
Spell checking result is in spell_check_wrong.txt

Words In spell_check_ignore.txt is skipped to check


## TODO
Seems that Myspell is case insentive, But Don't understatnd why this happens

September > True

september > False


Day > True

day > True


weird!!!
