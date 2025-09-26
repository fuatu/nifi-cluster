# Apache NiFi 2.6.0 Cluster Setup

This directory contains a complete Docker Compose setup for running a secure, high-availability Apache NiFi 2.6.0 cluster with automatic certificate generation and load balancing.

## Architecture Overview

The cluster consists of:
- **3 NiFi nodes** (nifi0, nifi1, nifi2) running Apache NiFi 2.6.0 with Java 21
- **1 ZooKeeper instance** for cluster coordination
- **1 NiFi Toolkit container** for automatic TLS certificate generation
- **1 Nginx load balancer** for distributing traffic across nodes

## Key Features

- ✅ **Automatic TLS Certificate Generation**: Uses NiFi Toolkit to generate all required certificates
- ✅ **Secure Cluster Communication**: All inter-node communication is encrypted
- ✅ **Load Balancing**: Nginx provides session-aware load balancing
- ✅ **Persistent Storage**: All NiFi data persists across container restarts
- ✅ **Health Checks**: Built-in health monitoring for all services
- ✅ **Java 21 Support**: Fully compatible with NiFi 2.6.0's Java 21 requirement <mcreference link="https://medium.com/@eray.ergulec/high-availability-ha-https-installation-guide-for-apache-nifi-2-5-0-on-a-development-environment-8c91cefc1291" index="4">4</mcreference>

## Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+
- At least 6GB of available RAM
- Ports 8443, 9443 available on the host

## Quick Start

1. **Clone or navigate to this directory**:
   ```bash
   cd ../nifi-local-cluster/2.6.0
   ```

2. **Start the cluster**:
   ```bash
   docker-compose up -d
   ```

3. **Monitor startup progress**:
   ```bash
   docker-compose logs -f
   ```

4. **Access NiFi UI**:
   
   **Important**: First add hostname resolution for SSL certificates:
   ```bash
   # Required for SSL certificate validation
   echo "127.0.0.1 nifi0 nifi1 nifi2" | sudo tee -a /etc/hosts
   ```
   
   Then access the cluster:
   - **Load Balanced URL**: https://localhost:8443/nifi/ (through Nginx)
   - **Direct Node Access**:
     - Node 0: https://nifi0:8443/nifi/
     - Node 1: https://nifi1:8443/nifi/
   - **HTTP Redirect**: http://localhost:80/nifi/ (redirects to HTTPS)

5. **Login Credentials**:
   - Username: `admin`
   - Password: `supersecret1`

## Certificate Management

### Automatic Certificate Generation

The `nifi-toolkit` service automatically generates all required certificates using the Apache NiFi Toolkit:
- **Root CA certificate** for the cluster (CN=localhost, OU=NIFI)
- **Individual keystores** for each NiFi node (CN=nifi0/nifi1, OU=NIFI)
- **Shared truststore** for inter-node communication
- **Client certificate** for admin access

### Certificate Details

**Certificate Configuration:**
- **Format**: JKS (Java KeyStore)
- **Algorithm**: SHA256withRSA with 2048-bit RSA keys
- **Validity**: ~2.3 years (until December 2027)
- **Password**: `supersecret1` (for all keystores and truststores)

**Generated Files per Node:**
```
/opt/certs/nifi0/ (and nifi1/nifi2/)
├── keystore.jks          # Node's private key and certificate
├── truststore.jks        # Trusted CA certificates  
└── nifi.properties       # Auto-configured SSL properties

/opt/nifi/nifi-current/conf/ (copied by init-certs.sh)
├── keystore.jks          # Node's private key and certificate (copied)
└── truststore.jks        # Trusted CA certificates (copied)
```

### Certificate Storage

All certificates are stored in the shared `260_nifi_certs` Docker volume:
```
260_nifi_certs/
├── nifi0/
│   ├── keystore.jks
│   ├── truststore.jks
│   └── nifi.properties
├── nifi1/
│   ├── keystore.jks
│   ├── truststore.jks
│   └── nifi.properties
├── nifi2/
│   ├── keystore.jks
│   ├── truststore.jks
│   └── nifi.properties
├── nifi-cert.pem (CA certificate)
└── nifi-key.key (CA private key)
```

### SSL Configuration

The following SSL properties are automatically configured in `nifi.properties`:
```properties
nifi.security.keystore=./conf/keystore.jks
nifi.security.keystoreType=jks
nifi.security.keystorePasswd=supersecret1
nifi.security.keyPasswd=supersecret1
nifi.security.truststore=./conf/truststore.jks
nifi.security.truststoreType=jks
nifi.security.truststorePasswd=supersecret1
nifi.web.https.host=nifi0  # (or nifi1 for the second node)
nifi.web.https.port=8443
```

### Certificate Regeneration

To regenerate certificates (if needed):
```bash
# Stop the cluster
docker-compose down

# Remove the certificate volume
docker volume rm 260_nifi_certs

# Restart the cluster (certificates will be regenerated)
docker-compose up -d
```

### Local Development Setup

**Important**: For local testing, you need to add hostname resolution to your `/etc/hosts` file:

```bash
# Add these entries to /etc/hosts for local development
echo "127.0.0.1 nifi0 nifi1" | sudo tee -a /etc/hosts
```

This is required because:
- SSL certificates are generated with specific hostnames (nifi0, nifi1)
- Browser SNI (Server Name Indication) validation requires matching hostnames
- Without this, you'll get "Invalid SNI" or certificate errors

## Cluster Configuration

### NiFi 2.6.0 Specific Settings <mcreference link="https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html" index="1">1</mcreference>

