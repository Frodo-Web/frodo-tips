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
## OpenSSL
Show DER, pkcs7 cert 
```
openssl pkcs7 -inform der -print_certs -in company-prod.p7b
```
Show pem cert
```
openssl x509 -in company-prod.pem -text -noout
```
Cert format reference
```
CRL

-----BEGIN X509 CRL-----
-----END X509 CRL-----

CRT

-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----

CSR

-----BEGIN CERTIFICATE REQUEST-----
-----END CERTIFICATE REQUEST-----

NEW CSR

-----BEGIN NEW CERTIFICATE REQUEST-----
-----END NEW CERTIFICATE REQUEST-----

PEM

-----BEGIN RSA PRIVATE KEY-----
-----END RSA PRIVATE KEY-----

PKCS7

-----BEGIN PKCS7-----
-----END PKCS7-----

PRIVATE KEY

-----BEGIN PRIVATE KEY-----
-----END PRIVATE KEY-----
```
