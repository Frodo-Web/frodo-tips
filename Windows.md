# Windows tips by Frodo<br>
## Install OpenSSH on Windows 10 (via PowerShell)
````
# Install OpenSSH client from PowerShell
PS C:\> Add-WindowsCapability -Online -Name OpenSSH.Client*
````
### The list of commands available:
- ssh.exe
- scp.exe
- sftp.exe
- ssh-add.exe
- ssh-agent.exe
- ssh-keygen.exe
- ssh-keyscan.exe
## Fix DNS resolution in WSL2
/etc/wsl.conf:
````
[network]
generateResolvConf = false
````
/etc/resolv.conf:
````
nameserver 1.1.1.1
nameserver 8.8.8.8
sudo chattr +i /etc/resolv.conf
````
