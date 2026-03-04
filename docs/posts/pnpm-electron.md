# Electron 与 pnpm 兼容性问题

最近在把项目从 npm 迁移到 pnpm，但是 Electron 没法正常使用，
```
"message":"Can't find Node.js binary "XXX/client/node_modules/.bin/electron.cmd": XXX/node_modules/.bin/electron.cmd
balabala...
throw new Error('Electron failed to install correctly, please delete node_modules/electron and try installing again');
```

提示安装失败，排查了网络等问题实际上根本没法解决。最后搜索发现是 pnpm 与 Electron 存在兼容问题。详细原因见[Electron 官方文档](https://www.electronjs.org/docs/latest/tutorial/tutorial-first-app#initializing-your-npm-project)，原文在下：
> Install dependencies with a regular node_modules folder
>
>Electron's packaging toolchain requires the node_modules folder to be physically on disk in the way that npm installs Node dependencies. By default, Yarn Berry and pnpm both use alternative installation strategies.
>
> Therefore, you must set nodeLinker: node-modules in Yarn or nodeLinker: hoisted in pnpm if you are using those package managers.