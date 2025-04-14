## Basic Setup

To build an Apache HTTP server image that functions as a reverse proxy and is ready to run on OpenShift, you can follow these steps:

### Key Steps:
1. **Create a Dockerfile** to define the Apache setup.
2. **Include your custom Apache configuration files** (e.g., `httpd.conf` or reverse proxy settings).
3. **Build the Image** and make it OpenShift-compatible.
4. **Deploy the Image** on OpenShift.

Here is a complete example:

# Apache Reverse Proxy on OpenShift

This guide explains how to build an Apache HTTP server Docker image with custom reverse proxy configurations to deploy on OpenShift.

## Steps

### 1. Create a Dockerfile
The `Dockerfile` defines the base image and the commands to set up the Apache HTTP server with your custom configuration.

```Dockerfile name=Dockerfile
# Use the official Apache HTTP server image as the base image
FROM httpd:2.4

# Set an environment variable for OpenShift compatibility
ENV HTTPD_PREFIX /usr/local/apache2

# Copy custom configuration files into the container
COPY httpd.conf /usr/local/apache2/conf/httpd.conf
COPY reverse-proxy.conf /usr/local/apache2/conf/extra/reverse-proxy.conf

# Expose port 8080 for OpenShift
EXPOSE 8080

# Override the CMD to run Apache in the foreground
CMD ["httpd", "-D", "FOREGROUND"]
```

---

### 2. Create Custom Apache Configuration Files

#### Main Apache Configuration (`httpd.conf`)
This file configures the Apache server and includes the reverse proxy configuration.

```apache name=httpd.conf
# Apache HTTP Server Configuration
ServerRoot "/usr/local/apache2"
Listen 8080

# Load required modules
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so

# Include the reverse proxy configuration
Include conf/extra/reverse-proxy.conf

# Basic settings
ServerAdmin admin@example.com
ServerName localhost
DocumentRoot "/usr/local/apache2/htdocs"
<Directory "/usr/local/apache2/htdocs">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

#### Reverse Proxy Configuration (`reverse-proxy.conf`)
This file sets up the reverse proxy to forward requests to a backend server.

```apache name=reverse-proxy.conf
# Reverse Proxy Configuration
ProxyRequests Off
ProxyPreserveHost On

<Proxy *>
    Require all granted
</Proxy>

# Reverse proxy rules
ProxyPass /backend http://backend-service:8080/
ProxyPassReverse /backend http://backend-service:8080/
```

---

### 3. Build the Docker Image
Run the following commands to build and tag the Docker image:

```bash
# Build the Docker image
docker build -t apache-reverse-proxy .

# Tag the image for OpenShift (optional)
docker tag apache-reverse-proxy quay.io/<your-repo>/apache-reverse-proxy:latest
```

---

### 4. Push the Image to a Registry
Push the image to a container registry that your OpenShift cluster can access:

```bash
docker push quay.io/<your-repo>/apache-reverse-proxy:latest
```

---

### 5. Deploy the Image on OpenShift
Use the OpenShift CLI (`oc`) to deploy the image:

```bash
# Create an application from the image
oc new-app quay.io/<your-repo>/apache-reverse-proxy:latest

# Expose the application as a route
oc expose svc/apache-reverse-proxy
```

---

### 6. Verify the Deployment
1. Make sure the pod is running:
   ```bash
   oc get pods
   ```
2. Access the application via the route created:
   ```bash
   oc get routes
   ```

---

This setup assumes that you have a backend service running at `http://backend-service:8080/` in your OpenShift project. You can update the `ProxyPass` and `ProxyPassReverse` directives in `reverse-proxy.conf` to point to your actual backend service.

## mTLS

To configure the Apache server to only accept HTTPS connections and require client certificate authentication, you'll need to update the Apache configuration files to include SSL settings and client authentication directives. Here’s how you can adjust your setup:

### Key Steps:
1. **Update the `Dockerfile` to include SSL modules and certificates**.
2. **Generate Self-Signed Certificates or Use Trusted Certificates**.
3. **Modify the Apache `httpd.conf` and `ssl.conf` to enforce HTTPS and client certificate authentication**.
4. **Deploy the updated image to OpenShift**.

### Updated Files