- **Java 21 Runtime**: Configured with appropriate heap settings (2GB per node)
- **Cluster Protocol**: Secure communication on port 11443
- **ZooKeeper Integration**: Embedded ZooKeeper coordination
- **Single User Authentication**: Simplified authentication for development

### Network Configuration

The cluster uses a custom Docker network (`nifi-cluster`) with subnet `172.20.0.0/16`:
- **nifi0**: Internal communication on port 8443
- **nifi1**: Internal communication on port 8443  
- **nifi2**: Internal communication on port 8443  
- **zookeeper**: Coordination on port 2181
- **nginx**: Load balancing on port 8443

### Port Mapping

| Service | Internal Port | External Port | Purpose |
|---------|---------------|---------------|---------|
| nginx | 8443 | 8443 | Load balanced HTTPS access |
| nginx | 80 | 80 | HTTP redirect to HTTPS |
| nifi0 | 8443 | 8443 | Direct node HTTPS access |
| nifi1 | 8443 | 8443 | Direct node HTTPS access |
| nifi2 | 8443 | 8443 | Direct node HTTPS access |
| zookeeper | 2181 | 2181 | Cluster coordination |

**Access URLs:**
- **Load Balanced**: https://localhost:8443/nifi/ (recommended)
- **Direct Node Access**: https://nifi0:8443/nifi/ or https://nifi1:8443/nifi/
- **HTTP Redirect**: http://localhost:80/nifi/ → https://localhost:8443/nifi/

## Management Commands

### Start the Cluster
```bash
docker-compose up -d
```

### Stop the Cluster
```bash
docker-compose down
```

### View Logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f nifi0
docker-compose logs -f nifi-toolkit
```

### Check Service Status
```bash
docker-compose ps
```

### Reset the Cluster (Clean Start)
```bash
# Stop and remove all containers and volumes
docker-compose down -v

# Start fresh
docker-compose up -d
```

## Troubleshooting

### Common Issues

1. **Certificates not generated**:
   ```bash
   # Check toolkit logs
   docker-compose logs nifi-toolkit
   
   # Verify certificate volume
   docker volume inspect 260_nifi_certs
   ```

2. **Nodes not joining cluster**:
   ```bash
   # Check ZooKeeper connectivity
   docker-compose logs zookeeper
   
   # Verify node logs
   docker-compose logs nifi0
   docker-compose logs nifi1
   docker-compose logs nifi2
   ```

3. **Load balancer issues**:
   ```bash
   # Check Nginx configuration
   docker-compose logs nginx
   
   # Test direct node access (requires /etc/hosts entry)
   curl -k https://nifi0:8443/nifi/
   curl -k https://nifi1:8443/nifi/
   
   # Test load balanced access
   curl -k https://localhost:8443/nifi/
   ```

4. **SSL Certificate Issues**:
   ```bash
   # Check certificate details
   docker exec nifi0 keytool -list -v -keystore /opt/nifi/nifi-current/conf/keystore.jks -storepass supersecret1
   
   # Verify SSL configuration
   docker exec nifi0 grep "nifi.security" /opt/nifi/nifi-current/conf/nifi.properties
   
   # Test SSL connection
   curl -k -v https://nifi0:8443/nifi/
   ```

5. **Hostname Resolution Issues**:
   ```bash
   # Verify /etc/hosts entry
   grep "nifi0\|nifi1\|nifi2" /etc/hosts
   
   # Should show: 127.0.0.1 nifi0 nifi1 nifi2
   # If missing, add it:
   echo "127.0.0.1 nifi0 nifi1 nifi2" | sudo tee -a /etc/hosts
   ```

### Health Checks

All services include health checks:
```bash
# View health status
docker-compose ps

# Check specific service health
docker inspect --format='{{.State.Health.Status}}' nifi0

# health check
docker ps --filter "name=nifi" --format "table {{.Names}}\t{{.Status}}"
```

### API checks
https://localhost:8444/nifi-api/flow/cluster/summary
https://localhost:8444/nifi-api/system-diagnostics
https://localhost:8444/nifi-api/controller/cluster

doc https://nifi.apache.org/nifi-docs/rest-api.html


### Performance Tuning

For production use, consider adjusting:
- **JVM Heap Size**: Modify `NIFI_JVM_HEAP_INIT` and `NIFI_JVM_HEAP_MAX`
- **Repository Locations**: Use external volumes for better I/O performance
- **Network Settings**: Adjust timeout values in nginx.conf

## Security Considerations

- **Development Use Only**: This setup uses self-signed certificates
- **Default Credentials**: Change the default admin password for production
- **Network Security**: Consider firewall rules for production deployments
- **Certificate Rotation**: Implement certificate rotation for long-term use

## Version Compatibility

- **NiFi Version**: 2.6.0
- **Java Version**: 21 (required for NiFi 2.6.0) <mcreference link="https://medium.com/@eray.ergulec/high-availability-ha-https-installation-guide-for-apache-nifi-2-5-0-on-a-development-environment-8c91cefc1291" index="4">4</mcreference>
- **ZooKeeper Version**: 3.9.2
- **Nginx Version**: 1.25-alpine

## Support

For issues specific to this setup, check:
1. Docker and Docker Compose versions
2. Available system resources (RAM, disk space)
3. Port conflicts with other services
4. Certificate generation logs

For NiFi-specific issues, refer to the [Apache NiFi Documentation](https://nifi.apache.org/docs/nifi-docs/) <mcreference link="https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html" index="1">1</mcreference>.