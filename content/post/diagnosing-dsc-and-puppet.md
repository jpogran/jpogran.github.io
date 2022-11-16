---
date: 2017-10-19T14:13:06-04:00
title: "Diagnosing DSC and Puppet Executions"
subtitle: "How to figure out what's going on when Puppet uses DSC to ensure desired state"
description: "How to figure out what's going on when Puppet uses DSC"
aliases: [
  "/blog/dsc/puppet/diagnosing-dsc-and-puppet/",
  "2017-10-19-diagnosing-dsc-and-puppet"
]
thumbnail: img/DAG_tagline_logo_background.png
featureImage: img/DAG_tagline_logo_background.png
shareImage: img/DAG_tagline_logo_background.png
tags: [ dsc, puppet, featured ]
category: 'technical'
series: ['featured']
toc: true
---

While developing your Puppet manifests using the Puppet DSC module you might run into a problem with one of the DSC Resources.

The problem shows itself every execution, when Puppet says that the state has drifted and needs to be corrected. When you look at the state of the target node it appears to be correct, something must be wrong with Puppet, right? Well, as with most things, the answer is more complicated than it appears.

The [Puppet dsc module](https://forge.puppet.com/puppetlabs/dsc) works by invoking DSC directly using the `Invoke-DscResource` cmdlet using `TEST`, passing on the properties and values you entered into the Puppet manifest. DSC then interrogates the target node for compliance, and reports whether it is in the desired state or not. If it's not in the desired state, then Puppet invokes `Invoke-DscResource` with a `SET` and DSC corrects the drift. It's up to DSC to both determine state and correctly set the state.

In our case above, it's DSC that is constantly reporting the target node is not in the desired state, fixing it, then reporting it's drifted in the next run. Puppet is just passing along the message, so to speak.

## Where to Start

So how do you figure out what is going wrong?

The first step is to run puppet with your same manifest, but in debug mode:

```powershell
PS C:\Users\Administrator\Desktop> puppet apply --debug wakka.pp
```

This outputs all the normal Puppet debug information you're used to, along with the debug information from the DSC execution. This extra information is largely the PowerShell code the Puppet dsc module generated in order to use `Invoke-DscResource`, which can be skipped at this moment, but will be helpful later on. Right now we're interested in the last message from the 'first pass', the `TEST` method:

```powershell
VERBOSE: Time taken for configuration job to complete is 2.478 seconds
{"rebootrequired":false,"indesiredstate":false,"errormessage":""}
```

This means DSC tested the state of the target node and found it non-compliant. But what was non-compliant?

## Showing Debug Information

Unfortunately DSC does not surface more information without some extra work. We have to do some digging, and rely on the DSC Resource author to have implemented some Verbose logging.

We go back to our output and copy the PowerShell code we saw earlier that invokes DSC:

```powershell
PS C:\Users\Administrator\Desktop> $invokeParams = @{
  Name          = 'xFirewall'
  Method        = 'test'
  Property      = @{
    name = 'my_firewall'
    enabled = 'True'
    remoteaddress = @('130.230.0.0/16')
  }
  ModuleName = @{
    ModuleName      = "C:/ProgramData/PuppetLabs/code/environments/production/modules/dsc/lib/puppet_x/dsc_resources/xNetworking/xNetworking.psd1"
    RequiredVersion = "5.1.0.0"
  }
}
```

In order to get more information, we add the `Verbose` switch:

```powershell
PS C:\Users\Administrator\Desktop> $invokeParams = @{
  Verbose       = $true
  Name          = 'xFirewall'
  Method        = 'test'
  Property      = @{
    name = 'my_firewall'
    enabled = 'True'
    remoteaddress = @('130.230.0.0/16')
  }
  ModuleName = @{
    ModuleName      = "C:/ProgramData/PuppetLabs/code/environments/production/modules/dsc/lib/puppet_x/dsc_resources/xNetworking/xNetworking.psd1"
    RequiredVersion = "5.1.0.0"
  }
}
```

Then we execute that inside our PowerShell console (without using Puppet):

```powershell
PS C:\Users\Administrator\Desktop> $result = Invoke-DscResource @invokeParams
VERBOSE: Perform operation 'Invoke CimMethod' with following parameters, ''methodName' = Resourcetest,'className' = MSFT_DSCLocalConfigurationManager,'namespaceName' = root/Microsoft/Windows/DesiredStateConfiguration'.
VERBOSE: An LCM method call arrived from computer WINVAGR-MNSKSOE with user sid S-1-5-21-850066514-2495575711-3657759813-500.
VERBOSE: [WINVAGR-MNSKSOE]: LCM:  [ Start  Test     ]  [[xFirewall]DirectResourceAccess]
VERBOSE: [WINVAGR-MNSKSOE]: [[xFirewall]DirectResourceAccess] Test-TargetResource: Checking settings for firewall rule with Name 'my_firewall'.
VERBOSE: [WINVAGR-MNSKSOE]: [[xFirewall]DirectResourceAccess] Test-TargetResource: Find firewall rule with Name 'my_firewall'.
VERBOSE: [WINVAGR-MNSKSOE]: [[xFirewall]DirectResourceAccess] Test-TargetResource: Check each defined parameter against the existing firewall rule with Name 'my_firewall'.
VERBOSE: [WINVAGR-MNSKSOE]: [[xFirewall]DirectResourceAccess] Get-FirewallRuleProperty: Get all the properties and add filter info to rule map.
VERBOSE: [WINVAGR-MNSKSOE]: [[xFirewall]DirectResourceAccess] Test-RuleProperties: RemoteAddress property value '130.230.0.0/255.255.0.0' does not match desired state '130.230.0.0/16'.
VERBOSE: [WINVAGR-MNSKSOE]: [[xFirewall]DirectResourceAccess] Test-RuleProperties: Test Firewall rule with Name 'my_firewall' returning False.
VERBOSE: [WINVAGR-MNSKSOE]: [[xFirewall]DirectResourceAccess] Test-TargetResource: Check Firewall rule with Name 'my_firewall' returning False.
VERBOSE: [WINVAGR-MNSKSOE]: LCM:  [ End    Test     ]  [[xFirewall]DirectResourceAccess] False in 0.6260 seconds.
VERBOSE: [WINVAGR-MNSKSOE]: LCM:  [ End    Set      ]    in  0.6420 seconds.
VERBOSE: Operation 'Invoke CimMethod' complete.
VERBOSE: Time taken for configuration job to complete is 0.717 seconds
```

This is a raw DSC run, doing the same exact stuff we had Puppet tell it to do before, with the addition of telling DSC to output **VERBOSE** information. Not all DSC Resources output **VERBOSE** information, it's not required so it will be hit and miss looking for it on other DSC Resources.

We're lucky, this particular DSC Resource outputs useful information to the **VERBOSE** PowerShell data stream, so we can see which property is not in the desired state.

```powershell
VERBOSE: [WINVAGR-MNSKSOE]: [[xFirewall]DirectResourceAccess] Test-RuleProperties: RemoteAddress property value '130.230.0.0/255.255.0.0' does not match desired state '130.230.0.0/16'.
```

And we've found our bug.
## Next Steps

There is something wrong in the way we're setting the value of **RemoteAddress**. We're saying it should be `130.230.0.0/16`, but it's being persisted as `130.230.0.0/255.255.0.0`. Knowing why this is happening requires further knowledge of how **New-NetFirewallRule** works and the evalautaion logic of the **Test-TargetResource** inside the **xFirewall DSC Resource**, but we've shown the problem lies there and not in Puppet. The next step is to open a issue in the **xNetworking** Github repo and work with the team there to resolve the issue.
