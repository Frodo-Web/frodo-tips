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
## WSL2 + Windows Terminal + OhMyZsh + Powerlevel10k
Install fonts
````
https://github.com/romkatv/powerlevel10k#fonts
````
Windows Terminal: <br>
Choose the font MesloGS NF in Profile > Defaults > Appearance. <br>
Use Ubuntu as default startup profile <br>
Install zsh:
````
sudo apt-get update && sudo apt-get install -y zsh
````
Install OhMyZsh:
````
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
````
Install Powerlevel10k:
````
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
````
Add this line to .zshrc:
````
ZSH_THEME="powerlevel10k/powerlevel10k"
````
Run zsh, follow instructions:
````
zsh
````
![image](https://user-images.githubusercontent.com/102017064/235617354-d0efbefe-c7d6-4790-820d-ed23aa29c31b.png)
Additionally, explore color schemes:
````
https://terminalsplash.com/
````
## Windows 11 disk performance test (Built-in utility)
````
winsat disk -drive c
````
By default it doesn't test Random Write speed though, so you could check random 16.0 write with:
````
winsat disk -write -ran -drive c
````
