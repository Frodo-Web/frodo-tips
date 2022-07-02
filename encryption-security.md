## GnuPG
### Encrypt a file with AES256 algorithm, will ask for a pass phrase:
````
$ gpg -c --cipher-algo AES256 './data.txt' // -> data.txt.gpg
````
### Decrypt, will ask for a pass phrase:
````
$ gpg './data.txt.gpg' // -> data.txt
````
## Node
### Generate JWT Secret
````
$ node
> require('crypto').randomBytes(64).toString('hex')

// '9e3412bb6339b844bea91856ae1b3e22317e7279653d6726191203923d7c2c27c4bb5d2629ec5546f629e5483fe8697f746d4e9e5bf97044a65d91e36058d3c2'
````
