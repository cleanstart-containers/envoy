## Container Documentation for Envoy Documentation

The CleanStart Envoy image provides a production-ready, security-hardened container optimized for enterprise environments. Built on a minimal base OS with comprehensive security hardening, this image delivers reliable application execution with advanced security features.

📌 **Base Foundation**: Production-ready container from cleanstart.

**Image Path**: `ghcr.io/cleanstart-containers/envoy`


## Pull Latest Image
Download the container image from the registry

```bash
docker pull ghcr.io/cleanstart-containers/envoy:latest
```
```bash
docker pull ghcr.io/cleanstart-containers/envoy:latest-dev
```

## Minimal envoy.yaml Example
A minimal configuration to get Envoy running on port 8080 with an admin interface on port 9901:

```bash
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 8080
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                direct_response:
                  status: 200
                  body:
                    inline_string: "Envoy is running\n"
          http_filters:
          - name: envoy.filters.http.router
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.http.router.v3.Router

admin:
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 9901

# Note: check port availability
```

## Basic Run
Run the container with basic configuration

```bash
docker run -it --name envoy \
  -v ~/envoy.yaml:/etc/envoy/envoy.yaml:ro \
  ghcr.io/cleanstart-containers/envoy:latest \
  envoy -c /etc/envoy/envoy.yaml
```

## Production Deployment
Deploy with production security settings

```bash
docker run -d --name envoy-prod \
  --security-opt=no-new-privileges \
  --user 1000:1000 \
  --restart unless-stopped \
  -p 8080:8080 \
  -p 9901:9901 \
  -v ~/envoy.yaml:/etc/envoy/envoy.yaml:ro \
  ghcr.io/cleanstart-containers/envoy:latest \
  envoy -c /etc/envoy/envoy.yaml
```

Volume Mount Mount local directory for persistent data

```bash
docker run \
  -v /app:/app \
  -v ~/envoy.yaml:/etc/envoy/envoy.yaml:ro \
  ghcr.io/cleanstart-containers/envoy:latest \
  envoy -c /etc/envoy/envoy.yaml
```

Port Forwarding Run with custom port mappings

```bash
docker run \
  -p 8080:8080 \
  -p 9901:9901 \
  -v ~/envoy.yaml:/etc/envoy/envoy.yaml:ro \
  ghcr.io/cleanstart-containers/envoy:latest \
  envoy -c /etc/envoy/envoy.yaml
```

## Verify the Container
```bash
curl http://localhost:8080
```
### Expected output: Envoy is running

## Check the admin endpoint health
```bash
curl http://localhost:9901/ready
```
### Expected output: LIVE

## Environment Variables
Configuration options available through environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| ENV | production | Environment mode |
| LOG_LEVEL | info | Logging level |


## Kubernetes Security Context
Recommended security context for Kubernetes deployments

```yaml
securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
  runAsUser: 1000
  runAsGroup: 1000
```

## Documentation Resources
Essential links and resources for further information

- **Container Registry**: [https://www.cleanstart.com/](https://www.cleanstart.com/)
- **CleanStart Community Images**: [https://hub.docker.com/u/cleanstart](https://hub.docker.com/u/cleanstart)
- **How-to-Run CleanStart images & sample projects**: [https://github.com/cleanstart-dev/cleanstart-containers](https://github.com/cleanstart-dev/cleanstart-containers)
  - How to run sample projects using Dockerfile
  - How to deploy via Kubernetes YAML
  - How to migrate from public images to CleanStart images

---

**Vulnerability Disclaimer**

CleanStart offers Docker images that include third-party open-source libraries and packages maintained by independent contributors. While CleanStart maintains these images and applies industry-standard security practices, it cannot guarantee the security or integrity of upstream components beyond its control.

Users acknowledge and agree that open-source software may contain undiscovered vulnerabilities or introduce new risks through updates. CleanStart shall not be liable for security issues originating from third-party libraries, including but not limited to zero-day exploits, supply chain attacks, or contributor-introduced risks.

Security remains a shared responsibility: CleanStart provides updated images and guidance where possible, while users are responsible for evaluating deployments and implementing appropriate controls.
