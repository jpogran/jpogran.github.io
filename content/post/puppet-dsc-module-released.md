---
date: 2016-01-20T14:13:06-04:00
title: "Puppet DSC module is now supported!"
subtitle: "Puppet DSC module is now supported!"
summary: "Puppet DSC module is now supported!"
aliases: [
  "/blog/dsc/puppet/puppet-dsc-module-released",
  "2016-01-20-puppet-dsc-module-released",
]
thumbnail: img/Puppet-Logo-Amber-White-lg.jpg
featureImage: img/Puppet-Logo-Amber-White-lg.jpg
tags: [ dsc, puppet ]
---

The new Puppet module I've been working so hard on is finally released as [supported](https://forge.puppetlabs.com/puppetlabs/dsc)! It was alot of hard work to get to here and now. Working closely with my coworkers and Microsoft to make this happen was the best learning experience and the most interesting thing I've done in a long time.

There are a host of improvements to the module since the first unsupported release, but the one I'm most excited about is the speed improvements. Instead of starting a PowerShell process for each reasource being executed, we only start one PowerShell process instance and continue to use it for the duration of the Puppet run. This results in a huge speed improvement as well as a decrease in system utilization. you can read more about what was done [here](https://puppetlabs.com/blog/powershell-dsc-module-now-supported).
