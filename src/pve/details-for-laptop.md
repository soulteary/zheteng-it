# 笔记本使用细节

虽然使用笔记本安装 PVE 能够默认拥有双网卡、自带键盘和显示器的终端。但是默认情，操作键盘进行输入会不时得到刺耳的警告蜂鸣声音。

触发这个声音的原理是终端使用类似 Tab 进行补全的时候，触发类似下面的行为:

```bash
echo -e '\a'
```

解决问题需要编辑 `/etc/inputrc` 文件，将以下配置禁用掉：

```bash
# do not bell on tab-completion
# set bell-style none
# set bell-style visible
```
