Install certbot

run 
 certbot certonly --manual --preferred-challenges dns

 This will ask you to enter a TXT record under your dns.
 Do that and it will create the certs for you

 The certs are only valid for 3 months
 