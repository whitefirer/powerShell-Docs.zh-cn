---
ms.date: 08/12/2017
ms.topic: conceptual
keywords: wmf,powershell,安装程序
title: WMF 5.1 发行说明
ms.openlocfilehash: 3512d2e80501a596e1fd6d7b33d4d75286cef1b9
ms.sourcegitcommit: 54534635eedacf531d8d6344019dc16a50b8b441
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/16/2018
ms.locfileid: "34189782"
---
# <a name="windows-management-framework-wmf-51"></a>Windows Management Framework (WMF) 5.1 #

WMF 使用户能够将现有的 Windows 系统更新至随 Windows Server 2016 一起发行的 PowerShell、WMI、WinRM 和软件清单日志记录 (SIL) 组件的版本。

WMF 5.1 可安装在 Windows 7、Windows 8.1、Windows Server 2008 R2、2012 和 2012 R2 上，并提供了对 WMF 5.0 RTM 的大量改进，其中包括：

- 新 cmdlet：本地用户和组；Get-ComputerInfo
- PowerShellGet 改进包括强制签名模块和安装 JEA 模块
- PackageManagement 增加了对容器、CBS 安装程序、基于 EXE 的安装程序、CAB 包的支持
- DSC 和 PowerShell 类的调试改进
- 安全增强包括强制执行来自请求服务器和使用 PowerShellGet cmdlet 时的目录签名模块
- 响应大量的用户请求和问题

若要了解此版本中的新增功能，请浏览[新方案和功能](https://docs.microsoft.com/en-us/powershell/wmf/5.1/scenarios-features)下列出的主题。

[安装和配置](https://docs.microsoft.com/en-us/powershell/wmf/5.1/install-configure)主题列出了安装和配置的相关要求，并提供 WMF 的安装说明。

[兼容性](https://docs.microsoft.com/en-us/powershell/wmf/5.1/compatibility)主题列出了可以在哪些 Windows 版本上安装哪些版本的 WMF。

[产品兼容性](https://docs.microsoft.com/en-us/powershell/wmf/5.1/productincompat)主题列出了目前尚未兼容 WMF 5.1 的 Microsoft 应用程序。

可以在 MSDN 文档中找到 WMF 组件的详细信息：

- [PowerShell 5.1](https://docs.microsoft.com/en-us/powershell/)
- [WMI](https://msdn.microsoft.com/en-us/library/jj152383(v=vs.85).aspx)
- [WinRM](https://msdn.microsoft.com/en-us/library/aa384426(v=vs.85).aspx)
- [软件清单日志记录](https://technet.microsoft.com/en-us/library/dn383584(v=ws.11).aspx)
