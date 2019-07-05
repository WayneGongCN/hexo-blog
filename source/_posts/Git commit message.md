---
title: Git commit message 规范化与校验
date: 2019-07-04
tags: notes
categories: notes
---


## 使用 git-cz 交互式提交 commit

安装 commitizen cli 命令行工具与 commitizen adapter 适配器（可以理解为 commit message 的模板）
```
yarn install commitizen cz-conventional-changelog
```

`package.json` 中配置 `script` 与 `config` 字段。

`config.commitizen.path` 为适配器名或路径。

```
  "script": {
    "commit": "git-cz"
  },
  ...
  "config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  }
```

现在尝试使用 `npm run commit` 可以使用交互式模式提交 commit 了。


## 校验 commit message

安装 @commitlint/cli 命令行工具与 @commitlint/config-conventional 校验规则 npm 包。
```shell
# 安装依赖
npm install @commitlint/config-conventional @commitlint/cli --save-dev

# 新建一个配置文件
echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
```

配置 git hook，这里使用 husky。
```
{
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }  
  }
}
```

现在试着随便提交一些 commit message，不符合规则的则会被 commitlin 提示并取消这次提交。
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMDA2ODM5MywxOTQ4MTQwOTA5XX0=
-->
