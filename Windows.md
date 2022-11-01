# Windows tips by Frodo<br>
## Install OpenSSH on Windows 10 (via PowerShell)
````
# Install OpenSSH client from PowerShell
PS C:\> Add-WindowsCapability -Online -Name OpenSSH.Client*
````
## The list of commands available:
- ssh.exe
- scp.exe
- sftp.exe
- ssh-add.exe
- ssh-agent.exe
- ssh-keygen.exe
- ssh-keyscan.exe
