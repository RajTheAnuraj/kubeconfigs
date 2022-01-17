Install certbot

run 
 certbot certonly --manual --preferred-challenges dns

 This will ask you to enter a TXT record under your dns.
 Do that and it will create the certs for you

 The certs are only valid for 3 months


 Use open ssl to convert the output to a pfx
 Linux comes with openssl
 Windows is a PITA coz we need to compile the openssl code from github. Prebuild images are not available and the ones available might not be trusted

 ```
openssl pkcs12 -inkey privkey1.pem -in fullchain1.pem -certfile cert1.pem -export -out anurajajithkumar.pfx
 ```