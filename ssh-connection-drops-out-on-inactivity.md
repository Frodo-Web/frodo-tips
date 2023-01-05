# OpenSSH Server connection drops out after few minutes of inactivity
## Server way:
````
$ sudo nvim /etc/ssh/sshd_config 
...
ClientAliveInterval 600
ClientAliveCountMax 10
````
- ClientAliveInterval: Sets a timeout interval in seconds (600) after which if no data has been received from the client, sshd will send a message through the encrypted channel to request a response from the client. The default is 0, indicating that these messages will not be sent to the client. This option applies to protocol version 2 only.
- ClientAliveCountMax: Sets the number of client alive messages (10) which may be sent without sshd receiving any messages back from the client. If this threshold is reached while client alive messages are being sent, sshd will disconnect the client, terminating the session.
## Client way:
### Increase SSH connection timeout using client side configuration
The following option is useful when you can not edit /etc/ssh/sshd_config file on the remote server due to permission issues. Hence, you edit your ~/.ssh/ssh_config client file on your Linux, UNIX, *BSD or macOS desktop.
````
$ nvim ~/.ssh/ssh_config
...
ServerAliveInterval 15
ServerAliveCountMax 3
````
- ServerAliveInterval 15 : Sets a timeout interval in seconds after which if no data has been received from the server, ssh will send a message through the encrypted channel to request a response from the server. For example, set a timeout to 15 seconds.
- ServerAliveCountMax 3 : Sets the number of server alive messages which may be sent without ssh command receiving any messages back from the server. If this threshold is reached while server alive messages are being sent, ssh will disconnect from the server, terminating the session. The server alive messages are sent through the encrypted channel and therefore will not be spoofable.
