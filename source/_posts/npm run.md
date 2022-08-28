---
title: npm run xxx 发生了什么
---
# npm run

1.npm run 发生了什么？

npm run 命令运行的是package.json 文件配置的脚本，

假设在package.json 文件配置了以下运行脚本：

<!--more--->

```
"scripts":{"lint": "vue-cli-service lint"}
```

我们执行命令 `npm run lint`，等价于执行`./node_modules/.bin/vue-cli-service lint`

可以知道：

npm run lint 命令实际运行的是项目文件目录./node_modules/.bin 下的可执行文件

npm run 会自动把./node_modules/.bin 添加到我们配置的 scripts 指令前面，就是说：

我们可以这样写：

```js
"scripts":{"lint":"vue-cli-service lint"}
```

而不必：

```js
"scripts":{"lint": "./node_modules/.bin/vue-cli-service lint"}
```

2.bin

之所以npm run 能运行指定脚本，是因为依赖的库为我们提供了相关的可执行文件

在依赖库中的package.json 中可以看到如下配置：

```js
{
  "bin": {
    "myapp": "./cli.js"
  }
}
```

在执行npm install 命令安装依赖的时候，



[更多](https://docs.npmjs.com/cli/v8/commands/npm-run-script)

[bin](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#bin)
