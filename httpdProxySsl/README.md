The Apache HTTP Serve
https://hub.docker.com/_/httpd

https://httpd.apache.org/docs/2.4/ssl/ssl_faq.html
TSL
https://www.haproxy.com/documentation/haproxy-configuration-tutorials/security/authentication/client-certificate-authentication/#create-a-certificate-authority
https://www.haproxy.com/documentation/haproxy-configuration-tutorials/security/ssl-tls/client-side-encryption/
https://www.baeldung.com/openssl-self-signed-cert
https://www.ibm.com/docs/en/api-connect/10.0.x_cd?topic=profile-using-openssl-generate-format-certificates

openssl req -newkey rsa:2048 -nodes -keyout private_key.pem -x509 -days 365 -out public_certificate.pem

openssl req \
-newkey rsa:2048 \
-nodes \
-x509 \
-days 3650 \
-keyout privateCA.pem \
-out ca.crt

the self-signed certificate ca.crt

A client certificate is a type of digital certificate used to verify the identity of a user or device trying to access server resources.

openssl req \
-newkey rsa:2048 \
-nodes \
-days 3650 \
-subj "/CN=exampleUser/O=exampleOrganization" \
-keyout clientKey.pem \
-out client.csr

Create a certificate extensions file named client-cert-extensions.cnf

basicConstraints = CA:FALSE
keyUsage = digitalSignature
extendedKeyUsage = clientAuth
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer

Use the following command to sign the certificate signing request. This uses our previously generated CSR client.csr, CA certificate ca.crt, and CA key privateCA.pem to create a client certificate, client.crt.

openssl x509 \
-req \
-in client.csr \
-out client.crt \
-CA ca.crt \
-CAkey privateCA.pem \
-CAcreateserial \
-days 3650 \
-extfile client-cert-extensions.cnf


In httpd.conf
Include conf/extra/httpd-ssl.conf

In httpd-ssl.conf
remove <virtual page section

Warning!
Modify Streamer configuration
session.authenticationUrl
