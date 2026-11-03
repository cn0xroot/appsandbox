<img width="1028" height="446" alt="App Sandbox" src="https://github.com/user-attachments/assets/f66dcaff-201e-4cf5-8e23-bc480338f353" />

[English](README.md) | **简体中文**

# App Sandbox

App Sandbox 是一款面向 Windows 和 macOS 的虚拟机应用,专注于性能与易用性。

Windows 功能:
- 支持 Windows 11 家庭版或专业版,无需 Hyper-V
- 支持 Windows 11 或 Ubuntu 26.04 LTS 虚拟机
- 零接触安装
- 复制和粘贴
- 双声道音频
- 通过半虚拟化实现 GPU 加速(GPU-PV),支持 DirectX 12(仅 Windows 客户机)、OpenGL、Vulkan、CUDA、OpenCL
- 支持 GPU 硬件视频解码/编码
- 通过 Hyper-V socket 代理进行 SSH(无需网络)
- 快照
- 固定 1080P60 显示
- 宿主机到客户机的热键支持
- 可联网或离线完成部署与启动
- 借助嵌套虚拟化,可在虚拟机内运行 Claude Cowork 和 Docker
- 无头(Headless)模式:本地 HTTP/JSON API 加无第三方依赖的 Python SDK(`asb.py`),可通过编程方式创建并驱动 GPU 加速虚拟机

Mac 功能:
- 支持 macOS 或 Windows 11 ARM 虚拟机
- Windows 11 和 macOS 均支持零接触安装
- 复制和粘贴
- 双声道音频
- macOS:通过半虚拟化实现 GPU 加速,支持 Metal;Windows:无 GPU 加速
- 通过 virtio-vsock 进行 SSH(无需网络)
- Windows 11 客户机固定 1080P60 显示,macOS 客户机动态调整显示尺寸
- 可联网或离线完成部署与启动
- 无头模式:与 Windows 相同的本地 HTTP/JSON API 和 Python SDK(`asb.py`),可通过编程方式驱动 macOS 和 Windows 11 虚拟机

系统要求:
搭载 x64 处理器的 Windows 11,或 macOS 27 及更高版本(M 系列芯片)

## 关于 App Sandbox

