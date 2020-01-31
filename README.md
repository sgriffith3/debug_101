# Debugging 101
Effective Debugging to Save Your Company Time and Money 

### Step 1: Check the Logs

In order to see what is really happening inside of your system, the first thing that you will need to do is check the system's logs. Whether you are in charge of thousands of servers or just simply want to troubleshoot what is going wrong with your personal laptop, if the failing program's output doesn't tell you what is wrong, check the logs.

In order to check the logs for your linux system, you will need to have sudo privileges. Once you are sure that you have these privileges, you can perform the following command from the CLI to view your system's logs:

`sudo cat /var/logs/syslog`

Once you perform this command, you will see an incredibly large amount of information that your system has been tracking for you, potentially without your knowledge. It will look something like this:
    
    Jan 31 09:29:12 sumi-01 systemd-udevd[10945]: Could not generate persistent MAC address for v-200-1810: No such file or directory
    Jan 31 09:29:12 sumi-01 systemd-networkd[908]: v-200-1810: Gained carrier
    Jan 31 09:29:12 sumi-01 networkd-dispatcher[1108]: WARNING:Unknown index 2725058 seen, reloading interface list
    Jan 31 09:29:12 sumi-01 kernel: [5249924.319880] br0: port 8(v-200-1810) entered blocking state
    Jan 31 09:29:12 sumi-01 kernel: [5249924.319884] br0: port 8(v-200-1810) entered disabled state
    Jan 31 09:29:12 sumi-01 kernel: [5249924.320023] device v-200-1810 entered promiscuous mode
    Jan 31 09:29:12 sumi-01 kernel: [5249924.320068] br0: port 8(v-200-1810) entered blocking state
    Jan 31 09:29:12 sumi-01 kernel: [5249924.320071] br0: port 8(v-200-1810) entered forwarding state
    Jan 31 09:29:12 sumi-01 systemd-networkd[908]: v-200-1810: Lost carrier
    Jan 31 09:29:12 sumi-01 kernel: [5249924.333276] br0: port 8(v-200-1810) entered disabled state
    Jan 31 09:29:12 sumi-01 kernel: [5249924.333719] device v-200-1810 left promiscuous mode
    Jan 31 09:29:12 sumi-01 kernel: [5249924.333727] br0: port 8(v-200-1810) entered disabled state
    Jan 31 09:29:12 sumi-01 systemd[1]: ansible-200-bchd.service: Control process exited, code=exited status=1
    Jan 31 09:29:12 sumi-01 systemd[1]: ansible-200-bchd.service: Failed with result 'exit-code'.
    Jan 31 09:29:12 sumi-01 systemd[1]: Failed to start ansible-200-bchd virual machine.
    Jan 31 09:29:14 sumi-01 systemd[1]: Stopped ansible-200-bchd virual machine.
    Jan 31 09:29:19 sumi-01 systemd[1]: Reloading.
    Jan 31 09:29:19 sumi-01 systemd[1]: Starting Daily apt download activities...
    Jan 31 09:29:20 sumi-01 systemd-resolved[983]: Server returned error NXDOMAIN, mitigating potential DNS violation DVE-2018-0001, retrying transaction with reduced feature level UDP.
    Jan 31 09:29:20 sumi-01 systemd-resolved[983]: Server returned error NXDOMAIN, mitigating potential DNS violation DVE-2018-0001, retrying transaction with reduced feature level UDP.
    Jan 31 09:29:29 sumi-01 systemd[1]: Started Daily apt download activities.

If you are new to using the system logs to troubleshoot, this is a lot of information to take it all at once. So let's break it down to a reasonable level by just looking at one of the specific lines of the log file.

`Jan 31 09:29:12 sumi-01 systemd-udevd[10945]: Could not generate persistent MAC address for v-200-1810: No such file or directory`

- `Jan 31 09:29:12` 
    - -------------> The time and date when this information was gathered into the log.
- `sumi-01`
    - -------------> The hostname of the system that you are on.
- `systemd-udevd[10945]:`
    - -------------> The specific software process that is reporting this information to the system log.
