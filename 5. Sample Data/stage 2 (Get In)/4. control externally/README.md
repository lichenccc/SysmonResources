# External Command & Control (C2)

This means the C2 server is outside your network. Another form of C2 exist within your network, thus Internal C2 & the notion of 'Pivot' which i will elaborate later.

## Mitre's View on C2

From https://attack.mitre.org/wiki/Command_and_Control, Mitre sees it as various protocols & levels of covertness, but there's also an interesting statement from that wiki page: 

*"The resulting breakdown should help convey the concept that detecting intrusion through command and control protocols without prior knowledge is a difficult proposition over the long term."*

In my view, that statement is true but it also mean it is limited to those methods listed there. Which leads me to consider the other aspects of C2 that we may discern even if it is some form of unknown obfuscation/evasion within channels that are being monitored.

## External vs Internal C2

I believe it is important to distinct between external from internal C2. *Why? For instance in an **air-gapped environment**, assuming the adversary managed to get a backdoor into the isolated machines, s/he still needs a channel for the compromised machine to communicate with. That by definition should be INTERNAL C2 since the controller will typically be within the targetted premises.* **It also means higher severity!**

## External -> Pivot -> Internal

![](img/c2types.png)

External C2 on the other hand refers to remote-controlling machines that have some form of Internet access. There's also another important notion known as the 'Pivot', which we can think of as a *"stepping stone"* for the adversary to reach a neighbouring machine that has no direct internet access but is allowed to communicate with the compromised machine that is connected to the Internet (right side of the diagram above). 

[In the Attack Life Cycle model](https://jym.sg), I deliberately put Internal C2 as a Stage 3 tactic. From an detection perspective, *sometimes we may totally missed External C2 or it may be a internal implant that uses side-channel*, but network sensors or HIPS may flag something suspicious between internal hosts.

## Beaconing vs Non-Beaconing

Many of these backdoors & malware tend to call-back to poll the C2 server for the next instruction. This periodic communication is sometimes known as beaconing. The frequency could be short or over a longer period depending on the offensive tools (aka *periodicity*). ***For simplistic C2 that does regular/frequent polls back to the server, it may be easily spot with eye-balls, but if it is slow & over longer periods, we will likely require some form of machine analytics to spot.*** 

The samples in this sub-folder is organised into these two general types. There could be various communication protocols used. The most covert C2s are those channels that **are not being monitored**, sometimes also known as "*side-channels*" eg. heat/power variations, lights blinking etc, see BGU's research, link below. 

***Why divide into these two classes?*** ***The latter are harder to catch as you will [see in my example](https://github.com/jymcheong/SysmonResources/tree/master/6.%20Sample%20Data/stage%202%20(Get%20In)/4.%20control%20externally/Non-Beaconing). The former, together with the data-point (Sysmon Event ID 3 + network sensors) of which process to what destination address & port, may be easier to spot, whether by human (by crafting good queries as with Threat Hunting, or machine-analytics).***

Some benign programs are known to beacon (eg. software updater processes), but they are likely to be minorities. Blue-teams may have the tendency to "white-list" (ignore) such processes, as such, a crafty adversary may inject into such processes thus we may still need to be careful if such software processes deviates from where it usually communicates to.

Rare programs (identified by their hash checksums) that beacon regularly are low-hanging fruits to catch. *Non-beaconing types that are event driven are trickier & may require network packet inspection or deeper host instrumentations for detection*. With the advent of TLS/SSL, it does not make things easier, but that reality makes [Cisco Stealthwatch interesting](https://www.cisco.com/c/en/us/products/security/stealthwatch/index.html#~stickynav=1#lightbox-cta).

I don't have samples for "side-channels" but an interesting list of air-gap circumventing research can be found at: https://cyber.bgu.ac.il//advanced-cyber/airgap
