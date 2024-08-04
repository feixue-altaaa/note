# why

# what

**Nginx (engine x)** 是一款轻量级的 Web 服务器 、反向代理服务器及电子邮件（IMAP/POP3）代理服务器

![img](https://raw.githubusercontent.com/dunwu/images/master/cs/web/nginx/nginx.jpg)

## **什么是反向代理？**

反向代理（Reverse Proxy）方式是指以代理服务器来接受 internet 上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给 internet 上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器

![img](https://raw.githubusercontent.com/dunwu/images/master/cs/web/nginx/reverse-proxy.png)

# how

## 常见命令

### 命令行参数

nginx支持以下命令行参数：

- `-?` 或者 `-h` —— 打印命令行参数帮助信息
- `-c file` —— 使用指定的配置文件来替代默认的配置文件
- `-g directive` —— 设置 [全局配置指令](http://nginx.org/en/docs/ngx_core_module.html)，例如：

```bash
nginx -g "pid /var/run/nginx.pid; worker_processes `sysctl -n hw.ncpu`;"
```

- `-p prefix` —— 设置 nginx 路径前缀，比如一个存放着服务器文件的目录（默认是 `/usr/local/nginx` ）。
- `-q` —— 在配置测试期间禁止非错误信息。
- `-s signal` —— 向 Master 进程发送信号，信号参数可以是以下：
    - `stop` —— 快速关闭。
    - `quit` —— 正常关闭。
    - `reload` —— 重新加载配置，使用新配置后启动新的 Worker 进程并正常退出旧的工作进程。
    - `reopen` —— 重新打开日志文件。
- `-t` —— 测试配置文件：nginx 检查配置文件的语法正确性，之后尝试打开配置中引用到的文件。
- `-T` —— 与 `-t` 一样，但另外将配置文件转储到标准输出（1.9.2）。
- `-v` —— 打印 nginx 版本。
- `-V` —— 打印 nginx 版本，编译器版本和配置参数

可以用信号控制 nginx。默认情况下，主进程（Master）的 pid 写在 `/use/local/nginx/logs/nginx.pid` 文件中。这个文件的位置可以在配置时更改或者在 nginx.conf 文件中使用 `pid` 指令更改。Master 进程支持以下信号：

| 信号      | 作用                                                         |
| :-------- | :----------------------------------------------------------- |
| TERM, INT | 快速关闭                                                     |
| QUIT      | 正常退出                                                     |
| HUP       | 当改变配置文件时，将有一段过渡时间段（仅 FreeBSD 和 Linux），新启动的 Worker 进程将应用新的配置，旧的 Worker 进程将被平滑退出 |
| USR1      | 重新打开日志文件                                             |
| USR2      | 升级可执行文件                                               |
| WINCH     | 正常关闭 Worker 进程                                         |

Worker 进程也是可以用信号控制的，尽管这不是必须的。支持以下信号：

| 信号      | 作用                                                         |
| :-------- | :----------------------------------------------------------- |
| TERM, INT | 快速关闭                                                     |
| QUIT      | 正常关闭                                                     |
| USR1      | 重新打开日志文件                                             |
| WINCH     | 调试异常终止（需要开启 [debug_points](http://nginx.org/en/docs/ngx_core_module.html#debug_points)） |

## Nginx安装与运维

### 启动、停止和重新加载配置

要启动 nginx，需要运行可执行文件。nginx 启动之后，可以通过调用可执行文件附带 `-s 参数` 来控制它。 使用以下语法：

```bash
nginx -s 信号
```

信号可能是以下之一：

- stop - 立即关闭
- quit - 正常关闭
- reload - 重新加载配置文件
- reopen - 重新打开日志文件

例如，要等待工作进程处理完当前的请求才停止 nginx 进程，可以执行以下命令：

```bash
nginx -s quit
```

> 这个命令的执行用户应该是与启动nginx用户是一致的

在将重新加载配置的命令发送到 nginx 或重新启动之前，配置文件所做的内容更改将不会生效。要重新加载配置，请执行：

```bash
nginx -s reload
```

一旦主进程（Master）收到要重新加载配置（reload）的信号，它将检查新配置文件的语法有效性，并尝试应用其中提供的配置。如果成功，主进程将启动新的工作进程（Worker），并向旧工作进程发送消息，请求它们关闭。否则，主进程回滚更改，并继续使用旧配置。旧工作进程接收到关闭命令后，停止接受新的请求连接，并继续维护当前请求，直到这些请求都被处理完成之后，旧工作进程将退出。

可以借助 Unix 工具（如 kill 工具）将信号发送到 nginx 进程，信号直接发送到指定进程 ID 的进程。默认情况下，nginx 主进程的进程 ID 是写入在 `/usr/local/nginx/logs` 或 `/var/run` 中的 nginx.pid 文件中。例如，如果主进程 ID 为 1628，则发送 QUIT 信号让 nginx 正常平滑关闭，可执行：

```bash
kill -s QUIT 1628
```

获取所有正在运行的 nginx 进程列表，可以使用 `ps` 命令，如下：

```bash
ps -ax | grep nginx
```

> 详细安装方法请参考：[Nginx 运维](docs/nginx-ops.md)

nginx 的使用比较简单，就是几条命令

常用到的命令如下

```batch
nginx -s stop       快速关闭Nginx，可能不保存相关信息，并迅速终止web服务。
nginx -s quit       平稳关闭Nginx，保存相关信息，有安排的结束web服务。
nginx -s reload     因改变了Nginx相关配置，需要重新加载配置而重载。
nginx -s reopen     重新打开日志文件。
nginx -c filename   为 Nginx 指定一个配置文件，来代替缺省的。
nginx -t            不运行，仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件。
nginx -v            显示 nginx 的版本。
nginx -V            显示 nginx 的版本，编译器版本和配置参数。
```

如果不想每次都敲命令，可以在 nginx 安装目录下新添一个启动批处理文件**startup.bat**，双击即可运行。内容如下：

```batch
@echo off
rem 如果启动前已经启动nginx并记录下pid文件，会kill指定进程
nginx.exe -s stop

rem 测试配置文件语法正确性
nginx.exe -t -c conf/nginx.conf

rem 显示版本信息
nginx.exe -v

rem 按照指定配置去启动nginx
nginx.exe -c conf/nginx.conf
```

### 连接处理方式

nginx 支持多种连接处理方式，每一种方式是否可用取决于所用的平台。在支持几种方式的平台上，nginx 会自动选择最有效的方式，然而，如果您需要明确指定使用哪一种方式，可以使用 [use](http://nginx.org/en/docs/ngx_core_module.html#use) 指令指定。

支持以下集中处理方式：

- **select**，标准方式。当平台上缺乏其他有效的方式时自动构建。`--with-select-module` 和 `-without-select_module` 配置参数开启或者禁用此模块构建。
- **poll**，标准方式，当平台上缺乏其它有效的处理方式时自动构建此模块。`-with-poll_moudle` 和 `-without-poll_module` 配置项开启或者禁用此模块构建。
- **kqueue**，在 FreeBSD 4.1+, OpenBSD 2.9+, NetBSD 2.0, 和 macOS 使用有效。
- **epoll**，在 Linux 2.6+ 上使用有效。

> 从 1.11.3 起支持 EPOLLRDHUP（Linux 2.6.17，glibc 2.8）和 EPOLLEXCLUSIVE（Linux 4.5，glibc 2.24）标志。一些类似于 SuSE 8.2 这样的老版本提供了对 2.4 内核支持 epll 的补丁。

- **/dev/poll**，在 Solaris 7 11/99+，HP / UX 11.22+（eventport），IRIX 6.5.15+ 和 Tru64 UNIX 5.1A+ 有效。
- **eventport**，事件端口，在 Solaris 10+ 有效（由于已知问题，推荐使用 `/dev/poll` 方式代替）

### 服务器名称

- [通配符名称](#通配符名称)
- [正则表达式名称](#正则表达式名称)
- [其他名称](#其他名称)
- [国际化名称](#国际化名称)

服务器名称是使用 [server_name](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) 指令定义的，它确定了哪一个 [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server) 块被给定的请求所使用。另请参见 [nginx 如何处理请求](nginx如何处理请求)。可以使用精确的名称、通配符或者正则表达式来定义他们：

```nginx
server {
    listen       80;
    server_name  example.org  www.example.org;
    ...
}

server {
    listen       80;
    server_name  *.example.org;
    ...
}

server {
    listen       80;
    server_name  mail.*;
    ...
}

server {
    listen       80;
    server_name  ~^(?<user>.+)\.example\.net$;
    ...
}
```

当通过名称搜索虚拟服务器时，如果名称与多个指定的变体匹配，例如通配符和正则表达式，则将按照优先顺序选择第一个匹配的变体：

1. 精确的名称
2. 以星号开头的最长的通配符名称，例如 `*.example.org`
3. 以星号结尾的最长的通配符名称，例如 `mail.*`
4. 第一个匹配的正则表达式（按照在配置文件中出现的顺序）

#### 通配符名称
通配符名称只能在名称的开头或者结尾包含一个星号，且只能在点的边界上包含星号。这些名称为 `www.*.example.org` 和 `w*.example.org` 都是无效的。然而，可以使用正则表达式指定这些名称，例如，`~^www\..+\.example\.org$”` 和 `~^w.*\.example\.org$`。星号可以匹配多个名称部分。名称 `*.example.org` 不仅匹配了 `www.example.org`，还匹配了 `www.sub.example.org`。

可以使用 `.example.org` 形式的特殊通配符名称来匹配确切的名称 `example.org` 和 通配符 `*.example.org`。

#### 正则表达式名称
nginx 使用的正则表达式与 Perl 编程语言（PCRE）使用的正则表达式兼容。要使用正则表达式，服务器名称必须以波浪符号开头：

```nginx
server_name  ~^www\d+\.example\.net$;
```
否则将被视为确切的名称，或者如果表达式中包含星号，将被视为通配符名称（并且可能是无效的）。别忘了设置 `^` 和 `$` 锚点。它们在语法上不是必须的，但在逻辑上是需要的。还要注意的是，域名的点应该使用反斜杠转义。正则表达式中的 `{` 和 `}` 应该被引号括起：

```nginx
server_name  "~^(?<name>\w\d{1,3}+)\.example\.net$";
```
否则 nginx 将无法启动并且显示错误信息：

```
directive "server_name" is not terminated by ";" in ...
```
正则表达式名称捕获到的内容可以作为变量为后面所用：

```nginx
server {
    server_name   ~^(www\.)?(?<domain>.+)$;

    location / {
        root   /sites/$domain;
    }
}
```
PCRE 库使用以下语法捕获名称：
- `?<name>` Perl 5.10 兼容语法，自 PCRE-7.0 起支持
- `?'name'` Perl 5.10 兼容语法，自 PCRE-7.0 起支持
- `?P<name>` Python 兼容语法，自 PCRE-4.0 起支持

如果 nginx 无法启动并且显示错误信息：

```
pcre_compile() failed: unrecognized character after (?< in ...
```

说明 PCRE 库比较老，应该尝试使用语法 `?P<name>`。捕获也可以使用数字形式：

```nginx
server {
    server_name   ~^(www\.)?(.+)$;

    location / {
        root   /sites/$2;
    }
}
```
然而，这种方式仅限于简单情况（如上所述），因为数字引用容易被覆盖。

#### 其他名称
有一些服务器名称需要被特别处理。

如果需要处理一个在不是默认的 [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server) 块中的没有 **Host** 头字段的请求，应该指定一个空名称：

```nginx
server {
    listen       80;
    server_name  example.org  www.example.org  "";
    ...
}
```
如果 [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server) 块中没有定义 [server_name](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name)，那么 nginx 将使用空名称作为服务器名。

> 此情况下，nginx 的版本到 0.8.48 使用机器的主机名作为服务器名称。

如果服务器名称被定义为 `$hostname`（0.9.4），则使用机器的主机名。

如果有人使用 IP 地址而不是服务器名称来发出请求，则 **Host** 请求 header 域将包含 IP 地址，可以使用 IP 地址作为服务器名称来处理请求。

```nginx
server {
    listen       80;
    server_name  example.org
                 www.example.org
                 ""
                 192.168.1.1
                 ;
    ...
}
```
在所有的服务器示例中，可以发现一个奇怪的名称 `_`：

```nginx
server {
    listen       80  default_server;
    server_name  _;
    return       444;
}
```
这个名称没有什么特别之处，它只是众多无效域名中的一个，它永远不会与任何真实的名称相交。此外还有其他无效的名称，如 `--` 和 `!@#` 也是如此。

nginx 到 0.6.25 版本支持特殊的名称 `*`，这常被错误地理解为所有名称。它并不是所有或者通配符服务器名称。相反，它的功能现在是由 [server_name_in_redirect](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name_in_redirect) 指令提供。现在不推荐使用特殊的名称 `*`，应该使用 [server_name_in_redirect](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name_in_redirect) 指令。请注意，无法使用 [server_name](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) 指令来指定所有名称或者默认服务器。这是 [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 指令的属性，而不是 [server_name](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) 的。请参阅 [nginx 如何处理请求](nginx如何处理请求.md)。可以定义监听 `*:80` 和 `*:8080` 端口的服务器，并指定一个为 `*:8080` 端口的默认服务器，另一个为 `*:80` 端口的默认服务器。

```nginx
server {
    listen       80;
    listen       8080  default_server;
    server_name  example.net;
    ...
}

server {
    listen       80  default_server;
    listen       8080;
    server_name  example.org;
    ...
}
```

#### 国际化名称
应使用 [server_name](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) 指令中的ASCII（Punycode）表示来指定国际化域名（[IDN](https://en.wikipedia.org/wiki/Internationalized_domain_name)）

```nginx
server {
    listen       80;
    server_name  xn--e1afmkfd.xn--80akhbyknj4f;  # пример.испытание
    ...
}
```

#### 优化
确切的名称、以星号开头的通配符名称和以星号结尾的通配符名称被存储在绑定到监听端口的三种哈希表中。哈希表的大小可以在配置阶段优化，因此可以在 CPU 缓存未命中的最低情况下找到名称。设置哈希表的具体细节在单独的 [文档](http://nginx.org/en/docs/hash.html) 中提供。

首先搜索确切的名称哈希表。如果为找到名称，则会搜索以星号开头的通配符名称的哈希表。如果还是没有找到名称，则搜索以星号结尾的通配符名称哈希。

搜索通配符哈希表比搜索确切名称的哈希表要慢，因为名称是通过域部分搜索的。请注意，特殊通配符格式 `.example.org` 存储在通配符哈希表中，而不是确切名称哈希表中。

由于正则表达式是按顺序验证的，因此是最慢的方法，并且是不可扩展的。

由于这些原因，最好是尽可能使用确切的名称。例如，如果最常见被请求的服务器名称是 `example.org` 和 `www.example.org`，则明确定义它们是最有效的：

```nginx
server {
    listen       80;
    server_name  example.org  www.example.org  *.example.org;
    ...
}
```

简化形式：

```nginx
server {
    listen       80;
    server_name  .example.org;
    ...
}
```

如果定义了大量的服务器名称，或者定义了非常长的服务器名称，则可能需要调整 `http` 级别的 [server_names_hash_max_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_max_size) 和 [server_names_hash_bucket_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size) 指令。[server_names_hash_bucket_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size) 指令的默认值可能等于 32 或者 64 或者其他值。具体取决于 CPU 超高速缓存储器线的大小。如果默认值为 32，并且服务器名称定义为 `too.long.server.name.example.org`，则 nginx 将无法启动并显示错误信息：

```bash
could not build the server_names_hash,
you should increase server_names_hash_bucket_size: 32
```
在这种情况下，指令值应该增加到 2 倍：

```nginx
http {
    server_names_hash_bucket_size  64;
    ...
```
如果定义了大量的服务器名称，则会显示另一个错误信息：

```bash
could not build the server_names_hash,
you should increase either server_names_hash_max_size: 512
or server_names_hash_bucket_size: 32
```
在这种情况下，首先尝试将 [server_names_hash_max_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_max_size) 设置为接近服务器名称数量的数值。如果这样没有用，或者如果 nginx 的启动时间长的无法忍受，那么请尝试增加 [server_names_hash_bucket_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size) 的值。

如果服务器是监听端口的唯一服务器，那么 nginx 将不会验证服务器名称（并且不会为监听端口建立哈希表）。但是，有一个例外。如果服务器名称具有捕获（captures）的正则表达式，则 nginx 必须执行表达式才能获取捕获。

#### 兼容性
- 自 0.9.4 起，支持特殊的服务器名称 `$hostname`。
- 自 0.8.48 起，默认服务器名称的值为空名称 `""`。
- 自 0.8.25 其，已经支持命名的正则表达式服务器名称捕获。
- 自 0.7.40 起，支持正则表达式名称捕获。
- 自 0.7.12 起，支持空的服务器名称 `""`。
- 自 0.6.25 起，支持使用通配符服务器名称和正则表达式作为第一服务器名称。
- 自 0.6.7 起，支持正则表达式服务名称。
- 自 0.6.0 起，支持类似 `example.*` 的通配符。
- 自 0.3.18 起，支持类似 `.example.org`。
- 自 0.1.13 起，支持类似 `*.example.org` 的通配符。

由 **Igor Sysoev** 书写，由 **Brian Mercer** 编辑

### QUIC 和 HTTP/3 支持

- [从源码构建](#从源码构建)
- [配置](#配置)
- [配置示例](#配置示例)
- [故障排除](#故障排除)

从1.25.0后，对 [QUIC](https://datatracker.ietf.org/doc/html/rfc9000) 和 [HTTP/3](https://datatracker.ietf.org/doc/html/rfc9114) 协议的支持可用。同时，1.25.0之后，QUIC 和 HTTP/3 支持在Linux二进制包 ([binary package](https://nginx.org/en/linux_packages.html))中可用。

> QUIC 和 HTTP/3 支持是实验性的，请谨慎使用。

#### 从源码构建

使用`configure`命令配置构建。请参考[从源码构建 nginx ](../How-To/从源码构建nginx.md)以获得更多细节。

当配置nginx时，可以使用 [`--with-http_v3_module`](../How-To/从源码构建nginx.md#http_v3_module) 配置参数来启用 QUIC 和 HTTP/3。

构建nginx时建议使用支持 QUIC 的 SSL 库，例如 [BoringSSL](https://boringssl.googlesource.com/boringssl)，[LibreSSL](https://www.libressl.org/)，或者 [QuicTLS](https://github.com/quictls/openssl)。否则，将使用不支持[早期数据](../模块参考/http/ngx_http_ssl_module.md#ssl_early_data)的[OpenSSL](https://openssl.org/)兼容层。

使用以下命令为 nginx 配置 [BoringSSL](https://boringssl.googlesource.com/boringssl)：

```bash
./configure
    --with-debug
    --with-http_v3_module
    --with-cc-opt="-I../boringssl/include"
    --with-ld-opt="-L../boringssl/build/ssl
                   -L../boringssl/build/crypto"
```

或者，可以使用 [QuicTLS](https://github.com/quictls/openssl) 配置 nginx：

```bash
./configure
    --with-debug
    --with-http_v3_module
    --with-cc-opt="-I../quictls/build/include"
    --with-ld-opt="-L../quictls/build/lib"
```

或者，可以使用现代版本的 [LibreSSL](https://www.libressl.org/) 配置 nginx：

```bash
./configure
    --with-debug
    --with-http_v3_module
    --with-cc-opt="-I../libressl/build/include"
    --with-ld-opt="-L../libressl/build/lib"
```

配置完成后，使用 `make` 编译和安装 nginx。

#### 配置

[ngx_http_core_module](../模块参考/http/ngx_http_core_module.md) 模块中的 `listen` 指令获得了一个新参数 [`quic`](../模块参考/http/ngx_http_core_module.md#quic)，它在指定端口上通过启用 HTTP/3 over QUIC。

除了 `quic` 参数外，还可以指定 [`reuseport`](../模块参考/http/ngx_http_core_module.md#reuseport) 参数，使其在多个工作线程中正常工作。

有关指令列表，请参阅 [ngx_http_v3_module](https://nginx.org/en/docs/http/ngx_http_v3_module.html)。

要[启用](https://nginx.org/en/docs/http/ngx_http_v3_module.html#quic_retry)地址验证：

```nginx
quic_retry on;
```

要[启用](../模块参考/http/ngx_http_ssl_module.md#ssl_early_data) 0-RTT：

```nginx
ssl_early_data on;
```

要[启用](https://nginx.org/en/docs/http/ngx_http_v3_module.html#quic_gso) GSO (Generic Segmentation Offloading)：

```nginx
quic_gso on;
```

为多个 token [设置](https://nginx.org/en/docs/http/ngx_http_v3_module.html#quic_host_key) host key：

```nginx
quic_host_key <filename>;
```

QUIC 需要 TLSv1.3 协议版本，该版本在 [`ssl_protocols`](../模块参考/http/ngx_http_ssl_module.md#ssl_protocols) 指令中默认启用。

默认情况下，[GSO Linux 特定优化](http://vger.kernel.org/lpc_net2018_talks/willemdebruijn-lpc2018-udpgso-paper-DRAFT-1.pdf)处于禁用状态。如果相应的网络接口配置为支持 GSO，请启用它。

#### 配置示例

```nginx
http {
    log_format quic '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent" "$http3"';

    access_log logs/access.log quic;

    server {
        # for better compatibility it's recommended
        # to use the same port for quic and https
        listen 8443 quic reuseport;
        listen 8443 ssl;

        ssl_certificate     certs/example.com.crt;
        ssl_certificate_key certs/example.com.key;

        location / {
            # required for browsers to direct them to quic port
            add_header Alt-Svc 'h3=":8443"; ma=86400';
        }
    }
}
```

#### 故障排除

一些可能有助于识别问题的提示：

- 确保 nginx 是使用正确的 SSL 库构建的。
- 确保 nginx 在运行时使用正确的 SSL 库（`nginx -V` 显示当前使用的内容）。
- 确保客户端实际通过 QUIC 发送请求。建议从简单的控制台客户端（如 [ngtcp2](https://nghttp2.org/ngtcp2)）开始，以确保服务器配置正确，然后再尝试使用可能对证书非常挑剔的真实浏览器。
- 使用[调试支持](../介绍/调试日志.md)构建nginx并检查调试日志。它应包含有关连接及其失败原因的所有详细信息。所有相关消息都包含“`quic`”前缀，可以轻松过滤掉。
- 为了进行更深入的调查，可以使用以下宏启用其他调试：`NGX_QUIC_DEBUG_PACKETS, NGX_QUIC_DEBUG_FRAMES, NGX_QUIC_DEBUG_ALLOC, NGX_QUIC_DEBUG_CRYPTO`。
```bash
./configure
    --with-http_v3_module
    --with-debug
    --with-cc-opt="-DNGX_QUIC_DEBUG_PACKETS -DNGX_QUIC_DEBUG_CRYPTO"
```

### 调试日志

- [为指定客户端做调试日志](#为指定客户端做调试日志)
- [记录日志到循环内存缓冲区](#记录日志到循环内存缓冲区)

要开启调试日志，需要在编译 Nignx 时增加如下配置：

```bash
./configure --with-debug ...
```

之后应该使用 [error_log](http://nginx.org/en/docs/ngx_core_module.html#error_log) 指令设置调试级别：

```nginx
error_log /path/to/log debug;
```

要验证 nginx 是否已经配置为支持调试功能，请运行 `nginx -V` 命令：

```bash
configure arguments: --with-debug ...
```

预构建 [Linux](http://nginx.org/en/linux_packages.html) 包为 nginx-debug 二进制文件的调试日志提供了开箱即用的支持，可以使用命令运行。

```bash
service nginx stop
service nginx-debug start
```

之后设置 `debug` 级别。Windows 的 nginx 在编译时就已经支持调试日志，因此只需设置 `debug` 级别即可。

请注意，重新定义日志而不指定 `debug` 级别将禁止调试日志。在下面的示例中，重新定义 `server` 上的日志级别，nginx 将不会在此服务器上做日志调试。

```nginx
error_log /path/to/log debug;

http {
    server {
        error_log /path/to/log;
        ...
```

为了避免这种情况，重新定义日志级别应该被注释掉，或者明确指定日志为 `debug` 级别。

```nginx
error_log /path/to/log debug;

http {
    server {
        error_log /path/to/log debug;
        ...
```

#### 为指定客户端做调试日志

也可以仅为选定的客户端地址启用调试日志：

```nginx
error_log /path/to/log;

events {
    debug_connection 192.168.1.1;
    debug_connection 192.168.10.0/24;
}
```

#### 记录日志到循环内存缓冲区

调试日志可以被写入到循环内存缓冲区中：

```nginx
error_log memory:32m debug;
```

在 `debug` 级别将日志写入到内存缓冲区中，即使在高负载情况下也不会对性能产生重大的影响。在这种情况下，可以使用如下 `gdb` 脚本来提取日志：

```bash
set $log = ngx_cycle->log

while $log->writer != ngx_log_memory_writer
    set $log = $log->next
end

set $buf = (ngx_log_memory_buf_t *) $log->wdata
dump binary memory debug_log.txt $buf->start $buf->end
```

### 记录日志到 syslog

[error_log](http://nginx.org/en/docs/ngx_core_module.html#error_log) 和 [access_log](http://nginx.org/en/docs/http/ngx_http_log_module.html#access_log) 指令支持把日志记录到 syslog。以下配置参数将使 nginx 日志记录到 syslog：

```yaml
server=address
```
> 定义 syslog 服务器的地址，可以将该地址指定为附带可选端口的域名或者 IP，或者指定为 “unix:” 前缀之后跟着一个特定的 UNIX 域套接字路径。如果没有指定端口，则使用 UDP 的 514 端口。如果域名解析为多个 IP 地址，则使用第一个地址。

<!--more -->

```
facility=string
```

> 设置 syslog 的消息 facility（设备），[RFC3164](https://tools.ietf.org/html/rfc3164#section-4.1.1) 中定义，facility可以是 `kern`，`user`，`mail`，`daemon`，`auth`，`intern`，`lpr`，`news`，`uucp`，`clock`，`authpriv`，`ftp`，`ntp`，`audit`，`alert`，`cron`，`local0`，`local7` 中的一个，默认是 `local7`。

```
severity=string
```

> 设置 [access_log](http://nginx.org/en/docs/http/ngx_http_log_module.html#access_log) 的消息严重程度，在 [RFC3164](https://tools.ietf.org/html/rfc3164#section-4.1.1) 中定义。可能值与 [error_log](http://nginx.org/en/docs/ngx_core_module.html#error_log) 指令的第二个参数（ `level`，级别）相同，默认是 `info`。错误消息的严重程度由 nginx 确定，因此在 `error_log` 指令中将忽略该参数。

```
tag=string
```
> 设置 syslog 消息标签。默认是 `nginx`。

```
nohostname
```
> 禁止将 `hostname` 域添加到 syslog 的消息（1.9.7）头中。

syslog配置示例：

```nginx
error_log syslog:server=192.168.1.1 debug;

access_log syslog:server=unix:/var/log/nginx.sock,nohostname;
access_log syslog:server=[2001:db8::1]:12345,facility=local7,tag=nginx,severity=info combined;
```

记录日志到 syslog 的功能自从 1.7.2 版本开始可用。作为我们 [商业订阅](http://nginx.com/products/?_ga=2.80571039.986778370.1500745948-1890203964.1497190280) 的一部分，记录日志到 syslog 的功能从 1.5.3 开始可用

## 配置文件结构

nginx 是由配置文件中指定的指令控制模块组成。指令可分为简单指令和块指令。一个简单的指令是由空格分隔的名称和参数组成，并以分号 `;` 结尾。块指令具有与简单指令相同的结构，但不是以分号结尾，而是以大括号`{}`包围的一组附加指令结尾。如果块指令的大括号内部可以有其它指令，则称这个块指令为上下文（例如：[events](http://nginx.org/en/docs/ngx_core_module.html#events)，[http](http://nginx.org/en/docs/http/ngx_http_core_module.html#http)，[server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server) 和 [location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location)）

配置文件中被放置在任何上下文之外的指令都被认为是主上下文 [main](http://nginx.org/en/docs/ngx_core_module.html)。`events` 和 `http` 指令在主 `main` 上下文中，`server` 在 `http` 中，`location` 又在 `server` 中

井号 `#` 之后的行的内容被视为注释

为了让 nginx 重新读取配置文件，应将 `HUP` 信号发送给 Master 进程。Master 进程首先会检查配置文件的语法有效性，之后尝试应用新的配置，即打开日志文件和新的 socket。如果失败了，它会回滚更改并继续使用旧的配置。如果成功，它将启动新的 Worker 进程并向旧的 Worker 进程发送消息请求它们正常关闭。旧的 Worker 进程关闭监听 socket 并继续为旧的客户端服务，当所有旧的客户端被处理完成，旧的 Worker 进程将被关闭。

我们来举例说明一下。 假设 nginx 是在 FreeBSD 4.x 命令行上运行的

```bash
ps axw -o pid,ppid,user,%cpu,vsz,wchan,command | egrep '(nginx|PID)'
```

得到以下输出结果：

```bash
  PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1148 pause  nginx: master process /usr/local/nginx/sbin/nginx
33127 33126 nobody   0.0  1380 kqread nginx: worker process (nginx)
33128 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
33129 33126 nobody   0.0  1364 kqread nginx: worker process (nginx)
```

如果把 `HUP` 信号发送到 master 进程，输出的结果将会是：

```bash
 PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33129 33126 nobody   0.0  1380 kqread nginx: worker process is shutting down (nginx)
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
```

其中一个 PID 为 33129 的 worker 进程仍然在继续工作，过一段时间之后它退出了：

```bash
PID  PPID USER    %CPU   VSZ WCHAN  COMMAND
33126     1 root     0.0  1164 pause  nginx: master process /usr/local/nginx/sbin/nginx
33134 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33135 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
33136 33126 nobody   0.0  1368 kqread nginx: worker process (nginx)
```

## nginx 如何处理请求

- [基于名称的虚拟服务器](#name_based_virtual_servers)
- [如何使用未定义的 server 名称来阻止处理请求](#how_to_prevent_undefined_server_names)
- [基于名称和 IP 混合的虚拟服务器](#mixed_name_ip_based_servers)
- [一个简单的 PHP 站点配置](#simple_php_site_configuration)

<a id="name_based_virtual_servers"></a>

### 基于名称的虚拟服务器

nginx 首先决定哪个 `server` 应该处理请求，让我们从一个简单的配置开始，三个虚拟服务器都监听了 `*:80` 端口：

```nginx
server {
    listen      80;
    server_name example.org www.example.org;
    ...
}

server {
    listen      80;
    server_name example.net www.example.net;
    ...
}

server {
    listen      80;
    server_name example.com www.example.com;
    ...
}
```

在此配置中，nginx 仅检验请求的 header 域中的 `Host`，以确定请求应该被路由到哪一个 `server`。如果其值与任何的 `server` 名称不匹配，或者该请求根本不包含此 header 域，nginx 会将请求路由到该端口的默认 `server` 中。在上面的配置中，默认 `server` 是第一个（这是 nginx 的标准默认行为）。你也可以在 [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 指令中使用 `default_server` 参数，明确地设置默认的 `server`。

```nginx
server {
    listen      80 default_server;
    server_name example.net www.example.net;
    ...
}
```

> `default_server` 参数自 0.8.21 版本起可用。在早期版本中，应该使用 `default` 参数。

请注意，`default_server` 是 `listen port` 的属性，而不是 `server_name` 的。之后会有更多关于这方面的内容。

<a id="how_to_prevent_undefined_server_names"></a>

### 如何使用未定义的 server 名称来阻止处理请求

如果不允许没有 “Host” header 字段的请求，可以定义一个丢弃请求的 server：

```nginx
server {
    listen      80;
    server_name "";
    return      444;
}
```

这里的 `server` 名称设置为一个空字符串，会匹配不带 `Host` 的 header 域请求，nginx 会返回一个表示关闭连接的非标准代码 444。

> 自 0.8.48 版本开始，这是 `server` 名称的默认设置，因此可以省略 `server name ""`。在早期版本中，机器的主机名被作为 `server` 的默认名称。

<a id="mixed_name_ip_based_servers"></a>

### 基于名称和 IP 混合的虚拟服务器

让我们看看更加复杂的配置，其中一些虚拟服务器监听在不同的 IP 地址上监听：

```nginx
server {
    listen      192.168.1.1:80;
    server_name example.org www.example.org;
    ...
}

server {
    listen      192.168.1.1:80;
    server_name example.net www.example.net;
    ...
}

server {
    listen      192.168.1.2:80;
    server_name example.com www.example.com;
    ...
}
```

此配置中，nginx 首先根据 [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server) 块的 `listen` 指令检验请求的 IP 和端口。之后，根据与 IP 和端口相匹配的 `server` 块的 [server_name](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_name) 项对请求的“Host” header 域进行检验。如果找不到服务器的名称（server_name），请求将由 `default_server` 处理。例如，在 `192.168.1.1:80` 上收到的对 `www.example.com` 的请求将由 `192.168.1.1:80` 端口的 `default_server` （即第一个 server）处理，因为没有 `www.example.com` 在此端口上定义。

如上所述，`default_server` 是 `listen port` 的属性，可以为不同的端口定义不同的 `default_server`：

```nginx
server {
    listen      192.168.1.1:80;
    server_name example.org www.example.org;
    ...
}

server {
    listen      192.168.1.1:80 default_server;
    server_name example.net www.example.net;
    ...
}

server {
    listen      192.168.1.2:80 default_server;
    server_name example.com www.example.com;
    ...
}
```

<a id="simple_php_site_configuration"></a>

## Nginx 实战

我始终认为，各种开发工具的配置还是结合实战来讲述，会让人更易理解。

### Http 反向代理

我们先实现一个小目标：不考虑复杂的配置，仅仅是完成一个 http 反向代理。

`nginx.conf` 配置文件如下：

> **_注：`conf/nginx.conf` 是 nginx 的默认配置文件。你也可以使用 nginx -c 指定你的配置文件_**

```nginx
#运行用户
#user somebody;

#启动进程,通常设置成和cpu的数量相等
worker_processes  1;

#全局错误日志
error_log  D:/Tools/nginx-1.10.1/logs/error.log;
error_log  D:/Tools/nginx-1.10.1/logs/notice.log  notice;
error_log  D:/Tools/nginx-1.10.1/logs/info.log  info;

#PID文件，记录当前启动的nginx的进程ID
pid        D:/Tools/nginx-1.10.1/logs/nginx.pid;

#工作模式及连接数上限
events {
    worker_connections 1024;    #单个后台worker process进程的最大并发链接数
}

#设定http服务器，利用它的反向代理功能提供负载均衡支持
http {
    #设定mime类型(邮件支持类型),类型由mime.types文件定义
    include       D:/Tools/nginx-1.10.1/conf/mime.types;
    default_type  application/octet-stream;

    #设定日志
	log_format  main  '[$remote_addr] - [$remote_user] [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log    D:/Tools/nginx-1.10.1/logs/access.log main;
    rewrite_log     on;

    #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
    #必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
    sendfile        on;
    #tcp_nopush     on;

    #连接超时时间
    keepalive_timeout  120;
    tcp_nodelay        on;

	#gzip压缩开关
	#gzip  on;

    #设定实际的服务器列表
    upstream zp_server1{
        server 127.0.0.1:8089;
    }

    #HTTP服务器
    server {
        #监听80端口，80端口是知名端口号，用于HTTP协议
        listen       80;

        #定义使用www.xx.com访问
        server_name  www.helloworld.com;

		#首页
		index index.html

		#指向webapp的目录
		root D:\01_Workspace\Project\github\zp\SpringNotes\spring-security\spring-shiro\src\main\webapp;

		#编码格式
		charset utf-8;

		#代理配置参数
        proxy_connect_timeout 180;
        proxy_send_timeout 180;
        proxy_read_timeout 180;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarder-For $remote_addr;

        #反向代理的路径（和upstream绑定），location 后面设置映射的路径
        location / {
            proxy_pass http://zp_server1;
        }

        #静态文件，nginx自己处理
        location ~ ^/(images|javascript|js|css|flash|media|static)/ {
            root D:\01_Workspace\Project\github\zp\SpringNotes\spring-security\spring-shiro\src\main\webapp\views;
            #过期30天，静态文件不怎么更新，过期可以设大一点，如果频繁更新，则可以设置得小一点。
            expires 30d;
        }

        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status           on;
            access_log            on;
            auth_basic            "NginxStatus";
            auth_basic_user_file  conf/htpasswd;
        }

        #禁止访问 .htxxx 文件
        location ~ /\.ht {
            deny all;
        }

		#错误处理页面（可选择性配置）
		#error_page   404              /404.html;
		#error_page   500 502 503 504  /50x.html;
        #location = /50x.html {
        #    root   html;
        #}
    }
}
```

好了，让我们来试试吧：

1.  启动 webapp，注意启动绑定的端口要和 nginx 中的 `upstream` 设置的端口保持一致。
2.  更改 host：在 C:\Windows\System32\drivers\etc 目录下的 host 文件中添加一条 DNS 记录

```
127.0.0.1 www.helloworld.com
```

3.  启动前文中 startup.bat 的命令
4.  在浏览器中访问 www.helloworld.com，不出意外，已经可以访问了。

### 配置 HTTPS反向代理服务器

要配置 HTTPS 服务器，必须在 [server](http://nginx.org/en/docs/http/ngx_http_core_module.html#server) 块中的 [监听套接字](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 上启用 `ssl` 参数，并且指定[服务器证书](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate) 和 [私钥文件](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate_key) 的位置：

```nginx
server {
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ...
}
```

<!-- more -->

服务器证书是一个公共实体。它被发送到每个连接到服务器的客户端。私钥是一个安全实体，存储在一个访问受限的文件中，但是它对 nginx 的主进程必须是可读的。私钥也可以存储在与证书相同的文件中：
```nginx
ssl_certificate     www.example.com.cert;
ssl_certificate_key www.example.com.cert;
```

这种情况下，文件的访问也应该被限制。虽然证书和密钥存储在一个文件中，但只有证书能被发送给客户端。

可以使用 [ssl_protocols](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols) 和 [ssl_ciphers](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_ciphers) 指令来限制连接，使其仅包括 SSL/TLS 的版本和密码。默认情况下，nginx 使用版本为 `ssl_protocols TLSv1 TLSv1.1 TLSv1.2`，密码为 `ssl_ciphers HIGH:!aNULL:!MD5`，因此通常不需要配置它们。请注意，这些指令的默认值已经被 [更改](#兼容性) 多次。

#### HTTPS 服务器优化
SSL 操作会消耗额外的 CPU 资源。在多处理器系统上，应该运行多个 [工作进程（worker process）](http://nginx.org/en/docs/ngx_core_module.html#worker_processes)，不得少于可用 CPU 核心的数量。大多数 CPU 密集型操作是发生在 SSL 握手时。有两种方法可以最大程度地减少每个客户端执行这些操作的次数。首先，启用 [keepalive](http://nginx.org/en/docs/http/ngx_http_core_module.html#keepalive_timeout) 连接，通过一个连接来发送多个请求，第二个是复用 SSL 会话参数，避免相同的和后续的连接发生 SSL 握手。会话存储在工作进程间共享的 SSL 会话缓存中，由 [ssl_session_cache](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_cache) 指令配置。1MB 缓存包含约 4000 个会话。默认缓存超时时间为 5 分钟，可以用 [ssl_session_timeout](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_timeout) 指令来增加。以下是一个优化具有 10MB 共享会话缓存的多核系统的配置示例：

```nginx
worker_processes auto;

http {
    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 10m;

    server {
        listen              443 ssl;
        server_name         www.example.com;
        keepalive_timeout   70;

        ssl_certificate     www.example.com.crt;
        ssl_certificate_key www.example.com.key;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;
        ...
```

#### SSL 证书链
某些浏览器可能会不承认由知名证书颁发机构签发的证书，而其他浏览器可能会接受该证书。之所以发生这种情况，是因为颁发机构已经在特定浏览器分发了一个中间证书，该证书不存在于知名可信证书颁发机构的证书库中。在这种情况下，权威机构提供了一系列链式证书，这些证书应该与已签名的服务器证书相连。服务器证书必须出现在组合文件中的链式证书之前：

```
$ cat www.example.com.crt bundle.crt > www.example.com.chained.crt
```

在 [ssl_certificate](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate) 指令中使用生成的文件：

```nginx
server {
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.chained.crt;
    ssl_certificate_key www.example.com.key;
    ...
}
```

如果服务器证书与捆绑的链式证书的相连顺序错误，nginx 将无法启动并显示错误消息：

```bash
SSL_CTX_use_PrivateKey_file(" ... /www.example.com.key") failed
   (SSL: error:0B080074:x509 certificate routines:
    X509_check_private_key:key values mismatch)
```

因为 nginx 已尝试使用私钥和捆绑的第一个证书而不是服务器证书。

浏览器通常会存储他们收到的中间证书，这些证书由受信任的机构签名，因此积极使用这些存储证书的浏览器可能已经具有所需的中间证书，不会发生不承认没有链接捆绑发送的证书的情况。可以使用 openssl 命令行工具来确保服务器发送完整的证书链，例如：

```bash
$ openssl s_client -connect www.godaddy.com:443
...
Certificate chain
 0 s:/C=US/ST=Arizona/L=Scottsdale/1.3.6.1.4.1.311.60.2.1.3=US
     /1.3.6.1.4.1.311.60.2.1.2=AZ/O=GoDaddy.com, Inc
     /OU=MIS Department/CN=www.GoDaddy.com
     /serialNumber=0796928-7/2.5.4.15=V1.0, Clause 5.(b)
   i:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
     /OU=http://certificates.godaddy.com/repository
     /CN=Go Daddy Secure Certification Authority
     /serialNumber=07969287
 1 s:/C=US/ST=Arizona/L=Scottsdale/O=GoDaddy.com, Inc.
     /OU=http://certificates.godaddy.com/repository
     /CN=Go Daddy Secure Certification Authority
     /serialNumber=07969287
   i:/C=US/O=The Go Daddy Group, Inc.
     /OU=Go Daddy Class 2 Certification Authority
 2 s:/C=US/O=The Go Daddy Group, Inc.
     /OU=Go Daddy Class 2 Certification Authority
   i:/L=ValiCert Validation Network/O=ValiCert, Inc.
     /OU=ValiCert Class 2 Policy Validation Authority
     /CN=http://www.valicert.com//emailAddress=info@valicert.com
...
```

在本示例中，www.GoDaddy.com 服务器证书 ＃0 的主体（“s”）由发行机构（“i”）签名，发行机构本身是证书 ＃1 的主体，是由知名发行机构 ValiCert, Inc. 签署的证书 ＃2 的主体，其证书存储在浏览器的内置证书库中。

如果没有添加证书包（certificate bundle），将仅显示服务器证书 ＃0。

#### HTTP/HTTPS 服务器
可以配置单个服务器来处理 HTTP 和 HTTPS 请求：

```nginx
server {
    listen              80;
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ...
}
```

> 在 0.7.14 之前，无法为各个 socket 选择性地启用 SSL，如上所示。只能使用 [ssl](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl) 指令为整个服务器启用 SSL，从而无法设置单个 HTTP/HTTPS 服务器。可以通过添加 [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 指令的 ssl 参数来解决这个问题。因此，不建议在现在的版本中使用 [ssl](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl) 指令。

#### 基于名称的 HTTPS 服务器
当配置两个或多个 HTTPS 服务器监听单个 IP 地址时，会出现一个常见问题：

```nginx
server {
    listen          443 ssl;
    server_name     www.example.com;
    ssl_certificate www.example.com.crt;
    ...
}

server {
    listen          443 ssl;
    server_name     www.example.org;
    ssl_certificate www.example.org.crt;
    ...
}
```

使用了此配置，浏览器会接收默认服务器的证书，即 www.example.com，而无视所请求的服务器名称。这是由 SSL 协议行为引起的。SSL连接在浏览器发送 HTTP 请求之前建立，nginx 并不知道请求的服务器名称。因此，它只能提供默认服务器的证书。

最古老、最强大的解决方法是为每个 HTTPS 服务器分配一个单独的 IP 地址：

```nginx
server {
    listen          192.168.1.1:443 ssl;
    server_name     www.example.com;
    ssl_certificate www.example.com.crt;
    ...
}

server {
    listen          192.168.1.2:443 ssl;
    server_name     www.example.org;
    ssl_certificate www.example.org.crt;
    ...
}
```

#### 具有多个名称的 SSL 证书
虽然还有其他方法允许在多个 HTTPS 服务器之间共享一个 IP 地址。但他们都有自己的缺点。一种方法是在 SubjectAltName 证书字段中使用多个名称的证书，例如 `www.example.com` 和 `www.example.org`。但是，SubjectAltName 字段长度有限。

另一种方法是使用附带通配符名称的证书，例如 `*.example.org`。通配符证书保护指定域的所有子域，但只能在同一级别上。此证书能匹配 `www.example.org`，但与 `example.org` 和 `www.sub.example.org` 不匹配。当然，这两种方法也可以组合使用的。SubjectAltName 字段中的证书可能包含确切名称和通配符名称，例如 `example.org` 和 `*.example.org`。

最好是将证书文件与名称、私钥文件放置在 http 级配置，以便在所有服务器中继承其单个内存副本：

```nginx
ssl_certificate     common.crt;
ssl_certificate_key common.key;

server {
    listen          443 ssl;
    server_name     www.example.com;
    ...
}

server {
    listen          443 ssl;
    server_name     www.example.org;
    ...
```

#### 服务器名称指示
在单个 IP 地址上运行多个 HTTPS 服务器的更通用的解决方案是 [TLS 服务器名称指示扩展](http://en.wikipedia.org/wiki/Server_Name_Indication)（SNI，RFC 6066），其允许浏览器在 SSL 握手期间传递所请求的服务器名称，因此，服务器将知道应该为此连接使用哪个证书。然而，SNI 对浏览器的支持是有限的。目前，它仅支持以下版本开始的浏览器：

- Opera 8.0
- MSIE 7.0（但仅在 Windows Vista 或更高版本）
- Firefox 2.0 和使用 Mozilla Platform rv:1.8.1 的其他浏览器
- Safari 3.2.1（支持 SNI 的 Windows 版本需要 Vista 或更高版本）
- Chrome（支持 SNI 的 Windows 版本需要 Vista 或更高版本）

> 只有域名可以在 SNI 中传递，然而，如果请求包含 IP 地址，某些浏览器可能会错误地将服务器的 IP 地址作为名称传递。不应该依靠这个。

要在 nginx 中使用 SNI，其必须支持构建后的 nginx 二进制的 OpenSSL 库以及在可在运行时动态链接的库。自 0.9.8f 版本起（OpenSSL），如果 OpenSSL 使用了配置选项 `--enable-tlsext` 构建，是支持 SNI 的。自 OpenSSL 0.9.8j 起，此选项是默认启用。如果 nginx 是用 SNI 支持构建的，那么当使用 `nginx -V` 命令时 ，nginx 会显示：

```bash
$ nginx -V
...
TLS SNI support enabled
...
```

但是，如果启用了 SNI 的 nginx 在没有 SNI 支持的情况下动态链接到 OpenSSL 库，那么 nginx 将会显示警告：

```bash
nginx was built with SNI support, however, now it is linked
dynamically to an OpenSSL library which has no tlsext support,
therefore SNI is not available
```

#### 兼容性
- 从 0.8.21 和 0.7.62 起，SNI 支持状态通过 `-V `开关显示。
- 从 0.7.14 开始，支持 listen 指令的 ssl 参数。在 0.8.21 之前，只能与 `default` 参数一起指定。
- 从 0.5.23 起，支持 SNI。
- 从 0.5.6 起，支持共享 SSL 会话缓存。
- 1.9.1 及更高版本：默认 SSL 协议为 TLSv1、TLSv1.1 和 TLSv1.2（如果 OpenSSL 库支持）。
- 0.7.65、0.8.19 及更高版本：默认 SSL 协议为 SSLv3、TLSv1、TLSv1.1 和 TLSv1.2（如果 OpenSSL 库支持）。
- 0.7.64、0.8.18 及更早版本：默认 SSL 协议为 SSLv2、SSLv3 和 TLSv1。
- 1.0.5及更高版本：默认 SSL 密码为 `HIGH:!aNULL:!MD5`。
- 0.7.65、0.8.20 及更高版本：默认 SSL 密码为 `HIGH:!ADH:!MD5`。
- 0.8.19 版本：默认 SSL 密码为 `ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM`。
- 0.7.64、0.8.18 及更早版本：默认SSL密码为 `ALL:!ADH:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP`。

由 **Igor Sysoev** 撰写，**Brian Mercer** 编辑

#### 原文档

- [Configuring HTTPS servers](http://nginx.org/en/docs/http/configuring_https_servers.html)

### 负载均衡

前面的例子中，代理仅仅指向一个服务器

但是，网站在实际运营过程中，大部分都是以集群的方式运行，这时需要使用负载均衡来分流，以优化资源利用率、最大化吞吐量、减少延迟和确保容错配置。

可以使用 nginx 作为高效的 HTTP 负载均衡器，将流量分布到多个应用服务器，并通过 nginx 提高 web 应用程序的性能、可扩展性和可靠性

![img](https://raw.githubusercontent.com/dunwu/images/master/cs/web/nginx/nginx-load-balance.png)

假设这样一个应用场景：将应用部署在 192.168.1.11:80、192.168.1.12:80、192.168.1.13:80 三台 linux 环境的服务器上。网站域名叫 www.helloworld.com，公网 IP 为 192.168.1.11。在公网 IP 所在的服务器上部署 nginx，对所有请求做负载均衡处理（下面例子中使用的是加权轮询策略）。

nginx.conf 配置如下：

```nginx
http {
     #设定mime类型,类型由mime.type文件定义
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    #设定日志格式
    access_log    /var/log/nginx/access.log;

    #设定负载均衡的服务器列表
    upstream load_balance_server {
        #weigth参数表示权值，权值越高被分配到的几率越大
        server 192.168.1.11:80   weight=5;
        server 192.168.1.12:80   weight=1;
        server 192.168.1.13:80   weight=6;
    }

   #HTTP服务器
   server {
        #侦听80端口
        listen       80;

        #定义使用www.xx.com访问
        server_name  www.helloworld.com;

        #对所有请求进行负载均衡请求
        location / {
            root        /root;                 #定义服务器的默认网站根目录位置
            index       index.html index.htm;  #定义首页索引文件的名称
            proxy_pass  http://load_balance_server ;#请求转向load_balance_server 定义的服务器列表

            #以下是一些反向代理的配置(可选择性配置)
            #proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            #后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_connect_timeout 90;          #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_send_timeout 90;             #后端服务器数据回传时间(代理发送超时)
            proxy_read_timeout 90;             #连接成功后，后端服务器响应时间(代理接收超时)
            proxy_buffer_size 4k;              #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            proxy_buffers 4 32k;               #proxy_buffers缓冲区，网页平均在32k以下的话，这样设置
            proxy_busy_buffers_size 64k;       #高负荷下缓冲大小（proxy_buffers*2）
            proxy_temp_file_write_size 64k;    #设定缓存文件夹大小，大于这个值，将从upstream服务器传

            client_max_body_size 10m;          #允许客户端请求的最大单文件字节数
            client_body_buffer_size 128k;      #缓冲区代理缓冲用户端请求的最大字节数
        }
    }
}
```

#### 负载均衡策略

Nginx 提供了多种负载均衡策略

负载均衡策略在各种分布式系统中基本上原理一致，对于原理有兴趣，不妨参考 [负载均衡](https://dunwu.github.io/blog/pages/98a1c1/)

##### 轮询（默认负载均衡配置）

```nginx
upstream bck_testing_01 {
  # 默认所有服务器权重为 1
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080
}
```

##### 加权轮询

```nginx
upstream bck_testing_01 {
  server 192.168.250.220:8080   weight=3
  server 192.168.250.221:8080              # default weight=1
  server 192.168.250.222:8080              # default weight=1
}
```

##### 最少连接

```nginx
upstream bck_testing_01 {
  least_conn;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080
}
```

##### 加权最少连接

```nginx
upstream bck_testing_01 {
  least_conn;

  server 192.168.250.220:8080   weight=3
  server 192.168.250.221:8080              # default weight=1
  server 192.168.250.222:8080              # default weight=1
}
```

##### IP Hash

**会话持久化**

请注意，使用轮询或者最少连接的负载均衡，每个后续客户端的请求都可能被分配到不同的服务器。不能保证同一个客户端始终指向同一个服务器。

如果需要将客户端绑定到特定的应用服务器，换而言之，使客户端会话「粘滞」或者「永久」，始终尝试选择特定的服务器，IP 哈希负载均衡机制可以做到这点。

使用 IP 哈希，客户端的 IP 地址用作为哈希键，以确定应用为客户端请求选择服务器组中的哪个服务器。此方法确保了来自同一个客户端的请求始终被定向到同一台服务器，除非该服务器不可用

```nginx
upstream bck_testing_01 {

  ip_hash;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080

}
```

##### 普通 Hash

```nginx
upstream bck_testing_01 {

  hash $request_uri;

  # with default weight for all (weight=1)
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080

}
```

#### 健康检查

nginx 中的反向代理实现包括了带内（或者被动）服务器健康检查。如果特定服务器的响应失败并出现错误，则 nginx 会将此服务器标记为失败，并尝试避免为此后续请求选择此服务器而浪费一段时间。

max_fails 用于设置在 fail_timeout 期间与服务器通信失败重新尝试的次数。默认情况下，max_fails 设置为 1。当设置为 0 时，该服务器的健康检查将被禁用。fail_timeout 参数还定义了服务器被标记为失败的时间。在服务器发生故障后的 fail_timeout 间隔后，nginx 开始以实时客户端的请求优雅地探测服务器。如果探测成功，则将服务器标记为活动

### 网站有多个 webapp 的配置

当一个网站功能越来越丰富时，往往需要将一些功能相对独立的模块剥离出来，独立维护。这样的话，通常，会有多个 webapp。

举个例子：假如 www.helloworld.com 站点有好几个 webapp，finance（金融）、product（产品）、admin（用户中心）。访问这些应用的方式通过上下文(context)来进行区分:

www.helloworld.com/finance/

www.helloworld.com/product/

www.helloworld.com/admin/

我们知道，http 的默认端口号是 80，如果在一台服务器上同时启动这 3 个 webapp 应用，都用 80 端口，肯定是不成的。所以，这三个应用需要分别绑定不同的端口号。

那么，问题来了，用户在实际访问 www.helloworld.com 站点时，访问不同 webapp，总不会还带着对应的端口号去访问吧。所以，你再次需要用到反向代理来做处理。

配置也不难，来看看怎么做吧：

```nginx
http {
	#此处省略一些基本配置

	upstream product_server{
		server www.helloworld.com:8081;
	}

	upstream admin_server{
		server www.helloworld.com:8082;
	}

	upstream finance_server{
		server www.helloworld.com:8083;
	}

	server {
		#此处省略一些基本配置
		#默认指向product的server
		location / {
			proxy_pass http://product_server;
		}

		location /product/{
			proxy_pass http://product_server;
		}

		location /admin/ {
			proxy_pass http://admin_server;
		}

		location /finance/ {
			proxy_pass http://finance_server;
		}
	}
}
```

### 提供静态内容服务

Web 服务器的一个重要任务是提供文件（比如图片或者静态 HTML 页面）服务。您将实现一个示例，根据请求，将提供来自不同的本地目录的文件： `/data/www`（可能包含 HTML 文件）和 `/data/images`（包含图片）。这需要编辑配置文件，在 `http` 中配置一个包含两个 `location` 块的 `server` 块指令。

首先，创建 `/data/www` 目录将包含任何文本内容的 `index.html` 文件放入其中，创建 `/data/images` 目录然后放一些图片进去。

其次，打开这个配置文件， 默认配置文件已经包含几个服务器块示例，大部分是已经注释掉的。现在注释掉这些块并且启动一个新的 `server` 块。

```
http {
    server {
    }
}
```

通常，配置文件可以包含几个由监听 [listen](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen) 端口和服务器域名 [server names](http://nginx.org/en/docs/http/server_names.html) 区分的 `server` 块指令 [distinguished](http://nginx.org/en/docs/http/request_processing.html)。一旦 nginx 决定由哪个 `server` 来处理请求，它会根据 `server` 块中定义的 `location` 指令的参数来检验请求头中指定的URI。

添加如下 `location` 块指令到 `server` 块指令中：

```
location / {
    root /data/www;
}
```

该 `location` 块指令指定 `/` 前缀与请求中的 URI 相比较。对于匹配的请求，URI 将被添加到根指令 [root](http://nginx.org/en/docs/http/ngx_http_core_module.html#root) 中指定的路径，即 `/data/ www`，以形成本地文件系统上所请求文件的路径。如果有几个匹配上的 `location` 块指令，nginx 将选择具有最长前缀的 `location` 块。上面的位置块提供最短的前缀，长度为 1，因此只有当所有其它 `location` 块不能匹配时，才会使用该块。

接下来，添加第二个 `location` 指令快：

```
location /images/ {
    root /data;
}
```

以 `/images/` 为开头的请求将会被匹配上（虽然 `location /` 也能匹配上此请求，但是它的前缀更短）

最后，`server` 块指令应如下所示：

```
server {
    location / {
        root /data/www;
    }

    location /images/ {
        root /data;
    }
}
```

这已经是一个监听标准 80 端口并且可以在本地机器上通过 `http://localhost/` 地址来访问的有效配置。响应以 `/images/` 开头的URI请求，服务器将从 `/data/images` 目录发送文件。例如，响应`http://localhost/images/example.png` 请求，nginx 将发送 `/data/images/example.png` 文件。如果此文件不存在，nginx 将发送一个404错误响应。不以 `/ images/` 开头的 URI 的请求将映射到 `/data/www` 目录。例如，响应 `http://localhost/some/example.html` 请求，nginx 将发送 `/data/www/some/example.html` 文件。

要让新配置立刻生效，如果 nginx 尚未启动可以启动它，否则通过执行以下命令将重新加载配置信号发送到 nginx 的主进程：

```
nginx -s reload
```

> 如果运行的效果没有在预期之中，您可以尝试从 `/usr/local/nginx/logs` 或 `/var/log/ nginx` 中的 `access.log` 和 `error.log` 日志文件中查找原因。

#### 设置哈希

为了快速处理静态数据集，如服务器名称、[map](http://nginx.org/en/docs/http/ngx_http_map_module.html#map) 指令值、MIME 类型、请求头字符串的名称，nginx 使用了哈希表。在开始和每次重新配置时，nginx 尽可能选择最小的哈希表，以便存储具有相同散列值的存储桶大小不会超过配置参数（哈希桶大小）。表的大小以桶为单位。调整是持续的，直到表的大小超过哈希的最大大小参数。大多数哈希具有修改这些参数对应的指令，例如，对于服务器名称哈希，它们是 [server_names_hash_max](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_max_size) 和 [server_names_hash_bucket_size](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_names_hash_bucket_size)。

哈希桶大小参数与处理器的高速缓存线大小的倍数对齐。可以通过减少内存访问的次数来加快现代处理器的哈希键搜索速度。如果哈希桶大小等于处理器的高速缓存线大小，则哈希键搜索期间内存访问次数最坏的情况下将有两次 —— 一是计算桶地址，二是在桶内搜索哈希键期间。因此，如果 nginx 发出消息请求增加到哈希的最大大小或者哈希桶的大小，那么首先应该增加第一个参数。

#### 案例一：设置一个简单的代理服务器

nginx 的一个常见用途是作为一个代理服务器，作用是接收请求并转发给被代理的服务器，从中取得响应，并将其发送回客户端。

我们将配置一个基本的代理服务器，它为图片请求提供的文件来自本地目录，并将所有其它请求发送给代理的服务器。在此示例中，两个服务器在单个 nginx 实例上定义。

首先，通过向 nginx 的配置文件添加一个 `server` 块来定义代理服务器，其中包含以下内容：

```
server {
    listen 8080;
    root /data/up1;

    location / {
    }
}
```

这是一个监听 8080 端口的简单服务器（以前，由于使用了标准 80 端口，所以没有指定 listen 指令），并将所有请求映射到本地文件系统上的 `/data/up1` 目录。创建此目录并将 `index.html` 文件放入其中。请注意，`root` 指令位于 `server` 上下文中。当选择用于处理请求的 `location` 块自身不包含 `root` 指令时，将使用此 `root` 指令

接下来，在上一节中的服务器配置基础上进行修改，使其成为代理服务器配置。在第一个 `location` 块中，使用参数指定的代理服务器的协议，域名和端口（在本例中为 `http://localhost:8080`）放置在 [proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) 指令处

```
server {
    location / {
        proxy_pass http://localhost:8080;
    }

    location /images/ {
        root /data;
    }
}
```

我们将修改使用了 `/images/` 前缀将请求映射到 `/data/images` 目录下的文件的第二个 `location` 块，使其与附带常见的图片文件扩展名的请求相匹配。修改后的 `location` 块如下所示：

```
location ~ \.(gif|jpg|png)$ {
    root /data/images;
}
```

该参数是一个正则表达式，匹配所有以`.gif`，`.jpg` 或 `.png` 结尾的 URI。正则表达式之前应该是 `~`。相应的请求将映射到 `/data/images` 目录。

当 nginx 选择一个 `location` 块来提供请求时，它首先检查指定前缀的 [location](http://nginx.org/en/docs/http/ngx_http_core_module.html#location) 指令，记住具有最长前缀的 `location`，然后检查正则表达式。如果与正则表达式匹配，nginx 会选择此 `location`，否则选择更早之前记住的那一个。

代理服务器的最终配置如下：

```
server {
    location / {
        proxy_pass http://localhost:8080/;
    }

    location ~ \.(gif|jpg|png)$ {
        root /data/images;
    }
}
```

此 `server` 将过滤以 `.gif`，`.jpg` 或 `.png` 结尾的请求，并将它们映射到 `/data/images` 目录（通过向 root 指令的参数添加 URI），并将所有其它请求传递到上面配置的代理服务器

要使新配置立即生效，请将重新加载配置文件信号（reload）发送到 nginx

#### 案例二：访问静态资源

举例来说：如果所有的静态资源都放在了 `/app/dist` 目录下，我们只需要在 `nginx.conf` 中指定首页以及这个站点的 host 即可

配置如下：

```nginx
worker_processes  1;

events {
	worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    gzip on;
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/javascript image/jpeg image/gif image/png;
    gzip_vary on;

    server {
		listen       80;
		server_name  static.zp.cn;

		location / {
			root /app/dist;
			index index.html;
			#转发任何请求到 index.html
		}
	}
}
```

然后，添加 HOST：

127.0.0.1 static.zp.cn

此时，在本地浏览器访问 static.zp.cn ，就可以访问静态站点了

### 搭建文件服务器

有时候，团队需要归档一些数据或资料，那么文件服务器必不可少。使用 Nginx 可以非常快速便捷的搭建一个简易的文件服务。

Nginx 中的配置要点：

- 将 autoindex 开启可以显示目录，默认不开启。
- 将 autoindex_exact_size 开启可以显示文件的大小。
- 将 autoindex_localtime 开启可以显示文件的修改时间。
- root 用来设置开放为文件服务的根路径。
- charset 设置为 `charset utf-8,gbk;`，可以避免中文乱码问题（windows 服务器下设置后，依然乱码，本人暂时没有找到解决方法）。

一个最简化的配置如下：

```nginx
autoindex on;# 显示目录
autoindex_exact_size on;# 显示文件大小
autoindex_localtime on;# 显示文件时间

server {
    charset      utf-8,gbk; # windows 服务器下设置后，依然乱码，暂时无解
    listen       9050 default_server;
    listen       [::]:9050 default_server;
    server_name  _;
    root         /share/fs;
}
```

### nginx 如何处理 TCP/UDP 会话

来自客户端的 TCP/UDP 会话以阶段的形式被逐步处理：

|阶 段&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|描 述|
|:----|:----|
| Post-accept | 接收客户端请求后的第一个阶段。[ngx_stream_realip_module](http://nginx.org/en/docs/stream/ngx_stream_realip_module.html) 模块在此阶段被调用。|
| Pre-access | 初步检查访问，[ngx_stream_limit_conn_module](http://nginx.org/en/docs/stream/ngx_stream_limit_conn_module.html) 模块在此阶段被调用。 |
| Access | 实际处理之前的客户端访问限制，ngx_stream_access_module 模块在此阶段被调用。 |
| SSL | TLS/SSL 终止，ngx_stream_ssl_module 模块在此阶段被调用。 |
| Preread | 将数据的初始字节读入 [预读缓冲区](http://nginx.org/en/docs/stream/ngx_stream_core_module.html#preread_buffer_size) 中，以允许如 [ngx_stream_ssl_preread_module](http://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html) 之类的模块在处理前分析数据。 |
| Content | 实际处理数据的强制阶段，通常 [代理](http://nginx.org/en/docs/stream/ngx_stream_proxy_module.html) 到 [upstream](http://nginx.org/en/docs/stream/ngx_stream_upstream_module.html) 服务器，或者返回一个特定的值给客户端 |
| Log | 此为最后阶段，客户端会话处理结果将被记录， [ngx_stream_log_module module](http://nginx.org/en/docs/stream/ngx_stream_log_module.html) 模块在此阶段被调用。 |

### 解决跨域

web 领域开发中，经常采用前后端分离模式。这种模式下，前端和后端分别是独立的 web 应用程序，例如：后端是 Java 程序，前端是 React 或 Vue 应用。

各自独立的 web app 在互相访问时，势必存在跨域问题。解决跨域问题一般有两种思路：

1.  **CORS**

在后端服务器设置 HTTP 响应头，把你需要允许访问的域名加入 `Access-Control-Allow-Origin` 中。

2.  **jsonp**

把后端根据请求，构造 json 数据，并返回，前端用 jsonp 跨域。

这两种思路，本文不展开讨论。

需要说明的是，nginx 根据第一种思路，也提供了一种解决跨域的解决方案。

举例：www.helloworld.com 网站是由一个前端 app ，一个后端 app 组成的。前端端口号为 9000， 后端端口号为 8080。

前端和后端如果使用 http 进行交互时，请求会被拒绝，因为存在跨域问题。来看看，nginx 是怎么解决的吧：

首先，在 enable-cors.conf 文件中设置 cors ：

```nginx
# allow origin list
set $ACAO '*';

# set single origin
if ($http_origin ~* (www.helloworld.com)$) {
  set $ACAO $http_origin;
}

if ($cors = "trueget") {
	add_header 'Access-Control-Allow-Origin' "$http_origin";
	add_header 'Access-Control-Allow-Credentials' 'true';
	add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
	add_header 'Access-Control-Allow-Headers' 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
}

if ($request_method = 'OPTIONS') {
  set $cors "${cors}options";
}

if ($request_method = 'GET') {
  set $cors "${cors}get";
}

if ($request_method = 'POST') {
  set $cors "${cors}post";
}
```

接下来，在你的服务器中 `include enable-cors.conf` 来引入跨域配置：

```nginx
# ----------------------------------------------------
# 此文件为项目 nginx 配置片段
# 可以直接在 nginx config 中 include（推荐）
# 或者 copy 到现有 nginx 中，自行配置
# www.helloworld.com 域名需配合 dns hosts 进行配置
# 其中，api 开启了 cors，需配合本目录下另一份配置文件
# ----------------------------------------------------
upstream front_server{
  server www.helloworld.com:9000;
}
upstream api_server{
  server www.helloworld.com:8080;
}

server {
  listen       80;
  server_name  www.helloworld.com;

  location ~ ^/api/ {
    include enable-cors.conf;
    proxy_pass http://api_server;
    rewrite "^/api/(.*)$" /$1 break;
  }

  location ~ ^/ {
    proxy_pass http://front_server;
  }
}
```

到此，就完成了。

## 资源

- [Nginx 的中文维基](http://tool.oschina.net/apidocs/apidoc?api=nginx-zh)
- [Nginx 开发从入门到精通](http://tengine.taobao.org/book/index.html)
- [nginx-admins-handbook](https://github.com/trimstray/nginx-admins-handbook)
- [nginxconfig.io](https://nginxconfig.io/) - 一款 Nginx 配置生成器
