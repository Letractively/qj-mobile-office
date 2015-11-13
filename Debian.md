версия `6.0 squeeze`

`man some_command` - справка (выход Q)

`tar -cvzpf some.zip somefileORfolder` - запаковать папку {-c создать архив, -v показать все архивируемые файлы, -z zip, -p сохранять права, =f some.zip явно называем архив}

`ps ax | grep ABC` - проверить работает ли ABC

`df -h` - анализ свободного места

`find / -name some_file` - поиск файла

`vim /etc/crontab` - задания
```
0 4 * * * root ruby /var/www/1.rb        #каждый день в 04:00
0,20,40 * * * * root ruby /var/www/2.rb  #каждые 20 минут

#min (0 - 59)
#hour (0 - 23)
#day of month (1 - 31)
#month (1 - 12)
#day of week (0 - 6) (0 is Sunday, or use names)
```


---


  * Система управления пакетами
`vim /etc/apt/sources.list`
```
deb http://ftp.ru.debian.org/debian/ squeeze main
deb-src http://ftp.ru.debian.org/debian/ squeeze main

deb http://mirror.yandex.ru/debian/ squeeze main
deb-src http://mirror.yandex.ru/debian/ squeeze main

deb http://ftp.ie.debian.org/debian squeeze main
deb-src http://ftp.ie.debian.org/debian/ squeeze main

deb http://security.debian.org/ squeeze/updates main
deb-src http://security.debian.org/ squeeze/updates main
```
| обновление списка пакетов согласно `/etc/apt/sources.list` | `apt-get update` |
|:-----------------------------------------------------------|:-----------------|
| обновить установленные пакеты                              | `apt-get upgrade` |
| установка зависимостей для пакета                          | `apt-get build-dep ?` |
| установка пакета                                           | `apt-get install ?` |
| удаление пакета                                            | `apt-get --purge remove ?` |
| удаление неиспользуемых пакетов                            | `apt-get autoremove` |
| список установленных пакетов                               | `dpkg --get-selections` |

  * Текстовый редактор `vim ?`
