---
ms.date: 06/12/2017
ms.topic: conceptual
keywords: wmf,powershell,安装程序
title: WMF 5.1 中的 DSC 改进
ms.openlocfilehash: 32bdde6d43d17cc76c454fe10b00097753a9eebe
ms.sourcegitcommit: 2d9cf1ccb9a653db7726a408ebcb65530dcb1522
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/19/2018
ms.locfileid: "34309536"
---
# <a name="improvements-in-desired-state-configuration-dsc-in-wmf-51"></a>WMF 5.1 中的 Desired State Configuration (DSC) 改进

## <a name="dsc-class-resource-improvements"></a>DSC 类资源改进

在 WMF 5.1 中，我们已解决下列已知问题：

- 如果基于类的 DSC 资源的 Get() 函数返回复杂/哈希表类型，则 Get-DscConfiguration 可能会返回空值 (null) 或错误。
- 如果在 DSC 配置中使用 RunAs 凭据，Get-DscConfiguration 将返回错误。
- 不能在复合配置中使用基于类的资源。
- 如果基于类的资源具有和自己类型一样的属性，Start-DscConfiguration 将挂起。
- 基于类的资源不能用作独占资源。

## <a name="dsc-resource-debugging-improvements"></a>DSC 资源调试改进

在 WMF 5.0 中，PowerShell 调试器不直接在基于类的资源方法（Get/Set/Test）处停止。
在 WMF 5.1 中，调试器在基于类的资源方法处停止，与遇到基于 MOF 的资源方法时一样。

## <a name="dsc-pull-client-supports-tls-11-and-tls-12"></a>DSC 请求客户端支持 TLS 1.1 和 TLS 1.2

以前，DSC 请求客户端仅支持基于 https 连接的 SSL3.0 和 TLS1.0。
当强制使用更安全的协议时，请求客户端停止工作了。
在 WMF 5.1 中，DSC 请求客户端不再支持 SSL 3.0，而增加了对更安全的 TLS 1.1 和 TLS 1.2 协议的支持。

## <a name="improved-pull-server-registration"></a>改进的请求服务器注册

在 WMF 的早期版本中，在使用 ESENT 数据库时同时注册 DSC 请求服务器或向其报告请求，将导致 LCM 无法注册和/或报告。
在这种情况下，请求服务器上的事件日志显示错误“实例名称已使用”。
这是由于在多线程情况下使用不正确的模式访问 ESENT 数据库。
在 WMF 5.1 中已修复此问题。
在 WMF 5.1 中，并发注册或报告（涉及 ESENT 数据库）可以正常运行。
此问题仅适用于 ESENT 数据库，并不适用于 OLEDB 数据库。

## <a name="enable-circular-log-on-esent-database-instance"></a>在 ESENT 数据库实例上启用循环日志

在旧版 DSC PullServer 中，ESENT 数据库日志文件填满了 pullserver 的磁盘空间，因为数据库实例在创建时未启用循环日志。 在此版本中，可以视需要使用 pullserver 的 web.config 来控制实例的循环记录行为。 默认情况下，CircularLogging 设为 TRUE。

```
<appSettings>
    <add key="dbprovider" value="ESENT" />
    <add key="dbconnectionstr" value="C:\Program Files\WindowsPowerShell\DscService\Devices.edb" />
    <add key="CheckpointDepthMaxKB" value="512" />
    <add key="UseCircularESENTLogs" value="TRUE" />
  </appSettings>
```

## <a name="pull-partial-configuration-naming-convention"></a>拉取部分配置命名约定

在以前的版本中，部分配置的命名约定为请求服务器/服务中 mof 文件名应与本地配置管理器设置中指定的部分配置名称匹配，反过来后者必须与 MOF 文件中嵌入的配置名称匹配。

请参阅以下快照：

- 本地配置设置，定义了节点可以接收的部分配置。

![示例 metaconfiguration](../images/MetaConfigPartialOne.png)

- 部分配置定义示例

```powershell
Configuration PartialOne
{
    Node('localhost')
    {
        File test
        {
            DestinationPath = "$env:TEMP\partialconfigexample.txt"
            Contents = 'Partial Config Example'
        }
    }
}
PartialOne
```

- 生成的 MOF 文件中嵌入的“ConfigurationName”。

![生成的 mof 文件示例](../images/PartialGeneratedMof.png)

- 拉取配置存储库中的文件名

![配置存储库中的文件名](../images/PartialInConfigRepository.png)

Azure 自动化服务名称生成的 MOF 文件名为 `<ConfigurationName>.<NodeName>.mof`。
因此，下面的配置编译到 PartialOne.localhost.mof 中。

这样将无法从 Azure 自动化服务中提取一个部分配置。

