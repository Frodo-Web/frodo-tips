# Configure SSH Key-Based Authentication & Disable Password-Based Authentication
## Client side
### Generate keys
````
$ ssh-keygen -f my-keys/freebsd-machine
````
### Copy generated keys to your server
````
$ ssh-copy-id -i my-keys/freebsd-machine.pub username@122.133.xxx.xxx
OR
$ scp my-keys/freebsd-machine.pub username@122.133.xxx.xxx:my-keys/freebsd-machine.pub
````
### Edit SSH config
````
$ cat .ssh/config
...
Host freebsd-machine
        Hostname 122.133.xxx.xxx
        User username
        Port 2222
        IdentityFile ~/my-keys/freebsd-machine
````
## Server side
### Add your public key to authorized keys
````
$ ssh username@122.133.xxx.xxx
$ mkdir .ssh
$ cat my-keys/freebsd-machine.pub >> .ssh/authorized_keys
````
You can add your own authorized keys file in /etc/ssh/sshd_config at "AuthorizedKeysFile" line
### Change default port & Disable Password-Based Authentication
````
$ sudo nvim /etc/ssh/sshd_config
...
Port 2222
PasswordAuthentication no
ChallengeResponseAuthentication no
````
### Restart SSH daemon
````
$ sudo service sshd restart
// Logout Ctrl + D
````
## Make sure Password-Based authentication is disabled
Try it out on both user and root
````
$ ssh root@122.133.xxx.xxx -p 2222
>> root@122.133.xxx.xxx: Permission denied (publickey).
$ ssh username@122.133.xxx.xxx -p 2222
>> username@122.133.xxx.xxx: Permission denied (publickey).
````
## Now you can log-in by simply typing your Host from .ssh/config
````
$ ssh freebsd-machine
````
