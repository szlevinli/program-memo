# VS Code Memo

## Debug

- 调用 `yarn` 来执行调试工作. 创建 `launch.json` 文件

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch via Yarn",
      "request": "launch",
      "runtimeArgs": ["ts-node", "${relativeFile}"],
      "runtimeExecutable": "yarn",
      "skipFiles": ["<node_internals>/**"],
      "type": "pwa-node"
    }
  ]
}

```

