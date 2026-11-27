---
name: onnxruntime-linux-telemetry-off
description: "只有 import 之前设 ORT_DISABLE_TELEMETRY=1 才关得掉；附断网实测方法和三种关法的对照数字"
type: 技巧
scene: [写代码, 排查问题]
status: 已验证
reviewed: 2026-11-24
date: 2026-11-23
---

# 关掉 onnxruntime 在 Linux 上的遥测

**做法**：在 `import onnxruntime` **之前**设环境变量 `ORT_DISABLE_TELEMETRY=1`。

- 凡是在会断网或有数据外发限制的环境里用 onnxruntime（包括 rapidocr 等封装它的库），程序入口第一行就要设这个变量，systemd 的 Environment 里也设一次。
- 这两种做法无效：调 `onnxruntime.disable_telemetry_events()`，在 Linux 上无效；导入之后再设环境变量，也无效。
- 升级 onnxruntime 后，要按下面的方法重新断网实测。

断网实测的方法：`unshare -rn` 建一个只有回环的网络环境，在里面用 `strace -f -tt -e trace=network` 跑进程；连 53 端口的一律算外连，哪怕目标是 127.0.0.1（systemd-resolved 的本机桩解析器）。

**为什么**：2026-11-20 在一个识别服务的断网实测里查出来的。

- onnxruntime 1.32.0 的 Linux 版（PyPI manylinux 包）内置了 1DS 遥测。服务空闲时，它每隔几秒就解析一次 `mobile.events.data.microsoft.com`，上报地址是 `/OneCollector/1.0`。上报内容包括：设备 ID、操作系统、CPU、内存、程序完整路径（路径里带账号名）、网络类型；不含模型输入。
- 它会往主目录下写 `.cache/Microsoft/DeveloperTools/.onnxruntime/`，里面有 `deviceid`，还有一个 SQLite 格式的事件队列 `onnxruntime.db`。
- 这些连接是 C++ 代码直接调 glibc 发起的，Python 层的 socket 拦截看不到，只有 strace 能抓到。

三种做法实测对比，各跑 120 秒、看 DNS 查询次数：

- 不关：20 次；
- 设环境变量：0 次，也不写文件；
- 调 `onnxruntime.disable_telemetry_events()`：仍是 20 次，在 Linux 上无效；
- 另外，导入之后再设环境变量也无效：40 秒内 16 次。

实测环境：2026-11-20，onnxruntime 1.32.0（PyPI manylinux 包）。升级后先验证再用。

依据类型：真实运行（2026-11-20 断网实测）；评测记录（三种做法各跑 120 秒对照）。

**什么时候用，什么时候不用**：

- 用：在会断网或有数据外发限制的环境里用 onnxruntime，或用封装了它的库。
- 不用：原文没有记录。原文只在 Linux 版上实测过，别的平台没测。
