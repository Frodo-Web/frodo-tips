# SSH commands
## Working with keys
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
### SSH pubkey auth order
If you will try to run SSH in a higher loglevel mode:
````
ssh -vvv ...
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
