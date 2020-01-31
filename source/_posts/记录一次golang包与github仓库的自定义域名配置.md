---
title: 记录一次golang包与github仓库的自定义域名配置
date: 2020-01-22 16:33:45
tags:
---

## 目的

最近将一个开源项目转换到组织，顺带注册了个域名，想要实现如下

- 使用 go get 从自己的域名拉取包，这样拉下的包存放在以自己域名命名的文件夹里
- 使用 GitHub 作为代码仓库，将自己域名作为 git remote

## 总体

总体思路就是做跳转，这里使用 nginx 来实现

## Golang 包的配置

参考[官方说明](https://golang.org/cmd/go/#hdr-Remote_import_paths)

需要在一个 html 加个 meta 标签填入包名与仓库类型和地址

`<meta name="go-import" content="example.org git https://code.org/r/p/exproj">`

于是做了个 `index.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta
      name="go-import"
      content="ehang.io/nps git https://github.com/ehang-io/nps"
    />
    <meta
      http-equiv="refresh"
      content="0; url=http://github.com/ehang-io/nps"
    />
  </head>
  <body>
    Read the <a href="https://github.com/ehang-io/nps">documentation</a>.
  </body>
</html>
```

扔在 nginx 的文件目录下`/usr/share/nginx/html/nps/index.html`

nginx 将请求配好

```nginx
location /nps {
        root /usr/share/nginx/html;
    }
```

这样使用 go get 自己的域名就行了，如果没有这个 meta 标签直接跳转，go mod 验证不会通过的

## Git Remote 配置

既然 Golang 包是通过 html 跳转的，显然 git 直接设置 remote 为域名就不行了

幸运的是，我们一般是用的 `.git` 这样作为 remote，所以就可以在 nginx 上用正则表达式来匹配`.git`，例如

```nginx
location ~* \.(git|git/) {
        return 307 https://github.com/ehang-io$request_uri;
    }
```

这样 remote 设置为`https://ehang.io/nps.git`就能正常使用 github 的仓库了
