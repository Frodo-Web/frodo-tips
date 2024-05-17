# SSH commands and options
## Common commands
Generate ssh key
````
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/my_custom_key
````
## Working with sshd
### Reload ssh-server configuration
````
# kill -SIGHUP $(pgrep -f "sshd -D") 
````
## Working with keys
### Supress warning messages
````
ssh -o LogLevel=quiet ...
````
### Force password authentication
````
ssh -vvv -o PubkeyAuthentication=no -o PreferredAuthentications=password frodo@8.8.8.8
````
### Force pubkey authentication
````
ssh -vvv -i keys/docker -o PubkeyAuthentication=yes -o PreferredAuthentications=publickey frodo@8.8.8.8
````
Other methods:
````
PreferredAuthentications
             Specifies the order in which the client should try protocol 2 authentication methods.  This allows a client to prefer
             one method (e.g. keyboard-interactive) over another method (e.g. password).  The default is:

                   gssapi-with-mic,hostbased,publickey,
                   keyboard-interactive,password
````
### Force specific key algorithm
You can use HostkeyAlgorithms and PubkeyAcceptedAlgorithms options for that:
````
ssh -i build/keys/docker -o HostkeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa ...
````
### SSH pubkey auth order
If you will try to run SSH in a higher loglevel mode:
````
ssh -vvv -i keys/docker ...
````
You will see the order in which ssh client will try to offer the keys to the server:
````
debug1: Will attempt key: frodo@google.com RSA SHA256:FCaBO7VP... agent
debug1: Will attempt key: frodo@yandex.ru RSA SHA256:FSG4Y... agent
debug1: Will attempt key: frodo@facebook.com RSA SHA256:Bxrj8sG... agent
debug1: Will attempt key: frodo@vk.com RSA SHA256:npTaWh9Hd... agent
debug1: Will attempt key: frodo@proton.me ED25519 SHA256:lojIxh.... agent
debug1: Will attempt key: keys/docker RSA SHA256:8wy2O... explicit
````
These identies are from ~/.ssh/. You can list them like this:
````
ssh-add -l
````
To ignore these identities, you need to use -i option together with -o IdentitiesOnly=yes
````
ssh -v -i keys/docker -o IdentitiesOnly=yes -o PubkeyAuthentication=yes -o PreferredAuthentications=publickey frodo@8.8.8.8
````
Then you will see there is only one identity which you listed in the commandline:
````
debug1: Will attempt key: keys/docker RSA SHA256:8wy2O... explicit
````
### Ignore ~/.ssh/known/hosts (Disable SSH Host Key Checking)
To hide and ignore messages like this:
````
The authenticity of host ***** can't be established.
RSA key fingerprint is *****.
Are you sure you want to continue connecting (yes/no)?
````
Or this:
````
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:ER4HhZ2GQv3aotG5d3zUbr2Snpc/oZ0vnDm75PtaZoE.
Please contact your system administrator.
Add correct host key in /home/dev/.ssh/known_hosts to get rid of this message.
Offending ED25519 key in /home/dev/.ssh/known_hosts:223
  remove with:
  ssh-keygen -f "/home/dev/.ssh/known_hosts" -R "localhost"
Host key for localhost has changed and you have requested strict checking.
Host key verification failed.
````
Simply add StrictHostKeyChecking=no to your command line arguments and ssh config, like this:
````
ssh -v -i keys/docker -o IdentitiesOnly=yes -o PubkeyAuthentication=yes -o PreferredAuthentications=publickey -o StrictHostKeyChecking=no frodo@8.8.8.8
````

### Final command:
````
ssh -i build/keys/docker -o HostkeyAlgorithms=+ssh-rsa -o PubkeyAcceptedAlgorithms=+ssh-rsa -o IdentitiesOnly=yes -o PubkeyAuthentication=yes -o PreferredAuthentications=publickey -o StrictHostKeyChecking=no -o LogLevel=quiet frodo@8.8.8.8
````

### Host key verification failed
When you see these messages:
````
[ERROR]: Failed to ssh to [hostname_here]. No ECDSA host key is known for greenplum-segment04 and you have requested strict checking.
Host key verification failed.
````
Possible solutions. <br>
First, generate know_hosts for those hostnames
````
ssh-keyscan [hostname] >> ~/.ssh/known_hosts
````
Second, use ssh client with StrictHostKeyChecking=no option
````
ssh -o StrictHostKeyChecking=no ...
````
