# 为镜像打网卡驱动拾遗

因为 ESXi 6.x 与 7.x 的网卡 API 版本不一致，所以这两个版本的网卡驱动不能够混用。

然而，有一些网卡驱动目前暂时没有 7.x 版本的适配文件，所以我们如果想在一些老设备上使用 ESXi ，可能不得不使用 6.x 版本的软件。

我选择的基础镜像版本是：**ESXi-6.7.0-20191204001-standard**

镜像下载地址：

```
https://customerconnect.vmware.com/cn/downloads/details?downloadGroup=ESXI67U3B&productId=742
```

驱动下载可以从  v-front.de 下载，也可以到厂商的下载中心进行下载（比如 dell、lenovo 等），但是不建议使用社区远古版本的驱动（尤其是 issue 反馈了大量未解决 bug 的项目）

```
https://vibsdepot.v-front.de/wiki/index.php/Net55-r8168
```


在[《NUC 折腾笔记 - 安装 ESXi 7》](https://soulteary.com/2021/06/22/nuc-notes-install-esxi7.html)中，我基本对 7.x 版本的镜像打驱动补丁的方式做了完整介绍。

但是在 6.x 中，命令需要一些变动，多一步修改系统允许加载的外部驱动等级，将默认等级降低至“接受社区驱动软件”


```bash
New-EsxImageProfile -CloneProfile "ESXi-6.7.0-20191204001-standard" -name "ESXi-6.7.0-20191204001-nic" -vendor "soulteary"
Set-EsxImageProfile -Name "ESXi-6.7.0-20191204001-nic" -AcceptanceLevel CommunitySupported
Add-EsxSoftwarePackage -ImageProfile  "ESXi-6.7.0-20191204001-nic" -SoftwarePackage "net55-r8168"
Export-EsxImageProfile -ImageProfile "ESXi-6.7.0-20191204001-nic" -ExportToISO -filepath .\exsi6.7.0.iso
```