App Sandbox 可以创建并运行完整的桌面虚拟机——Windows 11、Ubuntu 和 macOS。它免费开源(MIT 协议),以预编译二进制形式发布(Windows 版经 EV 签名,macOS 版经 Apple 开发者签名),下载地址见 [Releases](https://github.com/jamesstringer90/appsandbox/releases) 页面。使用方式有两种:图形界面,或通过本地 API 配合无依赖的 Python SDK(`asb.py`)以编程方式控制。无论哪种方式,你只需选择操作系统并指定安装镜像,App Sandbox 就会准备磁盘、执行无人值守安装并启动客户机。Windows 和 Ubuntu 需要你自备 ISO;macOS 客户机会自动下载恢复镜像。它可运行在 Windows 11(x64)电脑或 Apple Silicon Mac 上,包括笔记本。

**可在 Windows 11 家庭版运行,无需 Hyper-V。** 在 Windows 上,App Sandbox 不使用 Hyper-V 或 Hyper-V 管理器,而是通过 Windows 主机计算服务(HCS)和主机计算网络(HCN)API 创建并运行虚拟机,只需要启用*虚拟机平台*功能。Hyper-V 仅限 Windows 11 专业版和企业版,而虚拟机平台在 Windows 11 家庭版上也可用,因此不需要 Windows 11 专业版许可证。虚拟机平台与 WSL2 使用的是同一个 Windows 功能。

**适用场景:**
- 完整的 Windows 或 Ubuntu **桌面**虚拟机——带 GPU 加速(OpenGL、Vulkan、CUDA)的完整图形操作系统——用于在干净系统上构建和测试自己的软件。
- 在与主系统和文件隔离的虚拟机中运行程序。
- 开发和测试 Windows 驱动:内置的**测试模式**会在客户机内启用测试签名,宿主机的安全启动和引导配置保持不变。
- 在独立虚拟机中运行计算机操作类 AI 智能体(例如 Claude 的 computer use 模型)。App Sandbox 提供桌面和虚拟机;智能体自身的模型循环运行在客户机内。多个智能体可同时运行,每个使用各自的虚拟机,一台笔记本或台式机即可承载。
- 通过图形界面或无头 API 创建 GPU 加速(**GPU-PV**)虚拟机。
- 用代码脚本化虚拟机生命周期——创建、快照、分支、SSH 连接、删除 GPU 加速虚拟机——使用 Python SDK 完成。

**无头 API。** 图形界面会自动在 `127.0.0.1` 上开放本地 HTTP/JSON API。`appsandbox.exe --headless`(macOS 上为 `sudo /Applications/AppSandbox.app/Contents/MacOS/AppSandbox --headless`)会运行相同的核心,但不显示管理界面,也不自动打开查看器。同样支持 `-headless`;同一时间只能运行一个核心进程。无第三方依赖的 Python SDK(`asb.py`)对其进行了封装:创建 GPU 加速虚拟机(GPU-PV 是创建时的 `gpuMode` 选项)、通过自动部署的密钥 SSH 登录(Windows、Ubuntu 或 macOS 客户机)、快照与分支、打开实时显示,并可同时运行多台虚拟机——并发数量受宿主机 CPU、内存和显存限制,没有固定上限。该守护进程仅限单机,只驱动其所在机器上的虚拟机,不会配置云端虚拟机;如需用于 CI,请在自托管 runner 上运行。快照和分支仅支持 Windows 宿主机。模板支持 Windows 宿主机上的 Windows 客户机,以及 Mac 宿主机上的 macOS 客户机,可避免重新安装系统。虚拟机会一直保留,直到被删除。快照是在虚拟机停止状态下对磁盘状态做的检查点,分支则从快照派生出一个可写磁盘,用于运行不同的操作路径。完整的 API 参考和可运行示例见 [`tools/headless-api/`](tools/headless-api/)。

**工作原理:** 本仓库同时也是一个使用 Windows HCS/HCN API 和 Apple Virtualization.framework 以编程方式创建完整桌面虚拟机的可运行示例。在 Windows 上,App Sandbox 将手工构建的 HCS 虚拟机描述文档提交给 `computecore.dll` / `computenetwork.dll`——这正是 WSL2 和 Windows 沙盒底层使用的 HCS/HCN 层——而不是通过 Hyper-V 管理器。GPU 加速使用 GPU 半虚拟化(GPU-PV)把宿主机已安装的 GPU 共享给客户机——与 WSL2 走的是同一条路径,而非独占直通(不需要 VFIO/IOMMU,也不需要第二块 GPU)。Windows 客户机支持 DirectX 12、OpenGL、Vulkan、CUDA 和 OpenCL;Ubuntu 客户机相同但不含 DirectX;macOS 客户机支持 Metal。自定义的 **IddCx** 间接显示驱动和虚拟音频设备负责传输画面与声音,客户机与宿主机之间的剪贴板、音频、输入和 SSH 都通过 **Hyper-V socket** 传输。客户机磁盘由仓库内置工具构建,支持 ext4、squashfs、qcow2 和 VHDX。在 macOS 上,App Sandbox 使用 Apple 的 **Virtualization.framework**(`VZVirtualMachine`、`VZMacOSInstaller`)和 **virtio-vsock**。在 Windows 上,Linux(Ubuntu)客户机通过自定义 DRM/KMS 内核模块(`asb_drm`)、微软 WSL2 的 `dxgkrnl` 以及自定义构建的 Mesa 使用 GPU。应用本身是原生 C / Objective-C,界面采用 HTML/JS(Windows 上为 WebView2,macOS 上为 WKWebView)。

macOS 上的 Windows 客户机使用带 HVF 加速的 **QEMU 11.1.3**,源码来自我们公开的 [asb-qemu 分支](https://github.com/jamesstringer90/asb-qemu/tree/asb-ivshmem-11.1.3)。该分支保留了我们的四项 ivshmem 修改,包括对 macOS 的支持和可选的 `x-device-id` 覆盖。QEMU、其依赖库和固件都随 macOS 应用一起发布。固定的源码版本和重新构建方法见 [QEMU 构建说明](vendor/qemu-ivshmem/README.md)。

# 使用提示

默认用户名为 `user`,默认密码为 `test123`。

[Windows] 启用热键或将虚拟机静音:连接到虚拟机后,右键点击连接窗口的标题栏。

[Windows] 需要高性能远程桌面来远程访问虚拟机?[Phaze](https://phaze.app) 效果不错。

[Windows] 可以在 App Sandbox 的 Windows 虚拟机内运行 gpu-test.exe 来检查 GPU-PV 驱动配置是否正常。它会显示一个包含 6 个旋转立方体的窗口,分别使用不同的渲染引擎(D3D9、D3D10、D3D11、D3D12、OpenGL 和 Vulkan)。如果某个渲染 API 失败,对应的立方体就不会正常旋转。下载:[gpu-test.zip](https://github.com/jamesstringer90/appsandbox/releases/download/v0.1.2/gpu-test.zip)。注意:渲染 API 成功说明 GPU-PV 已生效,但有些游戏或应用的编写方式无法正确检测 GPU-PV 系统,仍然可能报错。

## 分享修复与功能

感谢你分享修复和想法!欢迎提交 Pull Request,包括来自你自己分支的 AI 辅助改动,但它们仅作为建议,不太可能被正式审查或合并;对改动有帮助的贡献会获得署名。详情见[贡献指南](CONTRIBUTING.md)。

## 许可证

AppSandbox 采用 [MIT 许可证](LICENSE)——版权所有 (c) 2026 James Stringer。

项目捆绑了**不**适用 MIT 许可证的第三方组件,它们保留各自的条款——其中最主要的是微软 WSL2 `dxgkrnl` GPU 驱动(GPL-2.0)和内嵌的 XZ 解码器(0BSD)。完整列表见 [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md)。AppSandbox 自带的 Linux DRM 内核模块(`tools/linux/asb_drm/`)采用 MIT OR GPL-2.0 双许可证,以便链接内核中仅限 GPL 的符号。

## 致谢

除了我自己构建 [Easy-GPU-PV](https://github.com/jamesstringerparsec/Easy-GPU-PV) 的经验外,[NanaBox](https://github.com/M2Team/NanaBox) 对理解 HCS 帮助很大。

## 作者

[James Stringer](https://www.linkedin.com/in/jamesstringerphotography/)——[Easy-GPU-PV](https://github.com/jamesstringerparsec/Easy-GPU-PV) 的作者。