```powershell
Configuration PartialOne
{
    Node('localhost')
    {
        File test
        {
            DestinationPath = "$env:TEMP\partialconfigexample.txt"
            Contents = 'Partial Config Example'
        }
    }
}
PartialOne
```

在 WMF 5.1 中，请求服务器/服务中的部分配置可以命名为 `<ConfigurationName>.<NodeName>.mof`。
此外，如果一台计算机从请求服务器/服务中提取一个配置，那么请求服务器配置存储库上的配置文件可以有任何文件名。
借助这种命名灵活性，可以使用 Azure 自动化服务管理部分节点（节点的一些配置来自 Azure 自动化 DSC），也可以在本地管理部分配置。

下面的元配置将节点设置为可以在本地管理，也可以由 Azure 自动化服务管理。

```powershell
[DscLocalConfigurationManager()]
Configuration RegistrationMetaConfig
{
    Settings
    {
        RefreshFrequencyMins = 30
        RefreshMode = "PULL"
    }

    ConfigurationRepositoryWeb web
    {
        ServerURL =  $endPoint
        RegistrationKey = $registrationKey
        ConfigurationNames = $configurationName
    }

    # Partial configuration managed by Azure Automation service.
    PartialConfiguration PartialConfigurationManagedByAzureAutomation
    {
        ConfigurationSource = "[ConfigurationRepositoryWeb]Web"
    }

    # This partial configuration is managed locally.
    PartialConfiguration OnPremisesConfig
    {
        RefreshMode = "PUSH"
        ExclusiveResources = @("Script")
    }

}

RegistrationMetaConfig
Set-DscLocalConfigurationManager -Path .\RegistrationMetaConfig -Verbose
```

## <a name="using-psdscrunascredential-with-dsc-composite-resources"></a>在 DSC 复合资源中使用 PsDscRunAsCredential

