# Traefik 101

## Prerequisites

Silahkan clone repository materi terlebih dahulu.

```bash
$ git clone https://github.com/azemoning/klim-traefik101.git
```

dan pastikan Docker dan Docker Compose sudah terinstall pada sistem masing-masing.
```bash
$ docker version && docker-compose version
```

## Overview

Traefik adalah Edge Router open-source yang memberikan pengalaman atau experience dalam publishing service-service milik kita menjadi mudah dan menyenangkan!

Traefik menerima requests atas nama sistem milik kita dan mencari tahu komponen mana yang bertanggung jawab untuk menanganinya.

Apa yang membedakan Traefik adalah, selain memiliki banyak fitur, Traefik dapat secara otomatis menemukan konfigurasi yang tepat untuk service kita (Auto Service Discovery).

Traefik secara native sesuai dengan mayoritas teknologi cluster yang ada saat ini, seperti Kubernetes, Docker, Docker Swarm, AWS, Mesos, Marathon, dan banyak lagi; dan dapat menangani semua dalam waktu yang bersamaan.

## Concepts

### Entrypoints

Entrypoint adalah titik masuk (literally entrypoint) network ke Traefik, yang dimana request dari client akan masuk melalui titik ini.

**Contoh konfigurasi dari entrypoint Traefik**

```toml
[entryPoints]
  [entryPoints.https]
  address = ":443"
  [entryPoints.https.tls]
    [entryPoints.https.tls.ClientCA]
    files = ["tests/clientca1.crt", "tests/clientca2.crt"]
    optional = false
    [[entryPoints.https.tls.certificates]]
    certFile = "tests/traefik.crt"
    keyFile = "tests/traefik.key"
```

### Frontends

Frontend terdiri dari satu set aturan yang menentukan bagaimana request yang masuk dan diteruskan dari entrypoint ke backend.

**Contoh konfigurasi dari frontend pada docker-compose menggunakan labels**

```dockerfile
labels:
      - "traefik.frontend.rule=PathPrefixStrip:/"
      - "traefik.frontend.rule=Host:www.example.com"
      - "traefik.enable=true"
      - "traefik.frontend.entryPoints=http"
```

### Backends

Backend bertanggung jawab untuk menyeimbangkan lalu lintas yang datang dari satu atau lebih frontend ke satu set server http.

**Contoh konfigurasi dari backend Traefik**

```toml
[backends]
  [backends.backend1]
    # ...
    [backends.backend1.servers.server1]
    url = "http://172.17.0.2:80"
    weight = 10
    [backends.backend1.servers.server2]
    url = "http://172.17.0.3:80"
    weight = 1
  [backends.backend2]
    # ...
    [backends.backend2.servers.server1]
    url = "https://172.17.0.4:443"
    weight = 1
    [backends.backend2.servers.server2]
    url = "https://172.17.0.5:443"
    weight = 2
  [backends.backend3]
    # ...
    [backends.backend3.servers.server1]
    url = "h2c://172.17.0.6:80"
    weight = 1
```

## Configuration

Konfigurasi pada Traefik memiliki dua bagian, yaitu:

- Static Configuration, yang hanya akan dibaca pada saat pertama kali service Traefik berjalan. 
- Dynamic Configuration, yang bisa melakukan *hot-reload* (tidak perlu melakukan restart service Traefik)

### Static Configuration

Static configuration adalah konfigurasi global yang mengatur koneksi ke konfigurasi backend dan entrypoint.

Contoh static configuration dari file traefik.toml

```toml
################################################################
# Global configuration
################################################################

# Enable debug mode
#
# Optional
# Default: false
#
# debug = true

# Log level
#
# Optional
# Default: "ERROR"
#
# logLevel = "DEBUG"

# Entrypoints to be used by frontends that do not specify any entrypoint.
# Each frontend can specify its own entrypoints.
#
# Optional
# Default: ["http"]
#
# defaultEntryPoints = ["http", "https"]

################################################################
# Entrypoints configuration
################################################################

# Entrypoints definition
#
# Optional
# Default:
[entryPoints]
    [entryPoints.http]
    address = ":80"

################################################################
# Traefik logs configuration
################################################################

# Traefik logs
# Enabled by default and log to stdout
#
# Optional
#
# [traefikLog]

# Sets the filepath for the traefik log. If not specified, stdout will be used.
# Intermediate directories are created if necessary.
#
# Optional
# Default: os.Stdout
#
# filePath = "log/traefik.log"

# Format is either "json" or "common".
#
# Optional
# Default: "common"
#
# format = "common"

################################################################
# Access logs configuration
################################################################

# Enable access logs
# By default it will write to stdout and produce logs in the textual
# Common Log Format (CLF), extended with additional fields.
#
# Optional
#
# [accessLog]

# Sets the file path for the access log. If not specified, stdout will be used.
# Intermediate directories are created if necessary.
#
# Optional
# Default: os.Stdout
#
# filePath = "/path/to/log/log.txt"

# Format is either "json" or "common".
#
# Optional
# Default: "common"
#
# format = "common"

################################################################
# API and dashboard configuration
################################################################

# Enable API and dashboard
[api]

  # Name of the related entry point
  #
  # Optional
  # Default: "traefik"
  #
  # entryPoint = "traefik"

  # Enabled Dashboard
  #
  # Optional
  # Default: true
  #
  # dashboard = false

################################################################
# Ping configuration
################################################################

# Enable ping
[ping]

  # Name of the related entry point
  #
  # Optional
  # Default: "traefik"
  #
  # entryPoint = "traefik"

################################################################
# Docker configuration backend
################################################################

# Enable Docker configuration backend
[docker]

# Docker server endpoint. Can be a tcp or a unix socket endpoint.
#
# Required
# Default: "unix:///var/run/docker.sock"
#
# endpoint = "tcp://10.10.10.10:2375"

# Default domain used.
# Can be overridden by setting the "traefik.domain" label on a container.
#
# Optional
# Default: ""
#
# domain = "docker.localhost"

# Expose containers by default in traefik
#
# Optional
# Default: true
#
# exposedByDefault = true
```

### Dynamic Configuration

Traefik dapat melakukan *hot-reload* pada rule-rule disediakan oleh berbagai macam konfigurasi backend, contoh: Docker.

Kita hanya perlu meng-enable option ***watch*** untuk membuat Traefik mengawasi perubahan konfigurasi backend dan men-generate ulang konfigurasi tersebut secara otomatis. Route ke service akan dibuat dan di-update secara langsung pada setiap perubahan.

## Presentation Files

[Download](https://docs.google.com/presentation/d/1xeOZ3YqORwAOBoGakCPzgH5j-s1JL1nmkYvYM6fDkG8/edit?usp=sharing)
