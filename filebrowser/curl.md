## Documenting the api of File Browser to send/receive data via curl.
### Get your auth token
curl --location 'http://localhost:8080/api/login' \
--header 'Content-Type: application/json' \
--data '{"username":"username", "password":"password"}'

### Download a file 
curl --location 'http://localhost/api/raw/download.png' \
--header 'X-Auth: token_here'

### Upload a file
curl --location 'http://localhost/api/resources/sample.png' \
--header 'X-Auth: token_here' \
--header 'Content-Type: image/png' \
--data '@sample.png'
