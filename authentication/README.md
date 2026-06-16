## Useful commands in creating certificates and keys for authentication

### Generating CA(certificate-authority) certificate

```
openssl genrsa -out file-name.key 2048 // create a key
openssl req -new -key file-name.key -subj "/CN=specific-name" -out file-name.csr      //create a csr(certificate signing request) using key to embed public key once certificate authority is created
openssl x509 -req -in file-name.csr -signkey file-name.key -out file-name.crt         // create a certificate signed with private key
```

### Generating admin user certificate using CA

```
openssl genrsa -out admin-user.key 2048                                                             // create admin key
openssl req -new -in admin-user.key -subj "/CN=kube-admin/O=system:masters" -out admin-user.csr     // create admin csr with subject containing common-name(CN) and group-member(O) as a key-value pair
openssl x509 -req -in admin-user.csr -CA file-name.crt -CAkey file-name.key -out admin-user.crt     // create admin certificate signed with private key of file-name.key and extracting subject name from file-name.crt to insert as issuer in admin-user.crt
```