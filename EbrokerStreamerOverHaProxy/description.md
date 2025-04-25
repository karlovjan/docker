
>https://hub.docker.com/_/haproxy/

https://www.haproxy.com/documentation/haproxy-configuration-manual/latest/


docker compose up --detach
docker compose up -d

docker compose down

**Explanation and Important Considerations:**

1.  **Docker Compose File:**
    * `version: '3.8'`: Specifies the Docker Compose file format version.
    * `services`: Defines the services (containers) in the application.
    * `haproxy`:
        * Uses the official `haproxy:latest` image.
        * Maps port 80 of the host to port 80 of the container. This is where HAProxy will listen for incoming HTTP requests.
        * Mounts the `haproxy.cfg` file from the host into the container. This file contains the HAProxy configuration.
        * `depends_on`: Ensures that `tomcat1` and `tomcat2` are started before HAProxy.
        * `networks`: Places the container in the `app-network` network.
    * `tomcat1` and `tomcat2`:
        * Uses the `tomcat:9.0-jdk11-slim` image.
        * Maps port 8080 and 8081 of the host to port 8080 of the containers (for debugging). Remove these ports in production.
        * Mounts the `webapp1.war` and `webapp2.war` files from the host into the Tomcat `webapps` directory. Replace `./webapp1` and `./webapp2` with the actual paths to your WAR files.
        * `networks`: Places the containers in the `app-network` network.
    * `networks`: Defines the `app-network` bridge network, allowing the containers to communicate with each other.

2.  **`haproxy.cfg`:**
    * `global`: Global HAProxy settings.
    * `defaults`: Default settings for the frontend and backend.
    * `frontend http-in`: Defines the frontend that listens for incoming HTTP requests.
        * `bind *:80`: Binds to all interfaces on port 80.
        * `default_backend app-servers`: Specifies the default backend to use.
    * `backend app-servers`: Defines the backend servers.
        * `balance roundrobin`: Uses the round-robin load balancing algorithm.
        * `server tomcat1 tomcat1:8080 check`: Adds `tomcat1` as a backend server. The `check` option enables health checks.
        * `server tomcat2 tomcat2:8080 check`: Adds `tomcat2` as a backend server.

3.  **WAR Files:**
    * You must have your `webapp1.war` and `webapp2.war` files in the same directory as the `docker-compose.yml` file, or specify the correct paths.
    * Replace `./webapp1` and `./webapp2` with the actual file paths.

4.  **Running the Application:**
    * Save the `docker-compose.yml` and `haproxy.cfg` files.
    * Open a terminal and navigate to the directory where you saved the files.
    * Run `docker-compose up -d` to start the containers in detached mode.
    * Your web applications should be accessible through HAProxy on port 80 of your host.

5.  **Health Checks:**
    * The `check` option in the HAProxy backend configuration enables health checks. HAProxy will periodically check the health of the Tomcat servers and only send traffic to healthy servers.
    * If your web applications have specific health check endpoints, you can configure HAProxy to use them.

6.  **Production Considerations:**
    * Remove the port mappings for `tomcat1` and `tomcat2` in production.
    * Use a more robust health check configuration.
    * Consider using a persistent volume for the HAProxy configuration.
    * Implement proper logging and monitoring.
    * Adjust timeouts according to the application needs.
    * For SSL/TLS, configure haproxy to handle the SSL termination.



Logging
global
log stdout format raw local0 info

default 
log global

frontend
log stdout format raw local0 info

backend
log stdout format raw local0 info


modify TokenAuthPlugin
ebr-web.xml
add
<property name="cookiesSecure" value="false" />
to tokenAuthPlugin


Streamer
Connection refused (Connection refused): localhost:10080
modify
streamer-distro\config\streamer.properties
session.authenticationUrl=http://tomcat1:8080/...

SSL/TSL
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

cat client.crt clientKey.pem > ebr.fio.cz.pem



Errors
Nevykresluji se grafy
java.lang.UnsatisfiedLinkError: /usr/local/openjdk-11/lib/libfontmanager.so: libfreetype.so.6: cannot open shared object file: No such file or directory
Caused by: java.lang.NoClassDefFoundError: Could not initialize class sun.font.SunFontManager

https://stackoverflow.com/questions/54720878/where-is-libfreetype-so-6-in-openjdk-i-got-an-unsatisfiedlinkerror-when-using-a

