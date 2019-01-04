---
date: 2019-01-02
title: Cheat sheet for common used CLI
---

## Calculate lines of code in project
```
find . -name '*.scala' | xargs wc -l
```  
## SSH tunel
```
ssh -L 8002:target-host-ip:80 -i ~/.ssh/ssh-key hostname@host-ip
```
## Generate table of contents for markdown
```
gh-md-toc README.md
```

## Reference


