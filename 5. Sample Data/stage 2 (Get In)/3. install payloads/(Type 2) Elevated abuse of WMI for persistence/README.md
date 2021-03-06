# Abusing WMI for Persistence

## Background

Windows Management Instrumentation (WMI) can be used to install event filters, providers, consumers, and bindings that execute code when a defined event occurs. Adversaries may use the capabilities of WMI to subscribe to an event and execute arbitrary code when that event occurs, providing persistence on a system. Adversaries may attempt to evade detection of this technique by compiling WMI scripts. https://attack.mitre.org/wiki/Technique/T1084. The reference section on the ATT&CK page has several good papers for those who want to dig deeper. 

## Empire

![](img/empire.png)

**Take note of the SubName field "Updater", we will need it later during Observations.** This module was used to install persistence. There is no direct file or registry key written but again one will notice some registry events later. 

It also interesting to contrast the relative ease of installing payload/persistence with a privilege Empire session, but yet it may not be so straight forward to detect such persistence if we don't know what to look for. There are other WMI related modules:

![](img/wmimodules.png)

The asterisk (*) means it requires admin rights or privilege session to run. 

## Observations

![](img/wmievents.png)

* All 3 WMI events (id 19, 20 & 21) all have *"Updater"* as value in the *Name* field. This value is the same as the Empire module. 

* From the other fields, there is no direct link to the Empire Powershell instance. The only clue we have is the entire sequence happened on the same second 10:50:06. 

* The registry events has nothing to do with the WMI implant but related to powershell networking.

* Even the Process Create event **has no direct link with the offensive Powershell process**:

  ![](img/processcreate.png)

  WMI Provider Host was invoked when implanting this WMI subscription. 

* The general sequence of WMI subscription: [setup Filter & Consumer, follow by linking Consumer](https://learn-powershell.net/2013/08/14/powershell-and-events-permanent-wmi-event-subscriptions/) to Filter. The *WMI consumer Command Line* can be seen below, has a the Empire stager encoded in the Base64 block:

  ![](img/consumer.png)

* WMI Event ID 21 that binds the Filter to Consumer looks like:

  ![](img/wmibind.png)

* The last two events are related to Powershell network connection back to the Empire listener (C2 server). *I used a stupidly obvious host name: empirec2 for illustration reason*. In the real world, it is trival to use web-servers (eg. Nginx) to front it with [valid but free SSL cert](https://letsencrypt.org) & domain name. 

  ![](img/empirec2.png)

  ​

## Questions

* Are these WMI events (id 19, 20 & 21) commonly seen in a your environment? More at server zones or both server & client zones?
* Apart from time of sequence, is there any other way to reliably to know which process created the WMI subscription?

I asked the 2nd question because after turning on the WMI trace, I still can't seem to find the answer. For C2 that are beaconing, we may still be able to see the preceding network activities (eg. Powershell in this case), ***but what if it is non-beaconing type of C2 with delayed or event-driven execution?*** One of those non-beaconing C2 was a PoC that I experimented with [Outlook VSTO](https://github.com/jymcheong/SysmonResources/tree/master/6.%20Sample%20Data/stage%202%20(Get%20In)/3.%20install%20payloads/(Type%201)%20Abuse%20Outlook%20VSTO). 

## Other References

https://www.fireeye.com/blog/threat-research/2016/08/wmi_vs_wmi_monitor.html before Sysmon had event ID 19 to 21, Mr [Matt Graeber](https://twitter.com/mattifestation?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor) is one of those folks who actively looked into WMI, both offensively and defensively.

[https://msdn.microsoft.com/en-us/library/aa826686(v=vs.85).aspx](https://msdn.microsoft.com/en-us/library/aa826686(v=vs.85).aspx)

https://nathanguagenti.blogspot.sg/2017/03/windows-event-forwarding-etl-etw.html

https://www.darkoperator.com/blog/2017/10/14/basics-of-tracking-wmi-activity