---
date: 2024-02-29
title: Setup Multi-User Environment for lbcd
tags: ["bitcoin", "blockchain"]
categories: ["Blockchain"]
---

## Prerequisites

Install [Caddy](https://caddyserver.com/docs/install)

Install [lbcd](https://github.com/lbryio/lbcd)

## Configuration

For Caddyfile

```shell
rpc.yourdomain.xyz {
    basicauth * {
        user1 hashed-password1
        user2 hashed-password2
    }
    reverse_proxy :9245 {
        header_up Authorization "Basic abcdefg"
    }
}
```

Hashed password is generated with

```shell
caddy hash-password -p plain-text-password
```

The `Authorization` header value `Basic abcdefc` can be extracted using Postman by setting Authorization -> Basic Auth with username and password.

Post to `https://rpc.yourdomain.xyz`
Body:

```json
{
  "version": "2.0",
  "id": 8337029974246184036,
  "method": "getblockcount",
  "params": []
}
```

## Start

```shell
caddy start
caddy adapt
caddy stop
```

```shell
~/go/bin/lbcd --rpcuser=user --rpcpass=password --notls

## shutdown
kill -9 lbcd-pid
```