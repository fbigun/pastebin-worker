# API 参考说明

## GET `/`

返回索引页面。

## **GET** `/<name>[.<ext>]` or `/<name>/<filename>[.<ext>]`

获取名称为 `<name>` 的粘贴内容。默认情况下，它将返回粘贴的原始内容。

`Content-Type` 头部会被设置为 `text/plain;charset=UTF-8`。如果指定了 `<ext>`，服务器会根据 `<ext>` 推断 MIME 类型并更改 `Content-Type`。此方法接受以下查询字符串参数：

`Content-Disposition` 头部默认设置为 `inline`。但可以通过 `?a` 查询字符串进行覆盖。如果粘贴上传时带有文件名，或者在给定的请求 URL 中设置了 `<filename>`，`Content-Disposition` 会附加 `filename*` 来指示文件名（如果存在 `<ext>` 也会包含）。

- `?a=`: 可选。若存在该参数，则将 `Content-Disposition` 设置为 `attachment`。
- `?lang=<lang>`: 可选。返回一个由 prism.js 提供语法高亮功能的网页。
- `?mime=<mime>`: 可选。指定 MIME 类型，忽略 `<ext>` 的影响。如果指定了 `lang`，则此参数无效（这种情况下 MIME 类型始终为 `text/html`）。

示例：`GET /abcd?lang=js`，`GET /abcd?mime=application/json`。

如果发生错误，服务器将返回不同于 `200` 的状态码：

- `404`: 未找到指定名称的粘贴。
- `500`: 出现意外异常。你可以将此问题报告给开发者进行修复。

使用示例：

```shell
$ curl https://shz.al/i-p-
https://web.archive.org/web/20210328091143/https://mp.weixin.qq.com/s/5phCQP7i-JpSvzPEMGk56Q

$ curl https://shz.al/~panty.jpg | feh -

$ firefox 'https://shz.al/kf7z?lang=nix'

$ curl 'https://shz.al/~panty.jpg?mime=image/png' -w '%{content_type}' -o /dev/null -sS
image/png;charset=UTF-8
```

## GET `/<name>:<passwd>`

返回用于编辑名称为 `<name>` 且密码为 `<passwd>` 的粘贴内容的网页。

如果发生错误，服务器将返回不同于 `200` 的状态码：

- `404`: 未找到指定名称的粘贴。
- `500`: 出现意外异常。你可以将此问题报告给开发者进行修复。

## GET `/u/<name>`

重定向到名称为 `<name>` 的粘贴中记录的 URL。

如果发生错误，服务器将返回不同于 `302` 的状态码：

- `404`: 未找到指定名称的粘贴。
- `500`: 出现意外异常。你可以将此问题报告给开发者进行修复。

使用示例：

```shell
$ firefox https://shz.al/u/i-p-

$ curl -L https://shz.al/u/i-p-
```

## 获取 `/a/<name>`

