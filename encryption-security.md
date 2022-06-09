## GnuPG
### Encrypt a file with AES256 algorithm, will ask for a pass phrase:
````
gpg -c --cipher-algo AES256 './data.txt' // -> data.txt.gpg
````
### Decrypt, will ask for a pass phrase:
````
gpg './data.txt.gpg' // -> data.txt
````
