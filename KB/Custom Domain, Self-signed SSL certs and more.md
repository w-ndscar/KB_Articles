
Create a Certificate Authority - rootCA key and cert
Setup a local Domain
For a self signed certificate, First Create a cert key, certificate signing request (csr)
Then using the csr, CA's rootCA key and cert.pem, create a certificate (crt) and use it in a webapp/proxy
Then add/import the CA's cert to the Trusted Certificates root so it can be trusted on that machine.