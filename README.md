# OpenWrt 路由软件 feed

本仓库是 [hnt510-great/openwrt](https://github.com/hnt510-great/openwrt) 定制主树使用的私有源码 feed。
项目面向 x86_64/glibc、APK、原生 systemd 和 GNOME 桌面；软件选择以主树
保存的完整配置为准。

| 项目 | 值 |
| --- | --- |
| Feed 名称 | `routing` |
| 本仓库分支 | `master` |
| 原始上游 | [https://github.com/openwrt/routing](https://github.com/openwrt/routing)（`master`） |

本 feed 基于 OpenWrt routing，提供可选的路由、邻居发现和组网软件配方。
当前目录包含 `batmand`、`cjdns`、`luci-app-cjdns`、`ndppd`、`pimbd`、
`ahcpd` 等组件；实际是否进入镜像由主树 `.config` 决定。

## 在本项目中的职责

本仓库作为独立的私有 Git feed 参与完整源码重建。主树 docs 没有为本 feed
定义额外的专属组网方案；不要把仓库中存在某个配方视为该服务已启用或已通过
物理网络验证。

主树的路由转发、full cone NAT、dnsmasq nftset 快路径，以及 daed eBPF
透明代理，分别由主树内核/网络配方和 `package/custom` 实现，不由本 feed
提供。原生 systemd 下的服务管理仍需遵循主树保留的 UCI/procd 接口。

完整镜像验证中的 DHCP、DNS 和 routing/NAT 检查，也不能替代对 cjdns、
batmand 等可选协议的专项组网验收。

## 获取与构建

先在构建机登录有权访问全部私有仓库的 GitHub 账号，再获取完整主树：

```sh
gh auth login --hostname github.com --git-protocol https --web
gh auth setup-git
git clone https://github.com/hnt510-great/openwrt.git
cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a
```

主树 `feeds.conf.default` 已包含：

```text
src-git routing https://github.com/hnt510-great/openwrt-routing.git;master
```

在现有主树中只更新本 feed，可执行：

```sh
./scripts/feeds update routing
./scripts/feeds install -a -p routing
```

先在主树配置中选择所需包并运行 `make defconfig`。单包构建示例：

```sh
make package/feeds/routing/cjdns/compile V=s
```

以上命令均从主树根目录运行。保留已有 `.config`；新 checkout 的配置恢复
步骤见 [本地构建指南](https://github.com/hnt510-great/openwrt/blob/main/docs/local-build.md)。`install` 安装的是构建配方链接，
不会在设备上安装软件。完整 `update -a` 还会检查并可能修改主树的 custom 配方。

## 维护与验证

`origin` 指向本私有仓库。新 clone 需要合并原始上游时，先添加一次 `upstream`：

```sh
git -C feeds/routing remote add upstream https://github.com/openwrt/routing.git
git -C feeds/routing fetch upstream
git -C feeds/routing merge upstream/master
# 审阅、解决冲突并完成相关验证后推送
git -C feeds/routing push origin master
```

已有 `upstream` 时跳过添加命令。`scripts/feeds update` 拉取本私有分支，
不会自动合并原始上游。本地集成修改与主树的提交分别管理，保留现有提交、
stash 和未提交文件。主树的 `patches/feeds/` 和功能清单是历史备份，
不要向已经包含修改的 feed 重复应用补丁。

涉及已发布 APK 内容的修改必须产生新包版本，不能覆盖同一包身份。源码
检查、包构建、VM 与物理设备验证应分别记录；本文汇总已有设计和验证记录，
不新增构建或设备验证结论。私有源码托管说明见
[GitHub 源码托管](https://github.com/hnt510-great/openwrt/blob/main/docs/github-source-hosting.md)。

## 设计与验证文档

- [当前源码与完整配置重建](https://github.com/hnt510-great/openwrt/blob/main/docs/local-build.md)
- [Full cone NAT 的源码归属](https://github.com/hnt510-great/openwrt/blob/main/docs/fullcone-nat.md)
- [dnsmasq nftset 快路径](https://github.com/hnt510-great/openwrt/blob/main/docs/dnsmasq-nftset.md)
- [daed eBPF 代理](https://github.com/hnt510-great/openwrt/blob/main/docs/daed.md)
- [完整镜像网络验证范围](https://github.com/hnt510-great/openwrt/blob/main/docs/full-image-validation-20261031.md)

## 许可与上游

保留上游版权和许可。各软件包源码的许可按其配方 `PKG_LICENSE` 及源码声明执行。
