# Goblin
[![](https://img.shields.io/badge/Category-Defense%20Evasion-E5A505?style=flat-square)]() [![](https://img.shields.io/badge/Language-C%20%2f%20C++%20%2f%20Python3-E5A505?style=flat-square)]()

## Summary
`Goblin` is a module to enumerate all the threads of the EventLog Service Module(`wevtsvc.dll`) and kill them in an effort to disable EventLog service from registering any new events. Disabling Windows Event Logging and Sysmon logging paves the way for operators to perform Post-Exploitation activities in a safe and stealthy manner.

![wevtsvc.dll Threads](Screenshots/evtlog-threads.png "wevtsvc.dll Threads")

Additionally, it also allows us to "revive" the EventLog service again after we are done with Post-Ex activities.

## Usage
### Using the pre-built binary
Grab it from the `Releases` section.
### Compiling yourself
Make sure you have a working VC++ 2019 dev environment set up beforehand and `Git` installed. Execute the following from a x64 Developer Command Prompt.
```
1. git clone https://github.com/slaeryan/AQUARMOURY
2. cd AQUARMOURY/Goblin
3. compile64.bat
```

`goblin_x64.dll` is the name of the module. It is converted to a PIC blob(shellcode) with the help of [sRDI](https://github.com/monoxgas/sRDI) courtesy of [@monoxgas](https://twitter.com/monoxgas?lang=en) and delivered straight to memory via your favourite C2 framework for inline execution/local execution in the implant process.

When `Goblin` module is executed on a host, it primarily has one of these two objectives to accomplish:
1. Kill `wevtsvc.dll` threads if they are running
2. If `wevtsvc.dll` threads are not running, "revive" the EventLog service

Ergo, run the module once to disable event logging and once again if you wish to re-enable event logging without requiring a reboot.

In case your C2 framework does not support inline execution of shellcode for stability issues, you can also use the `Fork&Run` feature but the former is preferred over the latter due to OPSEC concerns.

Keep in mind that this capability is meant to be run from an **Elevated** context and will not work if the process token does not have `SeDebugPrivilege`.

## OPSEC concerns
This tool is inspired from the great [Invoke-Phant0m](https://github.com/hlldz/Invoke-Phant0m) by [Halil Dalabasmaz](https://twitter.com/hlldz).

So why not use PowerShell?

Because, using PowerShell might not be the most OPSEC-safe way of doing this in 2020. There are numerous telling events generated by just running `Invoke-Phant0m` on a host from Enhanced PowerShell logging to Process Access Event(Sysmon Event ID 10).

Refer to [1](https://twitter.com/inzlain/status/867172350457925632/photo/1) and [2](https://malwarenailed.blogspot.com/2017/10/update-to-hunting-mimikatz-using-sysmon.html) for additional information on how to detect `Invoke-Phant0m`.

Enter Goblin.

![Goblin Overview](Screenshots/overview.png "Goblin Overview")

The first `notepad.exe` was started before running `Goblin` module on host and hence reported. The second one was launched after killing the EventLog service module threads and as expected the process creation event(Sysmon Event ID 1) never showed up in Sysmon logs. Note the time difference underlined in red, operators were successfuly able to conduct Post-Ex activities during this time without any of it showing up in Event Logs or being forwarded to SOC/SIEM.

After conducting Post-Exploitation we decided to enable logging again so as not to raise questions as to why a host has stopped sending events altogether to SOC.

So we run the tool again to "revive" the EventLog service. What this does under the hood is kill the process hosting the service(`svchost.exe`) and start the service again.
As is evident from the screenshot, this generates **two** potentially telling events which might give us away.

Key Takeaways:
1) **Killing the threads** do not report any kind of additional event itself so it should be **OPSEC-safe** compared to **restarting the EventLog service** which is **noisy** and will report additional events providing Blue-teams an oppurtunity to detect us.
2) While the `wevtsvc` threads are killed, the host will **not be reporting/forwarding any events** to SOC/SIEM which may or may not be noticed and alerted, ergo exercise caution.
3) This will not stop PSPs relying on ETW Tracing to detect malicious activity. For that we need to hook `NtTraceEvent` syscall from Kernel-mode(Ring-0).

For a more elegant solution that allows filtering of events reported, see [@bats3c](https://twitter.com/_batsec_) work on [EvtMute](https://github.com/bats3c/EvtMute).

## Detection
![CAPA Scan](Screenshots/capa.png "CAPA Scan")
Here is a mandatory [CAPA](https://github.com/fireeye/capa) scan result on the `Goblin` DLL.

![Detection](Screenshots/detection.png "Detection")
And here is an additional event reported as a result of "reviving" the EventLog service(System Event ID 7031)

Note that by killing the EventLog service threads, **NO** additional events show up in the Event Logs whatsoever. Detection from event logs is possible iff operator has restarted the service.

## Credits
1. This tool was inspired by [@spotheplanet](https://twitter.com/spotheplanet) lab on [Disabling Windows Event Logs by Suspending EventLog Service Threads](https://www.ired.team/offensive-security/defense-evasion/disabling-windows-event-logs-by-suspending-eventlog-service-threads). Although, suspending/resuming threads do not work in practice because all the events are going to be written to the EventLog once the threads are resumed, it is an excellent post that explains in great detail the process of finding `wevtsvc.dll` threads. The code and algorithm is hacked from the post and I'd highly recommend giving it a read.
2. [https://artofpwn.com/phant0m-killing-windows-event-log.html](https://artofpwn.com/2017/06/05/phant0m-killing-windows-event-log.html)
3. [@dtm](https://twitter.com/0x00dtm) for first bringing this to my attention while discussing ways to evade Sysmon.
4. As usual, [@reenz0h](https://twitter.com/Sektor7Net) and [RTO: MalDev course](https://institute.sektor7.net/red-team-operator-malware-development-essentials) for the templates that I keep using to this date.

## Author
Upayan ([@slaeryan](https://twitter.com/slaeryan)) [[slaeryan.github.io](https://slaeryan.github.io)]

## License
All the code included in this project is licensed under the terms of the GNU GPLv2 license.

#

[![](https://img.shields.io/badge/slaeryan.github.io-E5A505?style=flat-square)](https://slaeryan.github.io) [![](https://img.shields.io/badge/twitter-@slaeryan-00aced?style=flat-square&logo=twitter&logoColor=white)](https://twitter.com/slaeryan) [![](https://img.shields.io/badge/linkedin-@UpayanSaha-0084b4?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/upayan-saha-404881192/)