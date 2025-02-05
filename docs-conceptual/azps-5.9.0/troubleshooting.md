---
title: Troubleshooting the Azure Az PowerShell module
description: Troubleshooting the Azure Az PowerShell module.
ms.devlang: powershell
ms.topic: conceptual
ms.date: 09/21/2021
ms.custom: devx-track-azurepowershell
ms.service: azure-powershell
---

# Troubleshooting the Azure Az PowerShell module

## Enable debug logging

One of the first steps you should take in troubleshooting a problem with the Azure Az PowerShell
module is to enable debug logging.

To enable debug logging on a per command basis, specify the **Debug** parameter.

```azurepowershell-interactive
Get-AzResource -Name 'DoesNotExist' -Debug
```

To enable debug logging for an entire PowerShell session, you set the value of the
**DebugPreference** variable to `Continue`.

```azurepowershell-interactive
$DebugPreference = 'Continue'
```

## Command found but could not be loaded

The following message is returned by PowerShell when you attempt to run any of the Az PowerShell commands.

```Output
Connect-AzAccount: The 'Connect-AzAccount' command was found in the module 'Az.Accounts', but the module could not be loaded. For more information, run 'Import-Module Az.Accounts'.
```

This message occurs when you have both the Az and AzureRM PowerShell modules installed on the same
Windows-based system and they exist in the
[$env:PSModulePath](/powershell/module/microsoft.powershell.core/about/about_psmodulepath) for the
same version of PowerShell.

> [!IMPORTANT]
> When AzureRM is installed in the `AllUsers` scope of Windows PowerShell, it's installed in a
> location that's part of the `$env:PSModulePath` for PowerShell 7. This is not supported due to
> conflicts between the AzureRM and Az PowerShell modules.

Both Az and AzureRM may coexist on the same Windows system, but only if AzureRM is installed in the
`CurrentUser` scope of Windows PowerShell and Az installed in PowerShell 7. For more information, see
[Install the Azure Az PowerShell module](/powershell/azure/install-az-ps).

## On MacOS, an error returns when KeyChain authorization fails

When running Azure PowerShell on MacOS, you might encounter an error message while attempting to
sign in to your Azure account from a PowerShell session.

```txt
DeviceCodeCredential authentication failed: Persistence check failed. Reason: KeyChain authorization/authentication failed. .Error code: -25293. OS error code -25293.
```

As a workaround for this issue, you can disable storing credentials between sessions by running
the following command. After making this change however, you will need to run `Connect-AzAccount`
each time you start a new PowerShell session.

```powershell
Disable-AzContextAutosave
```

## Service Principal IdentifierUri verified domain error

Error: _Values of identifierUris property must use a verified domain of the organization or its
subdomain_ is displayed when running `New-AzADServicePrincipal` or `New-AzADApplication`.

Due to the Azure Active Directory breaking change requiring [AppId Uri in single tenant applications
to require use of default scheme or verified
domains](/active-directory/develop/reference-breaking-changes#appid-uri-in-single-tenant-applications-will-require-use-of-default-scheme-or-verified-domains)
you must upgrade the [Az.Resources](https://www.powershellgallery.com/packages/Az.Resources) module
to version 4.1.0 or later to continue using `New-AzADServicePrincipal` or `New-AzADApplication` cmdlets.

You can also upgrade to Az PowerShell module version 6.0 or greater.

### Timeline

The requirement will be in effect starting 10/15/2021.

### Impacted versions

The following versions of Azure PowerShell are impacted by the AzureAD breaking change:

- Az.Resources PowerShell module version 3.5.1-preview or lesser.
- Az PowerShell module version 5.9.0 or lesser.

If you are still encountering issues after upgrading, feel free to open an
[issue](https://github.com/Azure/azure-powershell/issues/new?assignees=&labels=needs-triage&template=az-module-bug-report.md&title=).

### Workaround

If you cannot upgrade to the PowerShell modules described above, you may follow those steps when
creating a service principal:

- If needed, [add your custom domain name using Azure Active Directory portal](/active-directory/fundamentals/add-custom-domain)
- Create an application with an accepted IdentifierUri
- Create a service principal referring this application