我们新增了对 DSC [复合](https://msdn.microsoft.com/en-us/powershell/dsc/authoringresourcecomposite)资源使用 [PsDscRunAsCredential](https://msdn.microsoft.com/cs-cz/powershell/dsc/runasuser) 的支持。

现在可以在使用配置内的复合资源时，指定 PsDscRunAsCredential 值。
如果指定了该值，则可以 RunAs 用户身份运行复合资源内部的所有资源。
如果复合资源调用了另一个复合资源，也将以 RunAs 用户运行该复合资源中的所有资源。
RunAs 凭据可以传播到复合资源层次结构的任一级别。
如果复合资源内的任何资源指定了自己的 PsDscRunAsCredential 值，那么会在配置编译期间生成合并错误。

本示例演示了在 PSDesiredStateConfiguration 模块中包含的 [WindowsFeatureSet](https://msdn.microsoft.com/en-us/powershell/wmf/dsc_newresources) 复合资源中该属性的使用。

```powershell
Configuration InstallWindowsFeature
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    Node $AllNodes.NodeName
    {
        WindowsFeatureSet features
        {
            Name = @("Telnet-Client","SNMP-Service")
            Ensure = "Present"
            IncludeAllSubFeature = $true
            PsDscRunAsCredential = Get-Credential
        }
    }
}

$configData = @{
    AllNodes = @(
        @{
            NodeName             = 'localhost'
            PSDscAllowDomainUser = $true
            CertificateFile      = 'C:\publicKeys\targetNode.cer'
            Thumbprint           = '7ee7f09d-4be0-41aa-a47f-96b9e3bdec25'
        }
    )
}

InstallWindowsFeature -ConfigurationData $configData
```

## <a name="dsc-module-and-configuration-signing-validations"></a>DSC 模块和配置签名验证

在 DSC 中，从请求服务器中将配置和模块分发到托管计算机。
如果请求服务器受到攻击，攻击者可能修改请求服务器上的配置和模块，并将其分发到所有托管节点，将损害所有这些节点。

在 WMF 5.1 中，DSC 支持验证目录和配置文件 (.MOF) 上的数字签名。
此功能可防止节点执行受信任的签名者未签名的配置或模块文件，或是在受信任的签名者签名后被篡改的配置或模块文件。

### <a name="how-to-sign-configuration-and-module"></a>如何对配置和模块进行签名

***
* 配置文件 (.MOF)：已扩展现有的 PowerShell cmdlet [Set-AuthenticodeSignature](https://technet.microsoft.com/library/hh849819.aspx) 用于支持 MOF 文件签名。
* 模块：使用以下步骤对相应的模块目录进行签名，从而完成模块签名：
    1. 创建目录文件：目录文件包含加密哈希或指纹的集合。
       每个指纹对应于模块中包含的一个文件。
       增加了新的 cmdlet [New-FileCatalog](https://technet.microsoft.com/library/cc732148.aspx)，支持用户为其模块创建目录文件。
    2. 对目录文件进行签名：使用 [Set-AuthenticodeSignature](https://technet.microsoft.com/library/hh849819.aspx) 对目录文件进行签名。
    3. 将目录文件放在模块文件夹中。
按照约定，模块目录文件应放在与模块同名的模块文件夹下面。

### <a name="localconfigurationmanager-settings-to-enable-signing-validations"></a>用于启用签名验证的 LocalConfigurationManager 设置。

#### <a name="pull"></a>请求

节点的 LocalConfigurationManager 根据其当前设置执行模块和配置的签名验证。
默认情况下，签名验证处于禁用状态。
你可以将 SignatureValidation 块添加到节点的元配置定义来启用签名验证，如下所示：

```powershell
[DSCLocalConfigurationManager()]
Configuration EnableSignatureValidation
{
    Settings
    {
        RefreshMode = 'PULL'
    }

    ConfigurationRepositoryWeb pullserver{
      ConfigurationNames = 'sql'
      ServerURL = 'http://localhost:8080/PSDSCPullServer/PSDSCPullServer.svc'
      AllowUnsecureConnection = $true
      RegistrationKey = 'd6750ff1-d8dd-49f7-8caf-7471ea9793fc' # Replace this with correct registration key.
    }
    SignatureValidation validations{
        # If the TrustedStorePath property is provided then LCM will use the custom path. Otherwise, the LCM will use default trusted store path (Cert:\LocalMachine\DSCStore) to find the signing certificate.
        TrustedStorePath = 'Cert:\LocalMachine\DSCStore'
        SignedItemType = 'Configuration','Module'         # This is a list of DSC artifacts, for which LCM need to verify their digital signature before executing them on the node.
    }

}
EnableSignatureValidation
Set-DscLocalConfigurationManager -Path .\EnableSignatureValidation -Verbose
```

在节点上设置上述元配置可以对下载的配置和模块进行签名验证。
本地配置管理器执行以下步骤来验证数字签名。

1. 验证配置文件 (.MOF) 上的签名是否有效。
   它使用在 5.1 中扩展的 PowerShell cmdlet [Get-AuthenticodeSignature](https://technet.microsoft.com/library/hh849805.aspx) 来支持 MOF 签名验证。
2. 验证授权签名人的证书颁发机构是否可信任。
3. 将配置的模块/资源依赖项下载到临时位置。
4. 验证模块中是否包含目录的签名。
    * 查找 `<moduleName>.cat` 文件并使用 cmdlet [Get-AuthenticodeSignature](https://technet.microsoft.com/library/hh849805.aspx) 验证其签名。
    * 验证对签名人进行身份验证的证书颁发机构是否可信任。
    * 验证未使用新的 cmdlet [Test-FileCatalog](https://technet.microsoft.com/library/cc732148.aspx) 篡改模块的内容。
5. 将模块安装到 $env:ProgramFiles\WindowsPowerShell\Modules\
6. 进程配置

> 注意：仅在第一次将配置应用到系统或下载并安装模块时，才执行对模块目录和配置的签名验证。
一致性运行不会验证 Current.mof 或其模块依赖项的签名。
如果在任何阶段验证失败（例如，如果从请求服务器请求获取的配置未签名），那么配置处理会终止，同时显示以下错误，并删除所有临时文件。

![错误输出配置示例](../images/PullUnsignedConfigFail.png)

同样，如果请求获取的模块包含未签名的目录，也会导致以下错误生成：

![错误输出模块示例](../images/PullUnisgnedCatalog.png)

#### <a name="push"></a>推送

通过使用推送提供的配置可能会在其提供到节点之前在源处被篡改。
本地配置管理器对推送或发布的配置执行类似的签名验证步骤。
下面是针对推送的签名验证的完整示例。

- 启用针对节点的签名验证。

```powershell
[DSCLocalConfigurationManager()]
Configuration EnableSignatureValidation
{
    Settings
    {
        RefreshMode = 'PUSH'
    }
    SignatureValidation validations{
        TrustedStorePath = 'Cert:\LocalMachine\DSCStore'
        SignedItemType =  'Configuration','Module'
    }

}
EnableSignatureValidation
Set-DscLocalConfigurationManager -Path .\EnableSignatureValidation -Verbose
```

- 创建示例配置文件。

```powershell
# Sample configuration
Configuration Test
{

    File foo
    {
        DestinationPath =  "$env:TEMP\signingTest.txt"
        Contents = "ABC"
    }
}
Test
```

- 尝试将未签名的配置文件推送到节点。

```powershell
Start-DscConfiguration -Path .\Test -Wait -Verbose -Force
```

![ErrorUnsignedMofPushed](../images/PushUnsignedMof.png)

- 使用代码签名证书对配置文件进行签名。

![SignMofFile](../images/SignMofFile.png)

- 请尝试推送已签名的 MOF 文件。

![SignMofFile](../images/PushSignedMof.png)
