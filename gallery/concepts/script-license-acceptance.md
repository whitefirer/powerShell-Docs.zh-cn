---
ms.date: 06/09/2017
schema: 2.0.0
keywords: powershell
title: 需要为脚本接受许可证
ms.openlocfilehash: 6374c8c8536dd0c8f27580a5b8895b8db18424f9
ms.sourcegitcommit: e9ad4d85fd7eb72fb5bc37f6ca3ae1282ae3c6d7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2018
ms.locfileid: "34048164"
---
# <a name="requiring-license-acceptance-for-scripts"></a>需要为脚本接受许可证

脚本不支持接受许可证。 但是，支持其中的脚本依赖于需要接受许可证的模块的方案。

脚本命令 (Install-Script/Save-Script/Update-Script) 支持新参数 -AcceptLicense，此参数的行为就好像用户看到了许可证。 如果未指定 -AcceptLicense，系统将向用户显示依赖模块的 license.txt，并提示用户接受许可证。

## <a name="examples"></a>示例

### <a name="example-1-install-script-with-dependencies-requiring-license-acceptance"></a>示例 1：安装脚本及需要接受许可证的依赖项

脚本“ScriptRequireLicenseAcceptance”依赖于“ModuleRequireLicenseAcceptance”模块。 系统提示用户接受许可证。

```PowerShell
PS> Install-Script -Name ScriptRequireLicenseAcceptance

License Acceptance
MIT License 2.0
Copyright (c) 2016 PowerShell Team
Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software.

Do you accept the license terms for module 'ModuleRequireLicenseAcceptance'.
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"):
```

### <a name="example-2-install-script-with-dependencies-requiring-license-acceptance-and--acceptlicense"></a>示例 2：安装脚本及需要接受许可证和 -AcceptLicense 的依赖项

脚本“ScriptRequireLicenseAcceptance”依赖于“ModuleRequireLicenseAcceptance”模块。 系统不提示用户接受许可证，因为已指定 -AcceptLicense。

```PowerShell
PS> Install-Script -Name ScriptRequireLicenseAcceptance -AcceptLicense
```

## <a name="more-details"></a>详细信息

- [模块的需要接受许可证支持](module-license-acceptance.md)
- [PowerShellGallery 上的需要接受许可证支持](../how-to/working-with-items/items-that-require-license-acceptance.md)
- [在部署到 Azure 自动化时需要接受许可证](../how-to/working-with-items/deploy-to-azure-automation.md)