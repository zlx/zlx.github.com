## 使用说明

update

#### 创建新博客

    bundle exec rake new_post

#### 本地构建 & 预览

    bundle exec jekyll serve

#### 推送文章

    git push digitalocean master


***注意：本地环境中默认对应 source 分支，如果需要 push 到 master，请务必使用 rake deploy 命令, 如出现错误，请检查 _deploy 目录下是否存在 .git 文件夹***
