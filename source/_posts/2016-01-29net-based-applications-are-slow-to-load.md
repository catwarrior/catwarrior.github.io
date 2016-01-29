title: .net based applications are slow to load
date: 2016-01-29 13:23:05
categories:
- coding
tags:
- asp.net
---

[.NET bases applications are slow to load or certain in-app tasks are slow](http://austintovey.blogspot.com/2012/01/net-bases-applications-are-slow-to-load.html)

This behaviour will occur with any .NET 1.1 and 2.0 assembly that is authenticode-signed, not only Measurement Studio assemblies. Digital signing is also referred to as code signing. Code signing a .NET library is strongly recommended by Microsoft, and Measurement Studio signs all of our ActiveX and .NET components.

Code signing of assemblies makes components tamper-proof and ensures that you know the identity of the component publisher.

The reason why this problem is occurring is because of the mechanism used by the .NET Common Language Runtime (CLR) to verify code-signed .NET assemblies. Part of the verification process requires an online look-up to check whether the certificate with which the assembly is signed has been revoked and is no longer valid. Windows does this by downloading a CRL (Certificate Revocation List). The first time a code-signed assembly is loaded by the .NET CLR, the CRL is downloaded from the certificate provider's server and cached on the system.

When the .NET CLR loads a code-signed assembly and is unable to reach the CRL distribution point, it records the failure as an inability to provide the assembly evidence that it was code-signed. So the assembly is allowed to load, but is not marked as being digitally signed. There is a 15 second delay for CRL retrievals. This is how long the CLR will keep on re-trying to download the CRL before it finally times out. So the delay in loading the .NET assembly occurs because Windows is unable to download the CRL and keeps trying to download it for 15 seconds before timing out.

This behavior is by design.

The .NET CLR will not indicate any error or throw any security exception when verifying a signed assembly if the CRL distribution point cannot be reached. An error here from WinVerifyTrust(), the API used by the .NET CLR to verify a code signed assembly, prevents the assembly from being marked as code signed. Note that this does not apply to assemblies loaded thru the Internet Explorer hosting interface.

You could manually download the CRL and install it on your system. But the CRL is valid only for 10-15 days, so unless your system is able to update the file after this time, you will run into the same problem again.

Microsoft recommends disabling CRL checking as a workaround by disabling this option in Internet Explorer. Use the following steps to disable the CRL checking in Internet Explorer:
Select Start»Control Panel.
Double-click Internet Options.
Select the Advanced tab.
In the Security section, uncheck the Check for publisher's certificate revocation option.
By disabling the CRL checking using the Internet Options, you are not exposing yourself to a security threat because this check is not working. The reason why this problem is showing up is because your network settings are not allowing Windows to access the CRL.

In addition, it is possible to programmatically set the CRL verification. When the Check for publisher's certificate revocation is unchecked, a setting in the registry is changed. To turn off CRL verification, set HKCU\Software\Microsoft\Windows\CurrentVersion\WinTrust\Trust Providers\Software Publishing\State  from 0x00023c00 to 0x00023e00. To turn CRL Checking on again, reset the State key to 0x00023c00

http://stackoverflow.com/questions/6042555/why-my-net-application-horribly-slow-to-start-after-the-machine-is-rebooted
http://stackoverflow.com/questions/1618596/why-are-signed-assemblies-slow-to-load/
http://www.actiprosoftware.com/support/kb/article/20001/code-signed-assemblies-can-make-application-s