#### Dockerfile
```Dockerfile name=Dockerfile
# Use the official Apache HTTP server image as the base image
FROM httpd:2.4

# Set an environment variable for OpenShift compatibility
ENV HTTPD_PREFIX /usr/local/apache2

# Copy custom configuration files into the container
COPY httpd.conf /usr/local/apache2/conf/httpd.conf
COPY ssl.conf /usr/local/apache2/conf/extra/ssl.conf
COPY certs/server.crt /usr/local/apache2/conf/server.crt
COPY certs/server.key /usr/local/apache2/conf/server.key
COPY certs/ca.crt /usr/local/apache2/conf/ca.crt

# Expose port 8443 for HTTPS (recommended for OpenShift)
EXPOSE 8443

# Override the CMD to run Apache in the foreground
CMD ["httpd", "-D", "FOREGROUND"]
```

---

#### Main Apache Configuration (`httpd.conf`)
This configuration ensures the server listens on SSL (port 8443) and includes the SSL configuration.

```apache name=httpd.conf
# Apache HTTP Server Configuration
ServerRoot "/usr/local/apache2"
Listen 8443

# Load required modules
LoadModule ssl_module modules/mod_ssl.so
LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so

# Include the reverse proxy and SSL configurations
Include conf/extra/reverse-proxy.conf
Include conf/extra/ssl.conf

# Basic settings
ServerAdmin admin@example.com
ServerName localhost:8443
DocumentRoot "/usr/local/apache2/htdocs"
<Directory "/usr/local/apache2/htdocs">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
```

---

#### SSL Configuration (`ssl.conf`)
This configuration enforces HTTPS and requires client certificate authentication.

```apache name=ssl.conf
# SSL Configuration
<VirtualHost *:8443>
    ServerName localhost

    # Enable SSL
    SSLEngine on

    # Server Certificate
    SSLCertificateFile "/usr/local/apache2/conf/server.crt"
    SSLCertificateKeyFile "/usr/local/apache2/conf/server.key"

    # Client Certificate Authentication
    SSLCACertificateFile "/usr/local/apache2/conf/ca.crt"
    SSLVerifyClient require
    SSLVerifyDepth 2

    # Log files
    ErrorLog "/usr/local/apache2/logs/ssl_error.log"
    TransferLog "/usr/local/apache2/logs/ssl_access.log"

    # Reverse Proxy Configuration
    ProxyRequests Off
    ProxyPreserveHost On

    <Proxy *>
        Require all granted
    </Proxy>

    ProxyPass /backend http://backend-service:8080/
    ProxyPassReverse /backend http://backend-service:8080/
</VirtualHost>
```

---

#### Generate SSL Certificates for Testing
You can generate self-signed certificates for testing purposes using `openssl`:

```bash
# Generate a server private key
openssl genrsa -out server.key 2048

# Generate a server certificate signing request
openssl req -new -key server.key -out server.csr -subj "/CN=localhost"

# Generate a self-signed server certificate
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

# Generate a CA certificate for client authentication
openssl req -x509 -new -nodes -keyout ca.key -out ca.crt -days 365 -subj "/CN=MyCA"

# Generate a client certificate signed by the CA
openssl genrsa -out client.key 2048
openssl req -new -key client.key -out client.csr -subj "/CN=Client"
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365
```

---

### Build and Deploy on OpenShift

1. **Build the Image**:
   ```bash
   docker build -t apache-reverse-proxy-ssl .
   ```

2. **Push the Image to your Registry**:
   ```bash
   docker tag apache-reverse-proxy-ssl quay.io/<your-repo>/apache-reverse-proxy-ssl:latest
   docker push quay.io/<your-repo>/apache-reverse-proxy-ssl:latest
   ```

3. **Deploy on OpenShift**:
   ```bash
   # Create an application from the image
   oc new-app quay.io/<your-repo>/apache-reverse-proxy-ssl:latest

   # Expose the service
   oc expose svc/apache-reverse-proxy-ssl --port=8443
   ```

---

### Verification
1. **Access the HTTPS Endpoint**:
   Obtain the route from:
   ```bash
   oc get routes
   ```
   Use a browser or `curl` to access the endpoint:
   ```bash
   curl --cert client.crt --key client.key --cacert ca.crt https://<route>
   ```

2. **Verify Client Certificate Requirement**:
   Without a valid client certificate, the server should reject the connection.
