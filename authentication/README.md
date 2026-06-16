## Useful commands in creating certificates and keys for authentication
'''
openssl genrsa -out file-name.key 2048 // create a key
openssl req -new -key file-name.key -subj "/CN=specific-name" -out file-name.csr      //create a csr(certificate signing request) using key to embed public key once certificate authority is created
openssl x509 -req -in file-name.csr -signkey file-name.key -out file-name.crt         // create a certificate signed with private key
'''