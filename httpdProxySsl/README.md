

generate a self-signed certificate for testing:
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout certs/ebr.baros.cz.key \
-out certs/ebr.baros.cz.crt \
-subj "/CN=ebr.baros.cz"


Note: host.docker.internal works for accessing the host from inside Docker on Mac, Windows, and recent Linux Docker versions. If not available, replace it with the actual IP of the host (e.g., 172.17.0.1).


Build docker image
docker build -t httpd-ssl-proxy .

Run the Container
docker run -d --name httpd-ssl-proxy-default-container \
-p 443:443 \
httpd-ssl-proxy
