1. [Angular commit convention](https://zj-git-guide.readthedocs.io/zh_CN/latest/message/Angular提交信息规范/)：Angular 项目的提交规范
2. [Conventional Commits](https://www.conventionalcommits.org/zh-hans/v1.0.0/)：约定式提交规范，基于 Angular 提交规范，提供了更加通用、简洁和灵活的提交规范。

安装commitzen

```shell
pnpm add -w --save-dev commitizen
```

pnpm配置.npmrc文件

```shell
# 允许在 workspace 根目录安装依赖
ignore-workspace-root-check=false
```

初始化规范配置

```shell
npx commitizen init cz-conventional-changelog --save-dev --save-exact --pnpm
```

配置git hook，对普通commit进行校验

```shell
npx husky add .husky/commit-msg "exec < /dev/tty && npx cz --hook || true"
```

使用 `conventional-changelog-cli` 可以自动生成 Change log：

```shell
pnpm add -w conventional-changelog-cli -D
```

生成 changelog

```
conventional-changelog -p angular -i CHANGELOG.md -s
```

可以在 package.json 中加入配置方便使用

```
"scripts": {
  "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s"
}
```

