# 获取当前的镜像地址

```js
npm get registry 
```

# 切换成淘宝镜像

```js
npm config set registry http://registry.npm.taobao.org/
```

# 换回原来

```js
npm config set registry https://registry.npmjs.org/
```

## npm 切换镜像站点

https://www.runoob.com/w3cnote/npm-switch-repo.html



# yarn切换镜像源

1、查看一下当前源

```
yarn config get registry
```

2、切换为淘宝源

```
yarn config set registry https://registry.npm.taobao.org
```

3、或者切换为自带的

```
yarn config set registry https://registry.yarnpkg.com
```