返回从名称为 `<name>` 的粘贴中存储的 Markdown 文件转换而来的 HTML。Markdown 转换遵循 GitHub 风格 Markdown (GFM) 规范，由 [remark-gfm](https://github.com/remarkjs/remark-gfm) 提供支持。

语法高亮由 [prism.js](https://prismjs.com/) 提供支持。LaTeX 数学公式由 [MathJax](https://www.mathjax.org/) 提供支持。

如果发生错误，服务器将返回不同于 `200` 的状态码：

- `404`: 未找到指定名称的粘贴。
- `500`: 出现意外异常。你可以将此问题报告给开发者进行修复。

使用示例：

```md
# Header 1
This is the content of `test.md`

<script>
alert("Script should be removed")
</script>

## Header 2

| abc | defghi |
:-: | -----------:
bar | baz

**Bold**, `Monospace`, *Italics*, ~~Strikethrough~~, [URL](https://github.com)

- A
 - A1
 - A2
- B

![Panty](Z-%E9%99%84%E4%BB%B6/f383d7455843af2f38744204589bc397_MD5.jpg)

1. first
2. second

> Quotation

$$
\int_{-\infty}^{\infty} e^{-x^2} = \sqrt{\pi}
$$

```

```shell
$ curl -Fc=@test.md -Fn=test-md https://shz.al

$ firefox https://shz.al/a/~test-md
```

## **POST** `/`

上传你的粘贴内容。它接受表单数据中的以下参数：

- `c`: 必需。粘贴的 **内容**，可以是文本或二进制数据。其大小不应超过 10 MB。获取粘贴内容时，`Content-Disposition` 中的 `filename` 会显示出来。
- `e`: 可选。粘贴的 **过期** 时间。在此时间段过后，粘贴内容将被永久删除。它可以是一个整数或浮点数，后面可跟一个可选的单位（默认为秒）。支持的单位有：`s`（秒）、`m`（分钟）、`h`（小时）、`d`（天）、`M`（月）。例如，`360.24` 表示 360.25 秒；`25d` 表示 25 天。由于 Cloudflare KV 存储的限制，该时间不应小于 60 秒。
- `s`: 可选。用于修改和删除粘贴内容的 **密码**。如果未指定，服务器将生成一个随机字符串作为密码。
- `n`: 可选。你为粘贴内容自定义的 **名称**。如果未指定，服务器将生成一个随机字符串（默认 4 个字符）作为名称。获取自定义名称的粘贴内容时，需要在名称前加上 `~`。名称长度至少为 3 个字符，可由字母、数字以及 `+_-[]*$=@,;/` 中的字符组成。
- `p`: 可选。**私有模式** 标志。如果指定了任何值，粘贴的名称长度将为 24 个字符。如果使用了 `n` 参数，此标志无效。

如果没有发生错误，`POST` 方法默认返回一个 JSON 字符串，例如：

```json
{
    "url": "https://shz.al/abcd",
    "admin": "https://shz.al/abcd:w2eHqyZGc@CQzWLN=BiJiQxZ",
    "expire": 100,
    "isPrivate": false
}
```

字段说明：

- `url`: 字符串。用于获取粘贴内容的 URL。使用自定义名称时，格式类似 `https://shz.al/~myname`。
- `suggestUrl`: 字符串或 null。可能携带文件名或 URL 重定向的 URL。
- `admin`: 字符串。用于更新和删除粘贴内容的 URL，是在 `url` 后面加上 `~` 和密码。
- `expire`: 字符串或 null。过期秒数。
- `isPrivate`: 布尔值。表示粘贴内容是否处于私有模式。

如果发生错误，服务器将返回不同于 `200` 的状态码：

- `400`: 你的请求格式错误。
- `409`: 名称已被使用。
- `413`: 内容太大。
- `500`: 出现意外异常。你可以将此问题报告给开发者进行修复。

使用示例：

```shell
$ curl -Fc="kawaii" -Fe=300 -Fn=hitagi https://shz.al  # uploading some text
{
  "url": "https://shz.al/~hitagi",
  "admin": "https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv",
  "isPrivate": false,
  "expire": 300
}

$ curl -Fc=@panty.jpg -Fn=panty -Fs=12345678 https://shz.al   # uploading a file
{
  "url": "https://shz.al/~panty",
  "admin": "https://shz.al/~panty:12345678",
  "isPrivate": false
}

# because `curl` takes some characters as filed separator, the fields should be
# quoted by double-quotes if the field contains semicolon or comma
$ curl -Fc=@panty.jpg -Fn='"hi/hello;g,ood"' -Fs=12345678 https://shz.al
{
  "url": "https://shz.al/~hi/hello;g,ood",
  "admin": "https://shz.al/~hi/hello;g,ood:QJhMKh5WR6z36QRAAn5Q5GZh",
  "isPrivate": false
}
```

## **PUT** `/<name>:<passwd>`

更新名称为 `<name>` 且密码为 `<passwd>` 的粘贴内容。它接受表单数据中的以下参数：

- `c`: 必需。与 `POST` 方法中的 `c` 参数相同。
- `e`: 可选。与 `POST` 方法中的 `e` 参数相同。注意，删除时间会重新计算。
- `s`: 可选。与 `POST` 方法中的 `s` 参数相同。

`PUT` 方法的返回结果与 `POST` 方法相同。

如果发生错误，服务器将返回不同于 `200` 的状态码：

- `400`: 你的请求格式错误。
- `403`: 你的密码不正确。
- `404`: 未找到指定名称的粘贴。
- `413`: 内容太大。
- `500`: 出现意外异常。你可以将此问题报告给开发者进行修复。

使用示例：

```shell
$ curl -X PUT -Fc="kawaii~" -Fe=500 https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv
{
  "url": "https://shz.al/~hitagi",
  "admin": "https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv",
  "isPrivate": false,
  "expire": 500
}

$ curl -X PUT -Fc="kawaii~" https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv
{
  "url": "https://shz.al/~hitagi",
  "admin": "https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv",
  "isPrivate": false
}
```

## DELETE `/<name>:<passwd>`

删除名称为 `<name>` 且密码为 `<passwd>` 的粘贴内容。全局同步删除操作可能需要几秒钟。

如果发生错误，服务器将返回不同于 `200` 的状态码：

- `403`: 你的密码不正确。
- `404`: 未找到指定名称的粘贴。
- `500`: 出现意外异常。你可以将此问题报告给开发者进行修复。

使用示例：

```shell
$ curl -X DELETE https://shz.al/~hitagi:22@-OJWcTOH2jprTJWYadmDv
the paste will be deleted in seconds

$ curl https://shz.al/~hitagi
not found
```
