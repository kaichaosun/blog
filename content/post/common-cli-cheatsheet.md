---
date: 2019-01-02
title: Cheatsheet for Common Usage CLI
tags: ["Tools", "DevOps"]
categories: ["DevOps"]
---

## Curl Post JSON
```shell
curl -i -d '{"id": "111111","date": "2018-04-27T00:00:00+10:00","des": "test"}]' -H "Content-Type: application/json" -X POST https://endpoint

curl -i -d '{"a":"1","b":"2"}' -H "Content-Type: application/json" -X POST https://endpoint
```

## Database

### MySQL

```shell
SHOW VARIABLES;

SELECT FROM_UNIXTIME(1510289234);

SELECT NOW();

SELECT UNIX_TIMESTAMP();
```
### MongoDB

```shell
mongod --dbpath ~/data/mongodb
```

## Zip / tar

```
zip filename.zip file1 file2
```

```shell
tar -czvf name-of-archive.tar.gz /path/to/directory-or-file
```

- -c: **C**reate an archive.
- -z: Compress the archive with g**z**ip.
- -v: Display progress in the terminal while creating the archive, also known as “**v**erbose” mode. The v is always optional in these commands, but it’s helpful.
- -f: Allows you to specify the **f**ilename of the archive.

```
tar -xzvf archive.tar.gz
```

## cp

```shell
rsync -av --progress /Users/kcsun/github/uiux/sourcefoder/ ./ --exclude .git
```



## SSH / scp

```
// setup tunel
ssh -L 8002:target-host-ip:80 -i ~/.ssh/ssh-key hostname@host-ip

// from local to remote
scp /filename hostname@ip:~

// from remote to local
scp -i ~/.ssh/ssh-key root@host:~/hello ./
```

## Docker
```
// docker download file
docker cp <containerId>:/file/path/within/container /host/path/target

// show sha256
docker inspect --format='{{index .RepoDigests 0}}' $IMAGE
```

## Journalctl
```
// Display kernal message:
journalctl -k | grep -i -e memory -e oom
```

## Generate table of contents for markdown
```
gh-md-toc README.md
```

## Calculate lines of code in project
```
find . -name '*.scala' | xargs wc -l
```

## Docker override entrypoint

```
docker run -it --entrypoint /usr/bin/redis-cli example/redis
```

## Git 
**Reset local branch after force push:**

```shell
git reset origin/master --hard
```

**Rebase to certain commit, like change commit msg:**
```shell
git rebase -i commit-id // p to pick, r to change msg
```

** Delete branch with illegal name:**
```shell
git branch -D -- --track
```

### Submodule

To remove a submodule

<https://gist.github.com/myusuf3/7f645819ded92bda6677>

```
git submodule status
```



## Nodejs npm mirror

```
npm config set registry=http://registry.npm.taobao.org
```

## Rust and Cargo

### Source

Configiure china source, modify the file `$HOME/.cargo/config`

```
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```

### cargo expand

<https://github.com/dtolnay/cargo-expand>



