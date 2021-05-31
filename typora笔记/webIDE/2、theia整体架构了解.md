[TOC]

## The repository structure

Theia repository has multiple folders:

- `packages` folder contains runtime packages, as the core package and extensions to it

- ```
  dev-packages
  ```

   

  folder contains devtime packages

  - [@theia/cli](https://github.com/eclipse-theia/theia/blob/master/dev-packages/cli/README.md) is a command line tool to manage Theia applications
  - [@theia/ext-scripts](https://github.com/eclipse-theia/theia/blob/master/dev-packages/ext-scripts/README.md) is a command line tool to share scripts between Theia runtime packages

- `examples` folder contains example applications, both Electron-based and browser-based

- `doc` folder provides documentation about how Theia works

- `scripts` folder contains JavaScript scripts used by npm scripts when installing

- the root folder lists dev dependencies and wires everything together with [Lerna](https://lerna.js.org/)



准备工作：

依赖注入框架：[Inversify.js](http://inversify.io/)



资料地址：

https://github.com/Pines-Cheng/blog/issues?page=1&q=is%3Aissue+is%3Aopen

https://zhaomenghuan.js.org/blog/theia-tech-architecture.html

https://www.yuque.com/zhaomenghuan/theia/nvwnzh