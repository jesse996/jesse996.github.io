# Webhook教程


# github webhook 是什么

当 github 上面的仓库发生变化时（push 或 issue），可以自定义回调（一条链接）。

# 先定义一个脚本

例如：

```bash
#/root/webhook/deploy.sh
#!/bin/bash

cd /path/to/directory
git pull

```

在同意目录下新建 webhook.js 文件

```js
var http = require('http')
var spawn = require('child_process').spawn
var createHandler = require('github-webhook-handler')
// 注意将 secret 修改你自己的
var handler = createHandler({ path: '/webhook', secret: 'yourwebhooksecret' })

http.createServer(function (req, res) {
    handler(req, res, function (err) {
        res.statusCode = 404
        res.end('no such location')
    })
}).listen(6666)

handler.on('error', function (err) {
    console.error('Error:', err.message)
})

handler.on('push', function (event) {
    console.log(
        'Received a push event for %s to %s',
        event.payload.repository.name,
        event.payload.ref
    )

    runCommand('sh', ['./deploy.sh'], function (txt) {
        console.log(txt)
    })
})

function runCommand(cmd, args, callback) {
    var child = spawn(cmd, args)
    var resp = 'Deploy OK'
    child.stdout.on('data', function (buffer) {
        resp += buffer.toString()
    })
    child.stdout.on('end', function () {
        callback(resp)
    })
}
```

用 pm2 启动服务

```
pm2 start webhook.js
```

# Nginx 配置

```
    # GitHub auto deploy webhook
    location /webhook {
        proxy_pass http://127.0.0.1:6666;
    }

```

# GitHub Webhook 配置

在 github 仓库的`Setting-webhooks`新建一个 webhook,`Content Type` 为 `application/json`，`secret` 设置成与 webhook.js 中的相同。
![图](https://pic.ioiox.com/image/CN5c)

# 验证

如果如图表示成功
![图](https://pic.ioiox.com/image/CYJn)

