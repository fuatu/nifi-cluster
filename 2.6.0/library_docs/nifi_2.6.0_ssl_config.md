# Apache NiFi 2.6.0 SSL Configuration

Based on the official documentation and search results, here are the key SSL configuration properties for NiFi 2.6.0:

## Environment Variables for SSL Configuration

NiFi 2.6.0 supports the following environment variables for SSL configuration:

### Keystore Configuration
- `KEYSTORE_PATH`: Path to the keystore file (e.g., `/opt/certs/nifi0/keystore.jks`)
- `KEYSTORE_TYPE`: Type of keystore (JKS, PKCS12, BCFKS) - **Default: JKS**
- `KEYSTORE_PASSWORD`: Password for the keystore
- `KEY_PASSWORD`: Password for the key (optional, defaults to keystore password)

### Truststore Configuration
- `TRUSTSTORE_PATH`: Path to the truststore file (e.g., `/opt/certs/nifi0/truststore.jks`)
- `TRUSTSTORE_TYPE`: Type of truststore (JKS, PKCS12, BCFKS) - **Default: JKS**
- `TRUSTSTORE_PASSWORD`: Password for the truststore

### SSL Protocol
- `SSL_PROTOCOL`: SSL/TLS protocol version (TLS, TLSv1.2, TLSv1.3, etc.)

## nifi.properties SSL Configuration

The generated nifi.properties file contains these SSL-related properties:

```properties
# Web HTTPS Configuration
nifi.web.https.host=nifi0
nifi.web.https.port=9443

# Security Configuration
nifi.security.keystore=./conf/keystore.jks
nifi.security.keystoreType=jks
nifi.security.keystorePasswd=supersecret1
nifi.security.truststore=./conf/truststore.jks
nifi.security.truststoreType=jks
nifi.security.truststorePasswd=supersecret1
```

## Key Points

1. The TLS Toolkit generates certificates with relative paths (`./conf/`) in nifi.properties
2. Environment variables can override these properties
3. The HTTPS port in nifi.properties (9443) must match the Docker configuration
4. Certificate hostnames must match the actual hostnames used by NiFi nodes

## Common Issues

1. **Path Mismatch**: Environment variables pointing to wrong certificate paths
2. **Port Mismatch**: nifi.properties HTTPS port not matching Docker configuration
3. **Hostname Mismatch**: Certificates generated for wrong hostnames
4. **Certificate Location**: Certificates not accessible at expected paths

## References

- Apache NiFi Administration Guide
- Apache NiFi Toolkit Guide
- Apache NiFi Walkthroughs