- `Could not generate persistent MAC address for v-200-1810: No such file or directory`
    - -------------> The specific message that is being reported.

**Note: If you see a message that appears to be relevant to your issue inside of the logs at this point, now is a great time to copy and paste the _message_ (and _software process_) to your favorite search engine. _SOMEBODY_ had to make the code report logs to the system when your specific error occurs, which means that _SOMEBODY ELSE_ has run into this issue already, and is probably very willing to help out.**

Assuming that you now know enough about how to read your logs, and how to search for more information on their specific outputs, let's dive a bit deeper into our logs and see how to only view the ones you care about.

The easiest way to do this is to use the **grep** command. This command allows you to search within a specific document (or more than one) for a specified pattern. Since we seem to be having an issue with something involving `v-200-1810`, we can use the same **cat** command, combined with **grep** to view only those errors.

    `sudo cat /var/log/syslog | grep v-200-1810`
    
Looking just at the logs that are relevant to your issue can be incredibly telling. For example, the end of my logs look like this:

Jan 31 09:29:06 sumi-01 systemd-udevd[10902]: Could not generate persistent MAC address for v-200-1810: No such file or directory
Jan 31 09:29:06 sumi-01 kernel: [5249919.068306] br0: port 8(v-200-1810) entered blocking state
Jan 31 09:29:06 sumi-01 systemd-networkd[908]: v-200-1810: Gained carrier
Jan 31 09:29:06 sumi-01 kernel: [5249919.068310] br0: port 8(v-200-1810) entered disabled state
Jan 31 09:29:06 sumi-01 kernel: [5249919.068460] device v-200-1810 entered promiscuous mode
Jan 31 09:29:06 sumi-01 kernel: [5249919.068529] br0: port 8(v-200-1810) entered blocking state
Jan 31 09:29:06 sumi-01 kernel: [5249919.068533] br0: port 8(v-200-1810) entered forwarding state
Jan 31 09:29:06 sumi-01 systemd-networkd[908]: v-200-1810: Lost carrier
Jan 31 09:29:06 sumi-01 kernel: [5249919.083081] br0: port 8(v-200-1810) entered disabled state
Jan 31 09:29:06 sumi-01 kernel: [5249919.083664] device v-200-1810 left promiscuous mode
Jan 31 09:29:06 sumi-01 kernel: [5249919.083671] br0: port 8(v-200-1810) entered disabled state
Jan 31 09:29:12 sumi-01 systemd-udevd[10945]: Could not generate persistent MAC address for v-200-1810: No such file or directory
Jan 31 09:29:12 sumi-01 systemd-networkd[908]: v-200-1810: Gained carrier
Jan 31 09:29:12 sumi-01 kernel: [5249924.319880] br0: port 8(v-200-1810) entered blocking state
Jan 31 09:29:12 sumi-01 kernel: [5249924.319884] br0: port 8(v-200-1810) entered disabled state
Jan 31 09:29:12 sumi-01 kernel: [5249924.320023] device v-200-1810 entered promiscuous mode
Jan 31 09:29:12 sumi-01 kernel: [5249924.320068] br0: port 8(v-200-1810) entered blocking state
Jan 31 09:29:12 sumi-01 kernel: [5249924.320071] br0: port 8(v-200-1810) entered forwarding state
Jan 31 09:29:12 sumi-01 systemd-networkd[908]: v-200-1810: Lost carrier
Jan 31 09:29:12 sumi-01 kernel: [5249924.333276] br0: port 8(v-200-1810) entered disabled state
Jan 31 09:29:12 sumi-01 kernel: [5249924.333719] device v-200-1810 left promiscuous mode
Jan 31 09:29:12 sumi-01 kernel: [5249924.333727] br0: port 8(v-200-1810) entered disabled state

Although I am just sharing this much output, I promise you there is a lot more that occured before these. This reveals to me that this issue has been occuring regularly, and that means I had better look into fixing this process, or at least stopping it to free up my system resources.
    
### STEP 2: Active Debugging

`sudo tail -f /var/log/syslog`
