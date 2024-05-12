# Caddy L4 Docker Image

This is a simple Docker image for Caddy v2 with Layer 4 (TCP/UDP) support.

## Usage

You can run this Docker image using the following commands:

Pull the Docker image:

```bash
docker pull ghcr.io/roamer7038/caddy-l4-docker:latest
```

Run the Docker container with the appropriate configuration file and volume mounts:

```
docker run -d --name caddy-l4-docker -p 80:80 -p 443:443/tcp -p 443:443/udp \
  -v $(pwd)/caddy.yaml:/etc/caddy/caddy.yaml:ro \
  -v caddy_data:/data \
  -v caddy_config:/config \
  ghcr.io/roamer7038/caddy-l4-docker:latest
```

This command mounts the local caddy.yaml file into the container, ensuring Caddy uses your custom configuration. It also mounts caddy_data and caddy_config to preserve SSL certificates and other configuration across container restarts.

Docker Compose example:

```yaml
services:
  caddy:
    image: ghcr.io/roamer7038/caddy-l4-docker:latest
    container_name: caddy-l4
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      # The caddy-l4 module does not support the Caddyfile
      # supported configuration is in YAML format/JSON format
      - ./caddy.yaml:/etc/caddy/caddy.yaml:ro
      - caddy_data:/data
      - caddy_config:/config
    extra_hosts:
      - "host.docker.internal:host-gateway"
    command: run --config /etc/caddy/caddy.yaml --adapter yaml

volumes:
  caddy_data:
  caddy_config:
```

You can build the Docker image yourself using the following command:

```bash
docker compose build
docker compose up -d
```

## Configuration

You can configure Caddy by mounting a configuration file to `/etc/caddy/caddy.yaml`.
The supported configuration formats are JSON and YAML.
Does not support Caddyfile format.

Example configuration in YAML format:

```yaml
# /etc/caddy/caddy.yaml
# The Caddyfile configuration in YAML format

apps:
  # Default Caddy HTTP Server
  http:
    servers:
      server:
        listen: [":3000"]
        routes:
          # configure the following for web server
          - handle:
              - handler: file_server
                root: "/usr/share/caddy"

  # Caddy Layer4 Server
  # https://caddyserver.com/docs/modules/layer4
  layer4:
    servers:
      server:
        listen: [":80"]
        routes:
          # configure the following for layer4 proxy
          - handle:
              - handler: proxy
                upstreams:
                  # To specify the host machine localhost for upstream
                  # - dial: ["host.docker.internal:3000"]
                  - dial: ["localhost:3000"]
```

For more information, see the official documentation:
[https://caddyserver.com/docs/modules/layer4](https://caddyserver.com/docs/modules/layer4)

Example of use:

- [livekit-meet-docker](https://github.com/roamer7038/livekit-meet-docker/blob/main/caddy.yaml)

## Build

You can build the image yourself with the following command:

```bash
docker build -t caddy-l4-docker .
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
