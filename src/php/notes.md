# PHP

## 偷懒小技巧

有一些时候，我们需要使用临时的 PHP 仿真环境调试需要输出交互的代码，命令行直接执行代码，配合 curl 来搞太低效了，可以使用下面的方式，快速创建一个容器环境：

（默认环境缺少不少组件，所以你也可以换成你封装好的镜像来玩）

```bash
docker run -d -p 11080:80 -v "$(pwd)/index.php":/var/www/html/index.php php:7.4-apache
```