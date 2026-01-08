https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/chrome-remote-debugger-pentesting/

Chrome Remote Debugger is a tool that debugs web applications.

## [Investigation](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/chrome-remote-debugger-pentesting/#investigation)

```bash
/usr/bin/google-chrome --remote-debugging-port=12345
```

If the target system is running **Google Chrome Debugger** with specific port, we can port forward and may be able to retrieve sensitive data in browser debugging mode.

  

## [Exploitation](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/chrome-remote-debugger-pentesting/#exploitation)

### [1. Port Forwarding](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/chrome-remote-debugger-pentesting/#1.-port-forwarding)

First off, start port forwarding in local machine.

```bash
ssh -L 12345:127.0.0.1:12345 remote-user@example.com
```

### [2. Configure Network Targets in Chrome](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/chrome-remote-debugger-pentesting/#2.-configure-network-targets-in-chrome)

Assume the chrome debugger is running on port **12345**.  
Open Chrome browser and input the following string in URL bar at the top of the window.

```bash
chrome://inspect/#devices
```

Then click **“Configure…”** at the right of **“Discover network targets”**. The modal window opens.  
In the modal window, enter **“localhost:12345”** then click **“Done”**.  
Now we should see the remote host appears at the bottom of the **“Remote Target”**.  
Click **“inspect”** then new browser open. We can browse the website.

### [(Option) Find Credentials](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/chrome-remote-debugger-pentesting/#(option)-find-credentials)

If the login page found when inspecting, we may see a credential in the developer tool at the right pane. Go to `Network` and click the target page such as `login.php` then go to the `Payload` tab. We can find credentials.