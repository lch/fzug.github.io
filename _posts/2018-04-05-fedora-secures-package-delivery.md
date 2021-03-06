---
layout: post
title:  "Fedora 如何保障软件包分发安全性"
date:   2018-04-05 14:01:00 +0800
---

今年早些时候， packagecloud 发布了一篇名为 [“attacks against GPG signed APT repositories” （针对启用了 GPG 签名的 APT 仓库的攻击）](https://blog.packagecloud.io/eng/2018/02/21/attacks-against-secure-apt-repositories/)的文章。当前，Fedora 使用了多种方式确保从 Fedora 仓库分发给你的软件包是安全的。本文将深入讲解 Fedora 项目如何提升更新发布的安全性。注意，本文的分析仅仅适用于 Fedora 项目在 Fedora 中的默认软件仓库。

## 经过签名的软件包

Fedora 项目发布的所有 RPM 包都使用 GPG 进行签名。 DNF（或者 YUM）在从设置为 `gpgcheck=1` 的仓库中安装软件包时会验证软件包的签名，而这个设置在 Fedora 安装后是默认的。如果你从仓库之外安装软件包，例如手动下载，将不会自动检查签名。然而，可以手动通过 `rpm -K 文件名.rpm` 使用已经导入的 GPG 签名证书进行检查。

据此，对软件包进行签名能防止软件包被修改，或者被提供了一个完全不同的软件包。但仍不能防止“冻结攻击 (freeze attack) ”或者恶意的软件仓库信息。

## 经过签名的仓库

虽然多次有人请求，但是 Fedora 项目（至今）尚未对仓库进行签名。一个主要原因是 DNF/YUM 仓库信息缺少一个类似 "ValidUntil" （有效期）这样的属性。此外，DNF 下载和验证仓库信息使用的库 librepo 会将失效的 GPG 签名视作有效。这意味着迄今完全没有办法将仓库标记为只有一段有效期。

就此而言，尽管该方法可以防止仓库信息被篡改，却不能防止攻击者（或过时的镜像）提供的仓库太过古老的或者其中的软件包带有已知安全漏洞。

## Metalink

那么，如何保护 Fedora 用户？ Fedora 项目使用了名为 "Metalink" 的数据格式，为 YUM/DNF 提供可以下载软件的镜像列表，以及一系列可能拿到的相对较新 (reasonable fresh) 的 repomd.xml 校验值。repomd.xml 内包含其他全部被引用的文件的校验值。所有的仓库都配置为使用 TLS 从 Fedora 项目的服务器获取 metalink。

这儿我们要讨论下 “相对较新 (reasonable fresh) ” 这个部分。仓库每天被构建，校验值发送后最多保留 3 天。这样确保稍稍滞后的镜像源不会立马被标记为无效。

然而，当有显著的安全问题被公布后，一旦仓库中已经包含了对问题的修复，有个 "Emergency" 按钮用来告诉服务器所有旧的 repomd.xml 校验值都已经过期。

当这个按钮被点击，所有不包含最新仓库信息的镜像都会被认为是已经过期的。这确保 Fedora 用户尽快获得安全补丁。然而在一小段时间内仅有主源被标记为可信，直到其他镜像完成了数据同步。采取这个解决方案从而免得重要的修复被拖延导致的用户在该时间内受到攻击的风险。

因此，只要确保 Fedora 安装的仓库默认设置没有变更，你就能使用到新的仓库，并能尽快获得最重要的安全更新修复。

> 本文翻译自 Patrick Uiterwijk 的文章 [How Fedora Secures Package Delivery](https://fedoramagazine.org/fedora-secures-package-delivery/)。
