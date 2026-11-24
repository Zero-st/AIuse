---
name: headless-chrome-no-keyring-prompt
description: Linux 上登录时密钥环没有自动解锁的环境里，用脚本起 Chrome（含无头）每次都弹「解锁密钥环」；一律加 --password-store=basic
type: 技巧
scene: [自动化, 写文档]
status: 已验证
reviewed: 2026-11-24
date: 2026-11-23
---

# 起无头 Chrome 不弹密钥环

**做法**：在 Linux 上用脚本起 Chrome，包括无头模式，一律加参数 `--password-store=basic`。

- 新写任何起 Chrome 的脚本都加。
- 起 Chrome 的是一个不想改的工具（第三方工具改了，升级后会被覆盖），而它允许指定 Chrome 的可执行文件时：指向一个两行的包装脚本，内容是 `exec /usr/bin/google-chrome --password-store=basic "$@"`。
- 别用「把密钥环密码设成空」来绕开。多用户共享的机器上，密钥会以明文落盘。

**为什么**：在一台远程 X 会话的机器上（没有 XDG_CURRENT_DESKTOP），登录时 gnome-keyring 不会自动解锁。Chrome 146 每次启动都会经 D-Bus 读「Chrome Safe Storage」（ReadAlias→Unlock→Prompt），于是每起一次 Chrome 就弹一次「解锁密钥环」，headless 也照弹。弹窗点「取消」不影响渲染，Chrome 不等弹窗，但每次画图都会打断我。

当时排查过：Claude Code 本身（凭据存在它自己的凭据文件里）和凭据走 gpg-agent 的 SVN 都不碰密钥环，来源是画图链路里起 Chrome 的两个脚本。

先红后绿：不加参数时 4 次密钥服务调用并弹窗，加 `--password-store=basic` 后 0 次。

实测环境：2026-11-13，Chrome 146。升级后先验证再用。

依据类型：真实运行（2026-11-13 先红后绿实测）。

**什么时候用，什么时候不用**：

- 用：Linux 上登录时密钥环没有自动解锁的环境里（比如远程 X 会话），用脚本起 Chrome。
- 不用：原文没有记录。

**相关**：[多人共用机器上驱动浏览器自动化的坑](../50-经验教训/多人共用机器上驱动浏览器自动化的坑.md)，共用机器上起无头 Chrome 的其他坑和最小配方。
