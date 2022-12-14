# Postgres

## 利用 Docker 快速在 **当前工作目录(`pwd`)** 启动一个本地 postgres 环境

使用 alpine based image，并且挂载了 unix socket。

```bash
$ mkdir conf
$ docker run -i --rm postgres:15-alpine cat /usr/local/share/postgresql/postgresql.conf.sample > conf/postgresql.conf
$ docker run -d \
-v `pwd`/conf/postgresql.conf:/etc/postgresql/postgresql.conf \
-v `pwd`/data:/var/lib/postgresql/data \
-v /var/run/postgresql:/var/run/postgresql \
-p 5432:5432 \
-e POSTGRES_USER=<YOUR USER NAME> \
-e LANG=zh_CN.utf8 \
-e POSTGRES_INITDB_ARGS="--locale-provider=icu --icu-locale=zh-CN" \
-e POSTGRES_PASSWORD=<YOUR PASSWORD> \
postgres:15-alpine -c 'config_file=/etc/postgresql/postgresql.conf'
```

## 使用 `pgcli` 来获得拥有更友好的客户端

```bash
$ sudo pacman -S pgcli
# -or-
$ sudo apt-get install pgcli
# -or-
$ brew install pgcli
# -or-
$ pip install -U pgcli
```

### 默认启动直接通过 unix socket 连接 postgres

```bash
$ pgcli
Server: PostgreSQL 15.1
Version: 3.5.0
Home: http://pgcli.com
m4n5ter> exit
Goodbye!
```

### linux navicat reset

下面的方法不会丢失已经存在连接(Navicat 16 Premium)：

```bash
#!/bin/bash

# Backup
cp ~/.config/dconf/user ~/.config/dconf/user.bk
cp ~/.config/navicat/Premium/preferences.json ~/.config/navicat/Premium/preferences.json.bk

# Clear data in dconf
dconf reset -f /com/premiumsoft/navicat-premium/
# Remove data fields in config file
sed -i -E 's/,?"([A-Z0-9]+)":\{([^\}]+)},?//g' ~/.config/navicat/Premium/preferences.json## Links
```

* [Postgres Current Version Document](https://www.postgresql.org/docs/current)
