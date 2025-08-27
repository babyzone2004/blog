安装环境

```shell
#安装homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh"
#安装ruby
brew install ruby
```

如果ruby提示版本问题
你这个情况就是 Homebrew 安装了新的 Ruby 3.4.5，但是系统默认调用的还是 macOS 自带的老版本 Ruby（2.6）。

要解决就是要把 Homebrew 的 ruby 放到 PATH 前面

```shell
# 找到 Homebrew 的 Ruby 路径
brew --prefix ruby
# 将 Homebrew 的 Ruby 添加到 PATH 前面
export PATH="/opt/homebrew/opt/ruby/bin:$PATH"
# 或者
export PATH="$(brew --prefix ruby)/bin:$PATH"
# 重新加载 shell 配置文件
source ~/.zshrc
#验证ruby版本
ruby -v
```

安装jekyll

```shell
gem install bundler jekyll
#jeklly项目初始化,进入项目目录
jekyll new --skip-bundle .
```

如果zsh 提示 command not found: jekyll，说明 jekyll 的可执行文件没在 PATH 里。

```shell
# 执行
gem env
# 找到类似路径：
# EXECUTABLE DIRECTORY: /opt/homebrew/lib/ruby/gems/3.4.0/bin
# 加入.zshrc
export PATH="/opt/homebrew/lib/ruby/gems/3.4.0/bin:$PATH"
# 执行
source ~/.zshrc
# 再试
jekyll -v
```

安装依赖

```shell
# 执行
bundle install
```

开启服务

```shell
bundle exec jekyll serve
```

[主题自定义](https://jekyllrb.com/docs/themes/#overriding-theme-defaults)，采用覆盖的方式

```shell
#执行，获取主题路径，
bundle info --path minima
# 输入：/usr/local/lib/ruby/gems/2.6.0/gems/minima-2.5.1，打开路径
```

把路径文件夹复制到项目根目录，根据需求更改