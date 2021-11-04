# Docker

## alpine 小技巧

偶尔使用 alpine 进行封装或者 shell 进行调试开发的时候，安装软件特别慢，可以考虑使用下面的方式快速切换软件源。

```bash
sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories
```