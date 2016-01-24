[原文](https://www.nginx.com/resources/wiki/start/topics/tutorials/config_pitfalls/)

新老用户都可能遇到陷阱。下面我们列出频繁出现的问题，以及如何解决。
在 Freenode IRC #nginx 频道，我们经常看到这些问题。

 [TOC]

## 关于本指南

最常见的是有人试图从其他指南拷贝配置片段。并非所有的指南是错误的，但绝大部分是有问题的。
甚至 Linode library 的质量也不高，NGINX 社区成员有义务去尝试更正。

这些文档由社区成员创建并审核。存在该指南的意义在于社区成员常见及反复出现的问题。

## 我的问题未列出

你遇到的问题在这里没有看到，也许我们在这里没有指明你经历的问题。不要以为你无缘无故的浏览到该
页面，你看到这些是因为你做错的事情列在这里。

当涉及到支持许多用户的很多问题，社区成员不想去支持未遵守约定的配置。通过阅读以下指南，修复你的配置。

## Chmod 777

绝不使用 777。它可能是个漂亮的号码，即使在测试中也表现的你没有任何线索在做什么。在整个路径中
查看权限，想想会发生什么。

轻松的显示路径权限，可以使用：

```
namei -om /path/to/check
```

## Root 在 Location 区块

**BAD**

```nginx
server {
    server_name www.example.com;
    location / {
        root /var/www/nginx-default/;
        # [...]
      }
    location /foo {
        root /var/www/nginx-default/;
        # [...]
    }
    location /bar {
        root /var/www/nginx-default/;
        # [...]
    }
}
```
这有效。把*root*放在*location*区块可以生效而且完全有效。错误之处在于开始添加*location*区块，
如果你在每个*location*区块添加*root*指令，没有匹配*location*区块将没有*root*。下面看看正确的配置。

**GOOD**

```nginx
server {
    server_name www.example.com;
    root /var/www/nginx-default/;
    location / {
        # [...]
    }
    location /foo {
        # [...]
    }
    location /bar {
        # [...]
    }
}
```

## 多重 Index 指令

**BAD**

```nginx
http {
    index index.php index.htm index.html;
    server {
        server_name www.example.com;
        location / {
            index index.php index.htm index.html;
            # [...]
        }
    }
    server {
        server_name example.com;
        location / {
            index index.php index.htm index.html;
            # [...]
        }
        location /foo {
            index index.php;
            # [...]
        }
    }
}
```
为什么在不需要时重复那么多行。简单的使用一次 “index” 指令。仅仅需要添加到*http{}*区块，将会被继承。

**GOOD**

```nginx
http {
    index index.php index.htm index.html;
    server {
        server_name www.example.com;
        location / {
            # [...]
        }
    }
    server {
        server_name example.com;
        location / {
            # [...]
        }
        location /foo {
            # [...]
        }
    }
}
```

## 使用 If

有几个页面介绍使用 if 语句。它叫做*IfIsEvil*，你真的应该看看。让我们看看使用 if 的坏处。

**See also**

[If Is Evil](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/)

## Server Name (if)

**BAD**

```nginx
server {
    server_name example.com *.example.com;
        if ($host ~* ^www\.(.+)) {
            set $raw_domain $1;
            rewrite ^/(.*)$ $raw_domain/$1 permanent;
        }
        # [...]
    }
}
```
这里有3个问题。第一个是*if*，我们关心的。当 NGINX 接受一个请求不管是子域名 www.example.com
还是 example.com，*if*指令总是会判断。因为你要求 NGINX 检查每个请求的 *Host* 头。
这样效率低下，你应该避免。像下面一样使用2个*server*指令。

**GOOD**

```nginx
server {
    server_name www.example.com;
    return 301 $scheme://example.com$request_uri;
}
server {
    server_name example.com;
    # [...]
}
```
除了配置文件更易于阅读，这种方法减少 NGINX 处理需求。我们摆脱了虚假的**if**，同时使用
`$scheme`,没有硬编码 URI 的结构，可以是 http 或者 https。

## 检查文件 (if) 存在

使用 if 去检查文件是否存在是糟糕的。如果你使用最新版本的 NGINX ，应该看看`try_files`,更
易使用。

**BAD**

```nginx
server {
    root /var/www/example.com;
    location / {
        if (!-f $request_filename) {
            break;
        }
    }
}
```

**GOOD**

```nginx
server {
    root /var/www/example.com;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

这里我们判断`$uri`是否存在不再需要`if`。使用`try_files`意味着可以依序测试。
如果`$uri`不存在，尝试`$uri/`，如果还不存在，尝试备用的*location*。

在这个案例中，会先查看`$uri`文件是否存在，存在就返回。如果不存在，就检查该目录是否存在。
如果不存在，将返回`index.html`,你必须确定存在该文件。

## Front Controller 模式 WEB 应用

“Front Controller 模式”设计是受欢迎的，且用在许多最流行的 PHP 软件包中。
很多实例比应有的更复杂。让  Drupal, Joomla 等运行，使用：

```nginx
try_files $uri $uri/ /index.php?q=$uri&$args;
```

注解 - 参数名称根据你使用的包是不同的。例如

- “q” 是 Drupal、Joomla、WordPress 使用的参数
- “page” 是 CMS Made Simple 使用的参数

一些软件甚至不用查询字符串，而且可以从`REQUEST_URI`读取（WordPress支持，例如）：

```nginx
try_files $uri $uri/ /index.php;
```

当然你的情况可能不同，你可能根据需求添加复杂的规则，但一个基本的网站，这些将完美的工作。
你可以从简单的开始。

如果你不在乎目录是否存在，你也可以决定跳过目录检查，从中删除`$uri/`。

## 把不受控制的请求给 PHP

很多对 PHP 的 NGINX 配置样例把每个以`.php`结尾的 URI 给 PHP 解释器。请注意在大多数 PHP
设置中，这将是个严重的安全问题，因为它允许任意第三方代码执行。

出问题的区域通常如下：

```nginx
location ~* \.php$ {
    fastcgi_pass backend;
    # [...]
}
```

这里每一个以`.php`结尾文件的请求将会给*FastCGI*后端。
这里的问题是如果全路径并不指向文件系统上实际的文件，默认的 PHP 配置会试图猜测你需要执行哪个文件。

例如，如果请求不存在的文件`/forum/avatar/1232.jpg/file.php`，但`/forum/avatar/1232.jpg`存在，
PHP 解释器将处理`/forum/avatar/1232.jpg `。如果它包含 PHP 代码，这段代码将相应的执行。

下面的选项可以避免上述情况：

- 在php.ini中设置`cgi.fix_pathinfo=0`。使得 PHP 解释器仅尝试给出的路径，如果文件没有找到就停止处理。
- 保证 NGINX 仅传给后端指定的 PHP 文件请求：

```nginx
location ~* (file_a|file_b|file_c)\.php$ {
    fastcgi_pass backend;
    # [...]
}
```

- 特别地在任何包含用户上传的目录中禁用 PHP 文件的执行

```nginx
location /uploaddir {
    location ~ \.php$ {return 403;}
    # [...]
}
```

- 使用*try_files*指令过滤

```nginx
location ~* \.php$ {
    try_files $uri =404;
    fastcgi_pass backend;
    # [...]
}
```

- 使用嵌套的 location 过滤

```nginx
location ~* \.php$ {
    location ~ \..*/.*\.php$ {return 404;}
    fastcgi_pass backend;
    # [...]
}
```

## Script Filename 中的 FastCGI Path

一些指南倾向使用绝对路径得到信息。这在 PHP 区块很常见。当你从源安装 NGINX ，通常你能够把
`include fastcgi_params;`加入到配置中。该文件通常在 NGINX 配置目录`/etc/nginx/`。

**GOOD**

```nginx
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
```

**BAD**

```nginx
fastcgi_param  SCRIPT_FILENAME    /var/www/yoursite.com/$fastcgi_script_name;
```
`$document_root`在哪里设置？是通过 root 指令在 server 区块设置。root 指令没有配置？看看
第一个陷阱。

## 繁杂的 Rewrites

不要感觉糟糕，正则表达式容易让人困惑。实际上，我们应该努力保持整洁、简单，不增加繁琐的东西，这容易做到。

**BAD**

```nginx
rewrite ^/(.*)$ http://example.com/$1 permanent;
```

**GOOD**

```nginx
rewrite ^ http://example.com$request_uri? permanent;
```

**BETTER**

```nginx
return 301 http://example.com$request_uri;
```

第一个 rewrite 捕获在第一个*/*之后的 URI。通过使用内建变量`$request_uri`，我们可以有效的避免做任何捕获或者匹配。

## Rewrite 丢失 *http://*

rewrite 是简单，记得添加`scheme`。

**BAD**

```nginx
rewrite ^ example.com permanent;
```

**GOOD**

```nginx
rewrite ^ http://example.com permanent;
```

在上面可以看到，我们在 rewrite 添加了`http://`。简单有效。

## 代理一切

**BAD**

```nginx
server {
    server_name _;
    root /var/www/site;
    location / {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/tmp/phpcgi.socket;
    }
}
```

令人讨厌的，在这种情况下把所有的请求都给 PHP 处理。为什么？Apache 也许是这样做，但你不必。让我这么说吧...
`try_files`指令存在的原因，它按特定的顺序尝试文件。意味着 NGINX 可以先尝试提供静态内容。如果不能，继续尝试。
这样 PHP 根本不参与，快的多。特别是如果你通过 PHP 提供 1MB 图片的时间是直接提供的1000倍，让我们看看该怎么做。

**GOOD**

```nginx
server {
    server_name _;
    root /var/www/site;
    location / {
        try_files $uri $uri/ @proxy;
    }
    location @proxy {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/tmp/phpcgi.socket;
    }
}
```

**Also GOOD**

```nginx
server {
    server_name _;
    root /var/www/site;
    location / {
        try_files $uri $uri/ /index.php;
    }
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/tmp/phpcgi.socket;
    }
}
```
很简单，是不是？如果请求的 URI 存在，NGINX 直接提供。如果不存在，检查目录是否存在。如果不存在，传递给你的代理。
只有当 NGINX 不能直接服务请求的 URI，你的代理将参与开销。

现在想想你的多少请求是静态资源，比如 images、css、javascript 等。你可能节省很多开销。

## 配置更改没有生效

浏览器缓存。你的配置也许是很棒的，但你会坐在那，撞墙一个月。问题出在你的浏览器缓存。
当你下载了东西，你的浏览器会保存下来。同时也保存了文件是如何提供的。如果你配置有 types 区块，你会遇到这情况。

处理方法：

- 在 Firefox 浏览器中按 Ctrl+Shift+Delete，检查缓存，点击 *Clear Now*。其他浏览器，可搜索下方法。
每次更改之后都清除下，会为你节省不少麻烦。
- 使用 *curl*。

## VirtualBox

如果这不起作用,而且你在 VirtualBox 中的虚拟机中运行 NGINX，也许是`sendfile()`引起的问题。
简单的注释掉`sendfile`指令，或者设置为“off”。

```nginx
sendfile off;
```
## 缺失的 HTTP 头

如果你没有明确的设置`underscores_in_headers on`，NGINX 将会默默地忽略掉带有下划线的头信息（根据 HTTP 标准是完全有效的）。
这样做为了防止头信息映射到 CGI 变量过程中破折号和下划线都映射成下划线引起歧义。

## 没有使用标准的 Document Root Locations

有些目录在任何文件系统中不应该用于托管数据。其中包括`/`和`/root`。你不应该使用这些作为网站根目录。

这样作让你的隐私数据毫无预期的返回给请求。

永远不要这样做！！！

```nginx
server {
    root /;

    location / {
        try_files /web/$uri $uri @php;
    }

    location @php {
        # [...]
    }
}
```

当一个请求`/foo`，因为文件找不到将代理到 php 。这看起来正常，直到请求`/etc/passwd`。是的，你给了服务器上所有用户列表。
在某些情况下，Nginx 服务器进程被设置以 root 用户运行。是的，我们现在有你的用户列表以及密码 hash 密码，以及如何 hash 的。

文件系统层级标准定义了数据应该存在哪里。你应该看看。
简而言之，你的 web 内容可以存放在`/var/www/`、`/srv`、`/usr/share/www`。

## 使用默认的 Document Root

Ubuntu、Debian 等其他操作系统中NGINX包，作为易于安装的包会提供默认的配置文件用作一个例子，而且会包含网站根目录存储基本的
HTML 文件。

多数系统包不检查默认的*Document Root*中文件是否存在或者修改。这样会在软件包升级时导致代码丢失。经验丰富的系统管理员没有期望在
升级期间默认的*document root*内容保持不变。

你不应该对将网站的关键文件使用默认的*document root*。在你的系统升级或者更新 NGINX 包时有很大可能性会导致默认的*document root*
内容会改变。

## 使用 Hostname 去解析地址

**BAD**

```nginx
upstream {
    server http://someserver;
}

server {
    listen myhostname:80;
    # [...]
}
```
你不应该在 listen 指令中使用 hostname。也许这样有效，也将会有大量的问题。
其中一个问题是在服务启动或者重启时 hostname 不能解析。这会引起 NGINX 无法绑定到特定的 TCP 端口，最终 NGINX 启动失败。

更安全的做法是知道需要绑定的 IP 地址，并使用 IP 而不是 hostname。 阻止 NGINX 需要查找 IP ，不依赖内部或者外部的解析。

upstream 区块也有同样的问题。并不是总要在 upstream 区块避免使用 hostname，这不是好的做法，使用时要考虑清楚。

**GOOD**

```nginx
upstream {
    server http://10.48.41.12;
}

server {
    listen 127.0.0.16:80;
    # [...]
}
```

## HTTPS 中使用 SSLv3

由于 SSLv3 的POODLE漏洞，建议在 SSL 的网站不启用 SSLv3。你可以很简单的禁用 SSLv3，只提供 TLS 协议。

```nginx
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
```