| режим редактирования | A |
|:---------------------|:--|
| сохранить            | `:w` |
| сохранить и выйти    | `:wq` |
| выйти без сохранения | `:q!` |
| удалить строку       | `DD` |
| поиск строки по файлу | `/some_string_to_find` |

  * [Прочие команды](http://qj-mobile-office.googlecode.com/files/debian_cheat_sheet.pdf)

# Доступ #

  * `ssh-keygen -t rsa` (passphrase может быть пустым)
  * `vim /etc/ssh/sshd_config`
```
PermitRootLogin         yes
PasswordAuthentication  no
RSAAuthentication       yes
PubkeyAuthentication    yes
AuthorizedKeysFile      .ssh/authorized_keys
```
  * `cat .ssh/id_rsa.pub >> .ssh/authorized_keys`
  * `/etc/init.d/ssh restart`
  * выкачиваем приватный ключ id\_rsa и прогоняем его через `puttygen.exe`: File - Load private key - Save private key (получили .ppk-файл)
  * в настройках `putty`: Connection - SSH - Auth - указываем наш .ppk

# Основа #

`apt-get update; apt-get upgrade; cd /usr/src`

  * Percona Server
    * `mkdir percona; cd percona`
    * `wget -r -l 1 -nd -A deb` **???** (берем самый новый http://www.percona.com/downloads/Percona-Server-5.5/ - deb/squeeze/x86\_64)
    * `dpkg` -i `*`.deb
    * `apt-get -f -y install`
    * `cd ..`
  * Lua
    * `wget` http://luajit.org/download/LuaJIT-2.0.0-beta9.tar.gz
    * `tar xzf LuaJIT-2.0.0-beta9.tar.gz`
    * `cd LuaJIT-2.0.0-beta9`
    * `make`
    * `make install`
    * `ln -sf luajit-2.0.0-beta9 /usr/local/bin/luajit`
    * `export LUAJIT_LIB=/usr/local/lib/lua/5.1/`
    * `export LUAJIT_INC=/usr/local/include/luajit-2.0/`
  * NDK
    * `wget` https://github.com/simpl/ngx_devel_kit/tarball/master; `tar xzf master; rm master`
  * NGX\_LUA
    * `wget` https://github.com/chaoslawful/lua-nginx-module/tarball/master; `tar xzf master; rm master`
  * PCRE
    * `wget` [ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.21.tar.gz;](ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.21.tar.gz;) `tar xzf pcre-8.21.tar.gz`
  * OpenSSL
    * `apt-get install openssl libssl-dev`
  * NGINX
    * `wget` http://nginx.org/download/nginx-1.1.11.tar.gz
    * `tar xzf nginx-1.1.11.tar.gz`
    * `cd nginx-1.1.11`
    * `./configure` --prefix=/opt/nginx --add-module=../simpl-ngx\_devel\_kit-**???** --add-module=../chaoslawful-lua-nginx-module-**???** --with-pcre=../pcre-8.21 --with-http\_ssl\_module
    * `vim objs/Makefile`

ближе к концу в секции
```
../pcre-8.21/Makefile:  objs/Makefile
        cd ../pcre-8.21 \
        && if [ -f Makefile ]; then $(MAKE) distclean; fi \
        && CC="$(CC)" CFLAGS="-O2 -fomit-frame-pointer -pipe " \
        ./configure --disable-shared
```
меняем последнюю строку
```
        ./configure --enable-utf8 --enable-unicode-properties --enable-jit --disable-shared
```
выполняем

`make -j2; make install`

# Настройка #

  * `vim /etc/init.d/nginx` (автозагрузка)
```
#! /bin/sh
### BEGIN INIT INFO
# Provides:          nginx
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
### END INIT INFO
# 0 - halt, 6 - reboot, 1 - "rescue" with no daemons
# 2 - 5 ~normal

set -e

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DESC="nginx daemon"
NAME=nginx
DAEMON=/opt/nginx/sbin/$NAME
PIDFILE=/opt/nginx/logs/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME

# Gracefully exit if the package has been removed.
test -x $DAEMON || exit 0

case "$1" in
 start)
 echo -n "Starting $DESC: $NAME"
 (test -f $PIDFILE && echo -n " already running") || $DAEMON
 ;;
 stop)
 echo -n "Stopping $DESC: $NAME"
 $DAEMON -s stop
 ;;
 restart)
 echo -n "Restarting $DESC: $NAME"
 $DAEMON -s reload
 ;;
 *)
 echo "Usage: $SCRIPTNAME {start|stop|restart}" >&2
 exit 3
 ;;
esac

echo "."

exit 0
```

`chmod +x /etc/init.d/nginx; insserv nginx`

  * `vim /opt/nginx/conf/nginx.conf` (конфигурация сервера)
```
user  www-data;
worker_processes  1;

worker_rlimit_nofile 80000;

pid logs/nginx.pid;

events {
        worker_connections  1024;
}
http {
        include mime.types;
        default_type text/html;

        lua_package_path '/var/www/noaccess/?.lua;;';

        keepalive_timeout 65;
        client_max_body_size 150m;

        server_tokens off;

#        server { listen 81; location / {
# content_by_lua "local h = ngx.req.get_uri_args(); assert(h.from and h.to);
# require('s3').go('PUT', h.to, require('std').read(h.from), h.secure)"; }}

        server {
                listen 80;
                charset utf-8;

                root /var/www;

                location /error {
                        internal;
                        content_by_lua " ngx.say('error!')";
                }
                error_page 403 404 500 /error;
                location ~* \.php$ {
                        return 404;
                }
                location /noaccess {
                        return 403;
                }
                ############################################

                location =/index {
                        content_by_lua " ngx.say('hello!') ";
                }
        }
}
```
`mkdir -p /var/www/noaccess`
  * `vim /etc/security/limits.conf`
```
root soft nofile 80000
root hard nofile 80000
```
  * SMTP-сервер
    * apt-get install exim4
    * dpkg-reconfigure exim4-config
      * Type of mail configuration: _internet site_
      * System name: _localhost_
      * IP-Address to listen on: _127.0.0.1_
      * Other destinations which is accepted: _пусто_
      * Domains to relay for: _пусто_
      * Machines to relay for: _пусто_
      * Keep DNS minimal: _yes_
      * Delivery method for local mail: _Maildir_
      * Split configuration: _yes_
      * _пусто_
  * `vim /etc/logrotate.d/nginx`
```
/opt/nginx/logs/*log {
    daily
    rotate 14
    missingok
    notifempty
    compress
    sharedscripts
    postrotate
        [ ! -f /opt/nginx/logs/nginx.pid ] || kill -USR1 `cat /opt/nginx/logs/nginx.pid`
    endscript
}
```
  * `dpkg-reconfigure tzdata` (выбор часового пояса)
  * `vim ~/.vimrc`
```
syntax on
set encoding=utf8
```
  * `vim ~/.bashrc`
```
cd /var/www
```
  * `reboot`

---

## Lua-модули ##
  * luasql
    * `wget` https://github.com/downloads/keplerproject/luasql/luasql-2.2.0.tar.gz
    * `tar xzf luasql-2.2.0.tar.gz`
    * `cd luasql-2.2.0`
    * `vim config`
```
T            = mysql
LUA_INC      = $(PREFIX)/include/luajit-2.0
DRIVER_LIBS  = -lmysqlclient
DRIVER_INCS  = -I/usr/include/mysql
CFLAGS       = -O2 $(WARN) -fPIC -I$(COMPAT_DIR) $(DRIVER_INCS) $(INCS) -DLUASQL_VERSION_NUMBER='"$V"' $(DEFS)
```
    * `make; make install`
  * luasocket
    * `wget` http://files.luaforge.net/releases/luasocket/luasocket/luasocket-2.0.2/luasocket-2.0.2.tar.gz
    * `tar xzf luasocket-2.0.2.tar.gz`
    * `cd luasocket-2.0.2`
    * `vim config`
```
LUAINC = -I/usr/local/include/luajit-2.0
```
    * `make; make install`
  * lua cjson
    * `wget` http://www.kyne.com.au/~mark/software/download/lua-cjson-2.1.0.tar.gz
    * `tar xzf lua-cjson-2.1.0.tar.gz`
    * `cd lua-cjson-2.1.0`
    * `vim Makefile`
```
LUA_INCLUDE_DIR =   $(PREFIX)/include/luajit-2.0
```
    * `make; make install`


---

## FFmpeg ##

  * `cd /usr/src`
  * `wget http://www.debian-multimedia.org/pool/main/d/debian-multimedia-keyring/debian-multimedia-keyring_2010.12.26_all.deb; dpkg -i debian-multimedia-keyring_2010.12.26_all.deb`
  * `vim /etc/apt/sources.list`
```
deb http://www.debian-multimedia.org squeeze main
```
  * `apt-get update; apt-get install git libxvidcore4-dev libfaac-dev libfaad-dev`
  * `git clone git://source.ffmpeg.org/ffmpeg.git ffmpeg`
  * `wget http://www.tortall.net/projects/yasm/releases/yasm-1.2.0.tar.gz; tar xzf yasm-1.2.0.tar.gz; cd yasm-1.2.0; ./configure; make; make install; cd ..`
  * `git clone git://git.videolan.org/x264.git x264; cd x264; ./configure --enable-shared; make; make install; cd ..`
  * `cd ffmpeg; ./configure --enable-gpl --enable-nonfree --enable-pthreads --enable-libx264 --enable-libxvid --enable-libfaac --enable-shared --enable-version3; make; make install; cd ..`
  * `ffmpeg -i X -strict experimental -vcodec libx264 Y`

Если возникла ошибка `error while loading shared libraries: libavdevice.so.52: cannot open shared object file`
  * `cd /etc/ld.so.conf.d; vim any-libs.conf`
```
/usr/local/lib
```
  * `ldconfig`

Дополнительные утилиты: `p7zip` (архиватор 7z), `mc` (файловый менеджер), `pwgen` (генератор паролей)

Ссылки: [Lua-Nginx](https://github.com/chaoslawful/lua-nginx-module), [Lua](http://www.lua.org/manual/5.1/manual.html), [Nginx](http://nginx.org/ru/docs/http/ngx_http_core_module.html), [LuaSQL](http://www.keplerproject.org/luasql/manual.html), [LuaSocket](http://w3.impa.br/~diego/software/luasocket/reference.html),
[LuaCJSON](http://www.kyne.com.au/~mark/software/lua-cjson-manual.html#_api_functions)