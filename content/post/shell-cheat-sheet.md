---
date: 2019-01-02
title: Cheatsheet for Shell programming
---

## If else
```
if [[ -z ${APP_ENV} ]]; then
  echo "The APP_ENV should be set to prod or staging, got: $0"
  exit -1
fi
```

## For loop
```
for i in *.tar; do echo working on $i; tar xvzf $i ; done
```

## Reference


