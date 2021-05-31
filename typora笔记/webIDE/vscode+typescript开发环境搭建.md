这里vscode的安装不做介绍，只介绍怎么安装typescript和怎么在vscode里面使用。



安装typescript

```shell
# 安装typescript
$ npm install -g typescript --registry=https://registry.npm.taobao.org
$ tsc --version
Version 4.1.3

# 这是一个typescript的交互式控制台，可以用来调试ts脚本，不然只能调试编译后的js
$ npm install -g ts-node --registry=https://registry.npm.taobao.org
$ ts-node --version
v9.1.1
```

创建项目

```shell
# 创建项目目录
$ mkdir tomgs-ts
$ cd tomgs-ts
# 生成package.json
$ npm init [-y]
# 创建tsconfig.json 文件
$ tsc --init
# 创建ts程序目录
$ mkdir -p src/main
# 创建ts程序编译目录
$ mkdir -p src/build
```

配置项目

- 修改tsconfig.json

  参照下面的改就行了。。。

  ```json
  {
      "compilerOptions": {
          "module": "commonjs",
          "noImplicitAny": true,
          "removeComments": true,
          "emitDecoratorMetadata": true,
          "experimentalDecorators":true,
          "preserveConstEnums": true,
          "strictNullChecks": true,
          "noImplicitReturns": false,
          "moduleResolution": "node",
          "esModuleInterop":true,
          "target": "es6",/*编译成es6规范的js*/
          "allowJs": false,/*不允许js混合编程，新项目强烈推荐，迁移的老项目只能设置成true了*/
          "sourceMap": true,/*调试时的时候必须开sourceMap*/
          "outDir": "build"//js文件的输出目录
      },
      // "files": [
  
      // ],
      "include": [
          "src/*"
      ],
      "exclude": [
          "node_modules"
      ]
  }
  ```

安装@types

在 [http://www.cnblogs.com/haogj/p/6194472.html](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Fwww.cnblogs.com%2Fhaogj%2Fp%2F6194472.html) 中有介绍 安装 .d.ts文件的方法如下

```shell
$ npm install @types/node --dev-save
```

使用vscode打开项目



编译

```
npm run build
```

先执行`npm install -g live-server`安装 live-server 之后,修改 package.json

```
  "scripts": {
    "start": "tsc -w & live-server",
  },
```

执行 `npm start` 就可以写一些 ts 的小 demo 了



参考文档

https://my.oschina.net/jim19770812/blog/1831493

https://www.jianshu.com/p/0569d2604119

https://juejin.cn/post/6844904071250313223

https://my.oschina.net/u/4302374/blog/3383555

