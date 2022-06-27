---  
layout: post  
title: Shell tips  
subtitle: updating  
tags: [Tips]
comments: true
---
1. back to the last directory  
```shell
$ cd -
```

2. clear terminal  
clear all
```shell
$ clear
```
clear and keep history output```ctrl + L```

3. save & take out directories  
```shell
$ pushd my_dir # cd and save dir
$ popd my_dir # cd to the latest
```

4. history  
For zsh: edit `~/.zshrc` add  
```shell
export HISTFILEFLESIZE=1000000000
export HISTSIZE=1000000000
export HISTTIMEFORMAT="[%F %T ]"
setopt HIST_IGNORE_ALL_DUPS
```
Functions:   
- Drop dublicates.  
- Show date and time  

```shell
$ history -E
    2  23.6.2022 18:53  cd

    5  23.6.2022 18:54  hisotry

    9  23.6.2022 18:54  ps -ef | clash

   10  23.6.2022 18:55  fg
```