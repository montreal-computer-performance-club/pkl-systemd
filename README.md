# Pkl Systemd

Type-safe [Pkl](https://pkl-lang.org/) configuration for [systemd](https://systemd.io/) unit files.

This package provides comprehensive Pkl modules for generating systemd unit files with full type safety, validation, and IDE support. All systemd unit types and their properties are supported with proper type constraints based on the official systemd documentation.

## Features

- **Complete systemd support**: all eleven unit types, from Service to Device
- **Podman Quadlet support**: Container, Pod, Volume, Network, Kube, Build and Image units
- **Type safety**: all properties use proper types (String, Boolean, Duration, enums, listings) with validation

## Installation

Add the package to your `PklProject` dependencies:

```
dependencies {
    ["systemd"] = "package://github.com/montreal-computer-performance-club/pkl-systemd/releases/download/X.Y.Z/pkl-systemd@X.Y.Z"
}
```

For the current version and release-specific notes, see the **[latest release notes](https://github.com/montreal-computer-performance-club/pkl-systemd/releases/latest)**.

## Quick Start

Create `hello.pkl` (this is [`examples/hello.pkl`](examples/hello.pkl), with the package URI in place of the relative path):

```pkl
amends "package://github.com/montreal-computer-performance-club/pkl-systemd/releases/download/X.Y.Z/pkl-systemd@X.Y.Z#/Service.pkl"

unit = new {
  description = "Hello World Service"
  wants { "network-online.target" }
  after { "network-online.target" }
}

service = new {
  type = "simple"
  execStart { "/usr/bin/echo 'Hello World'" }
  restart = "on-failure"
  restartSec = 5.s
}

install = new {
  wantedBy { "multi-user.target" }
}
```

Generate the systemd unit file:

```bash
pkl eval hello.pkl
```

Output:

```ini
[Unit]
Description=Hello World Service
Wants=network-online.target
After=network-online.target

[Install]
WantedBy=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/echo 'Hello World'
Restart=on-failure
RestartSec=5s
```

## Usage

There are two ways to use pkl-systemd.

### Option 1: Direct Amends (Simple)

Amend the module for the unit type you want, as in the Quick Start above. The module renders itself as a unit file, so `pkl eval` produces the output directly.

### Option 2: Import and Instantiate (Flexible)

Import the `systemd` module and create unit instances:

```pkl
import "package://github.com/montreal-computer-performance-club/pkl-systemd/releases/download/X.Y.Z/pkl-systemd@X.Y.Z#/systemd.pkl" as systemd

output {
  text = (new systemd.Service {
    unit = new {
      description = "Hello World Service"
      wants { "network-online.target" }
      after { "network-online.target" }
    }
    service = new {
      type = "simple"
      execStart { "/usr/bin/echo 'Hello World'" }
      restart = "on-failure"
    }
    install = new {
      wantedBy { "multi-user.target" }
    }
  }).output.text
}
```

The import approach is useful when you need to generate multiple units or combine with other Pkl logic.

## Supported Unit Types and Examples

Every unit type has at least one example in the [`examples/`](examples/) directory. The examples are rendered as part of the test suite, so they are always up to date with the modules.

| Module | Description | Documentation | Examples |
|--------|-------------|---------------|----------|
| `Service.pkl` | Service units for running processes | [systemd.service(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html) | [hello](examples/hello.pkl), [podman-service](examples/podman-service.pkl), [reference-service](examples/reference-service.pkl) |
| `Timer.pkl` | Timer units for scheduled execution | [systemd.timer(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html) | [timer](examples/timer.pkl), [backup-timer](examples/backup-timer.pkl) |
| `Socket.pkl` | Socket units for socket-based activation | [systemd.socket(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.socket.html) | [socket](examples/socket.pkl), [web-socket](examples/web-socket.pkl) |
| `Mount.pkl` | Mount units for filesystem mounting | [systemd.mount(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.mount.html) | [mount](examples/mount.pkl), [network-mount](examples/network-mount.pkl) |
| `Automount.pkl` | Automount units for on-demand mounting | [systemd.automount(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.automount.html) | [automount](examples/automount.pkl) |
| `Target.pkl` | Target units for grouping and synchronization | [systemd.target(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.target.html) | [target](examples/target.pkl) |
| `Path.pkl` | Path units for file system monitoring | [systemd.path(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.path.html) | [path](examples/path.pkl) |
| `Swap.pkl` | Swap units for swap file/partition management | [systemd.swap(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.swap.html) | [swap](examples/swap.pkl) |
| `Slice.pkl` | Slice units for resource management | [systemd.slice(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.slice.html) | [slice](examples/slice.pkl) |
| `Scope.pkl` | Scope units for managing external processes | [systemd.scope(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.scope.html) | [scope](examples/scope.pkl) |
| `Device.pkl` | Device units (automatically generated) | [systemd.device(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.device.html) | |
| `Unit.pkl` | Base unit with common properties | [systemd.unit(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html) | |

### Podman Quadlet Units

[Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) units are systemd unit files with an extra Podman section, such as `[Container]`, that Podman's systemd generator turns into a regular service. The modules in [`podman/`](podman/) model them; they share the `[Unit]`, `[Install]` and `[Service]` sections with the rest of the package.

```pkl
amends "package://github.com/montreal-computer-performance-club/pkl-systemd/releases/download/X.Y.Z/pkl-systemd@X.Y.Z#/podman/Container.pkl"

container {
  image = "docker.io/library/nginx:latest"
  publishPort { "8080:80" }
}

install {
  wantedBy { "default.target" }
}
```

```bash
pkl eval nginx.pkl > ~/.config/containers/systemd/nginx.container
systemctl --user daemon-reload
systemctl --user start nginx.service
```

The `podman.pkl` module re-exports every Quadlet unit type, mirroring `systemd.pkl`.

| Module | Description | Documentation | Examples |
|--------|-------------|---------------|----------|
| `podman/Container.pkl` | Run a container | [Container units](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#container-units-container) | [nginx](examples/podman/nginx.pkl), [postgres](examples/podman/postgres.pkl), [redis](examples/podman/redis.pkl) |
| `podman/Pod.pkl` | Create a pod for containers to join | [Pod units](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#pod-units-pod) | [web-pod](examples/podman/web-pod.pkl), [database-pod](examples/podman/database-pod.pkl) |
| `podman/Volume.pkl` | Create a volume | [Volume units](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#volume-units-volume) | [postgres-volume](examples/podman/postgres-volume.pkl), [shared-volume](examples/podman/shared-volume.pkl) |
| `podman/Network.pkl` | Create a network | [Network units](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#network-units-network) | [app-network](examples/podman/app-network.pkl), [isolated-network](examples/podman/isolated-network.pkl) |
| `podman/Kube.pkl` | Run a Kubernetes YAML file with `podman kube play` | [Kube units](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#kube-units-kube) | [webapp-kube](examples/podman/webapp-kube.pkl), [microservices-kube](examples/podman/microservices-kube.pkl) |
| `podman/Build.pkl` | Build an image | [Build units](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#build-units-build) | [webapp-build](examples/podman/webapp-build.pkl), [multistage-build](examples/podman/multistage-build.pkl) |
| `podman/Image.pkl` | Pull an image | [Image units](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#image-units-image) | [redis-image](examples/podman/redis-image.pkl), [private-registry-image](examples/podman/private-registry-image.pkl) |
| `podman/Quadlet.pkl` | Base module with the `[Quadlet]` section | [Quadlet section](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#quadlet-section-quadlet) | |

## Type System

### Property Names

A property is named after its unit file key with the first letter in lower case, and nothing else changes. Keys that start with an acronym therefore look unusual, but they render exactly as systemd and Quadlet expect them:

```pkl
slice = new { cPUWeight = 200 }   // CPUWeight=200
container { dNS { "10.0.0.1" } }  // DNS=10.0.0.1
pod { uIDMap { "0:100000:65536" } } // UIDMap=0:100000:65536
```

### Repeatable Properties

systemd options that may be specified more than once (`After=`, `Wants=`, `ExecStart=`, `WantedBy=`, ...) are typed as `Listing<String>`. Each element renders as its own line:

```pkl
unit = new {
  wants {
    "network-online.target"
    "time-sync.target"
  }
}
// Wants=network-online.target
// Wants=time-sync.target
```

Repeatable properties deliberately do not accept a plain string. Using a listing everywhere means that amending a unit *appends* to the inherited values by default, which is usually what you want:

```pkl
amends "hello.pkl"

unit {
  after { "time-sync.target" }
}
// After=network-online.target
// After=time-sync.target
```

To discard the inherited values instead, assign a new listing:

```pkl
amends "hello.pkl"

unit {
  after = new Listing { "time-sync.target" }
}
// After=time-sync.target
```

### Boolean Properties

Boolean properties render as `yes`/`no`:

```pkl
service = new {
  remainAfterExit = true     // RemainAfterExit=yes
  guessMainPID = false       // GuessMainPID=no
}
```

### Durations

Time-valued properties use Pkl durations:

```pkl
service = new {
  restartSec = 5.s           // RestartSec=5s
  timeoutStopSec = 1.min     // TimeoutStopSec=60s
}
```

### Type Aliases

Enum-like values use type aliases for validation:

```pkl
service = new {
  type = "notify"                    // ServiceType
  restart = "on-failure"             // RestartPolicy
  killMode = "mixed"                 // KillMode
  notifyAccess = "main"              // NotifyAccess
}
```

### Environment Variables

Environment variables are handled as mappings:

```pkl
service = new {
  environment = new {
    ["LOG_LEVEL"] = "debug"
    ["DATA_DIR"] = "/var/lib/app"
  }
  // Environment=LOG_LEVEL=debug
  // Environment=DATA_DIR=/var/lib/app
}
```

## Testing

The package includes tests for type validation and for rendering every file in `examples/`, including `examples/podman/`:

```bash
pkl test
```

## Contributing

This package closely follows the official systemd documentation. When adding new properties or unit types:

1. Reference the official systemd man pages, or [podman-systemd.unit(5)](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) for Quadlet units
2. Use proper type constraints (Boolean, Duration, type aliases, `Listing<String>` for repeatable options)
3. Add comprehensive tests and examples
4. Update documentation

### Releasing

Releases are cut by pushing a tag named after the bare version:

```bash
git tag 1.2.3
git push origin 1.2.3
```

The release workflow runs the tests, packages the project with that version, and publishes a GitHub release with the package metadata and zip attached. The tag name is part of the package URI, so it must be a plain `X.Y.Z` with no prefix.

## Future Enhancements

- [ ] pkldoc documentation generation
- [ ] Byte size type constraints (e.g., `"1M"`, `"512K"`)
- [ ] Additional systemd specifier support
