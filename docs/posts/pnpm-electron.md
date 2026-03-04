# Electron 与 pnpm 兼容性问题

最近在把项目从 npm 迁移到 pnpm，但是 Electron 没法正常使用，
```
"message":"Can't find Node.js binary "XXX/client/node_modules/.bin/electron.cmd": XXX/node_modules/.bin/electron.cmd
balabala...
throw new Error('Electron failed to install correctly, please delete node_modules/electron and try installing again');
```

提示安装失败，排查了网络等问题实际上根本没法解决[Electron 官方文档](https://www.electronjs.org/docs/latest/tutorial/tutorial-first-app#initializing-your-npm-project)，则表示如下
> Install dependencies with a regular node_modules folder
>
>Electron's packaging toolchain requires the node_modules folder to be physically on disk in the way that npm installs Node dependencies. By default, Yarn Berry and pnpm both use alternative installation strategies.
>
> Therefore, you must set nodeLinker: node-modules in Yarn or nodeLinker: hoisted in pnpm if you are using those package managers.

说使用 `node-linker=true` 然而对我并没有用，最后在 `pnpm install` 提示的时候命令行看到了 WARN：
```
╭ Warning ───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮ 
│                                                                                                                                        │ 
│   Ignored build scripts: @parcel/watcher@2.5.6, better-sqlite3@12.5.0, electron-winstaller@5.4.0, electron@39.2.5, esbuild@0.25.12,    │ 
│   esbuild@0.26.0, esbuild@0.27.3, lmdb@3.4.3, msgpackr-extract@3.0.3.                                                                  │ 
│   Run "pnpm approve-builds" to pick which dependencies should be allowed to run scripts.                                               │ 
│                                                                                                                                        │ 
╰────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯ 
Done in 16.3s using pnpm v10.30.3
```
提示如果要允许脚本则使用上面的命令，而 Electron 是通过自己的一个 js 脚本进行后续的 Electron 应用安装的。执行此命令后，本次的问题就解决了。
