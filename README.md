# Pkl Systemd

Type-safe [Pkl](https://pkl-lang.org/) configuration for [systemd](https://systemd.io/) unit files.

This package provides comprehensive Pkl modules for generating systemd unit files with full type safety, validation, and IDE support. All systemd unit types and their properties are supported with proper type constraints based on the official systemd documentation.

## Features

- **Complete systemd support**: Service, Timer, Socket, Mount, and Unit configurations
- **Type safety**: All properties use proper types (String, Boolean, enums) with validation
- **Union types**: Properties that accept single values or lists are properly typed as `(String|Listing<String>)?`
- **Documentation**: Comprehensive examples and inline documentation
- **IDE support**: Full IntelliSense and autocomplete with Pkl IDE extensions

## Installation

For installation instructions and usage examples, see the **[latest release notes](https://github.com/declix/pkl-systemd/releases/latest)**.

## Usage

There are two ways to use pkl-systemd:

### Option 1: Direct Amends (Simple)

Directly amend specific unit types:

```pkl
amends "package://pkl.declix.org/pkl-systemd@1.2.3#/Service.pkl"

unit = new {
    description = "My Service"
    after = "network.target"
}

service = new {
    type = "simple"
    execStart = "/usr/bin/my-app"
    restart = "on-failure"
}

install = new {
    wantedBy = "multi-user.target"
}
```

### Option 2: Import and Instantiate (Flexible)

Import the systemd module and create unit instances:

```pkl
import "package://pkl.declix.org/pkl-systemd@1.2.3#/systemd.pkl" as systemd

output {
    text = (new systemd.Service {
        unit = new {
            description = "My Service"
            after = "network.target"
        }
        service = new {
            type = "simple"
            execStart = "/usr/bin/my-app"
            restart = "on-failure"
        }
        install = new {
            wantedBy = "multi-user.target"
        }
    }).output.text
}
```

The import approach is useful when you need to generate multiple units or combine with other Pkl logic.

## Supported Unit Types

| Module | Description | Documentation |
|--------|-------------|---------------|
| `Service.pkl` | Service units for running processes | [systemd.service(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.service.html) |
| `Timer.pkl` | Timer units for scheduled execution | [systemd.timer(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.timer.html) |
| `Socket.pkl` | Socket units for socket-based activation | [systemd.socket(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.socket.html) |
| `Mount.pkl` | Mount units for filesystem mounting | [systemd.mount(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.mount.html) |
| `Target.pkl` | Target units for grouping and synchronization | [systemd.target(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.target.html) |
| `Path.pkl` | Path units for file system monitoring | [systemd.path(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.path.html) |
| `Swap.pkl` | Swap units for swap file/partition management | [systemd.swap(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.swap.html) |
| `Automount.pkl` | Automount units for on-demand mounting | [systemd.automount(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.automount.html) |
| `Slice.pkl` | Slice units for resource management | [systemd.slice(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.slice.html) |
| `Scope.pkl` | Scope units for managing external processes | [systemd.scope(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.scope.html) |
| `Device.pkl` | Device units (automatically generated) | [systemd.device(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.device.html) |
| `Unit.pkl` | Base unit with common properties | [systemd.unit(5)](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html) |

## Quick Start

### Simple Service

Create `hello.pkl`:

```pkl
amends "package://pkl.declix.org/pkl-systemd@0.0.8#/Service.pkl"

unit = new {
    description = "Hello World Service"
    wants = "network-online.target"
    after = "network-online.target"
}

service = new {
    type = "simple"
    execStart = "/usr/bin/echo 'Hello World'"
    restart = "on-failure"
    restartSec = "5"
}

install = new { 
    wantedBy = "multi-user.target" 
}
```

Generate the systemd unit file:

```bash
pkl eval hello.pkl
```

Output:
```ini
[Unit]
After=network-online.target
Description=Hello World Service
Wants=network-online.target

[Service]
ExecStart=/usr/bin/echo 'Hello World'
Restart=on-failure
RestartSec=5
Type=simple

[Install]
WantedBy=multi-user.target
```

### Advanced Service Example

```pkl
amends "package://pkl.declix.org/pkl-systemd@0.0.8#/Service.pkl"

unit = new {
    description = "Advanced Web Service"
    documentation = new Listing {
        "man:web-service(8)"
        "https://example.com/docs"
    }
    
    // Dependencies with both single and multiple values
    requires = "network.target"
    wants = new Listing {
        "network-online.target"
        "time-sync.target"
    }
    after = new Listing {
        "network-online.target"
        "systemd-resolved.service"
    }
    
    // Failure handling
    onFailure = "notify-admin@%n.service"
    startLimitBurst = 5
    startLimitIntervalSec = "300"
}

service = new {
    type = "notify"
    remainAfterExit = false
    
    // Multiple pre-start commands
    execStartPre = new Listing {
        "/usr/bin/mkdir -p /run/web-service"
        "/usr/bin/web-service --validate-config"
    }
    execStart = "/usr/bin/web-service --daemon"
    execReload = "/bin/kill -HUP $MAINPID"
    
    // Restart configuration
    restart = "on-failure"
    restartSec = "10"
    restartMaxDelaySec = "60"
    
    // Environment variables
    environment = new {
        ["LOG_LEVEL"] = "info"
        ["CONFIG_FILE"] = "/etc/web-service/config.yaml"
    }
    environmentFile = "/etc/web-service/environment"
    
    // Timeouts and limits
    timeoutStartSec = "60"
    timeoutStopSec = "30"
    runtimeMaxSec = "24h"
    
    // Process management
    killMode = "mixed"
    sendSIGHUP = true
}

install = new {
    wantedBy = "multi-user.target"
    also = "web-service-monitor.timer"
}
```

### Timer Example

```pkl
amends "package://pkl.declix.org/pkl-systemd@0.0.8#/Timer.pkl"

unit = new {
    description = "Daily backup timer"
    requires = "backup.service"
}

timer = new {
    // Run daily at 2 AM with randomization
    onCalendar = "daily"
    randomizedDelaySec = "30min"
    
    // Persistence and behavior
    persistent = true
    wakeSystem = false
    
    // Clock change handling
    onClockChange = true
    onTimezoneChange = true
}

install = new {
    wantedBy = "timers.target"
}
```

### Socket Example

```pkl
amends "package://pkl.declix.org/pkl-systemd@0.0.8#/Socket.pkl"

unit = new {
    description = "Web application socket"
    requires = "network.target"
}

socket = new {
    // Multiple listen addresses
    listenStream = new Listing {
        "127.0.0.1:8080"
        "/run/webapp.sock"
    }
    
    // Socket permissions
    socketUser = "webapp"
    socketGroup = "webapp"
    socketMode = "0660"
    
    // Connection handling
    accept = true
    maxConnections = 1000
    
    // Advanced options
    keepAlive = true
    noDelay = true
    removeOnStop = true
}

install = new {
    wantedBy = "sockets.target"
}
```

### Mount Example

```pkl
amends "package://pkl.declix.org/pkl-systemd@0.0.8#/Mount.pkl"

unit = new {
    description = "Application data mount"
    requires = "network-online.target"
    conflicts = "umount.target"
}

mount = new {
    what = "/dev/disk/by-uuid/12345678-1234-1234-1234-123456789abc"
    where = "/mnt/app-data"
    type = "ext4"
    options = "defaults,noatime,user_xattr"
    
    // Mount behavior
    timeoutSec = 30.s
    directoryMode = "0755"
    forceUnmount = true
}

install = new {
    wantedBy = "local-fs.target"
}
```

### Target Example

```pkl
amends "package://pkl.declix.org/pkl-systemd@0.0.8#/Target.pkl"

unit = new {
    description = "Web Stack Target"
    wants = new Listing {
        "nginx.service"
        "postgresql.service"
        "redis.service"
    }
    requires = "network.target"
    allowIsolate = true
}

install = new {
    wantedBy = "multi-user.target"
}
```

### Path Example

```pkl
amends "package://pkl.declix.org/pkl-systemd@0.0.8#/Path.pkl"

unit = new {
    description = "Watch for configuration changes"
}

path = new {
    pathChanged = "/etc/myapp"
    pathExistsGlob = "/etc/myapp/*.conf"
    unit = "app-reload.service"
    makeDirectory = true
    triggerLimitIntervalSec = 5.s
}

install = new {
    wantedBy = "multi-user.target"
}
```

### Slice Example

```pkl
amends "package://pkl.declix.org/pkl-systemd@0.0.8#/Slice.pkl"

unit = new {
    description = "Application resource slice"
}

slice = new {
    // CPU limits
    cPUAccounting = true
    cPUWeight = 200
    cPUQuota = "50%"
    
    // Memory limits
    memoryAccounting = true
    memoryHigh = "2G"
    memoryMax = "4G"
    
    // Task limits
    tasksMax = "1000"
}

install = new {
    wantedBy = "slices.target"
}
```

## Type System

### Boolean Properties

All boolean properties are properly typed and render as `yes`/`no` in the output:

```pkl
service = new {
    remainAfterExit = true     // → RemainAfterExit=yes
    guessMainPID = false       // → GuessMainPID=no
}
```

### Union Types

Properties that can accept either a single string or multiple strings use union types:

```pkl
unit = new {
    // Single string
    after = "network.target"
    
    // Multiple strings
    wants = new Listing {
        "network-online.target"
        "time-sync.target"
    }
    
    // Both render correctly:
    // After=network.target
    // Wants=network-online.target
    // Wants=time-sync.target
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
        ["CONFIG_PATH"] = "/etc/app/config.yaml"
        ["DATA_DIR"] = "/var/lib/app"
    }
    // Renders as:
    // Environment=LOG_LEVEL=debug
    // Environment=CONFIG_PATH=/etc/app/config.yaml
    // Environment=DATA_DIR=/var/lib/app
}
```

## Testing

The package includes comprehensive tests to ensure type safety and correct rendering:

```bash
# Run type validation tests
pkl test tests/comprehensive_types.pkl

# Run example rendering tests  
pkl test tests/examples.pkl

# Run validation tests
pkl test tests/validation.pkl
```

## Examples

See the `examples/` directory for comprehensive examples:

**Service Units:**
- `service.pkl` - Basic service configuration
- `enhanced_service.pkl` - Service with advanced features
- `advanced_service.pkl` - Production-ready service with full configuration

**Timer Units:**
- `timer.pkl` - Basic timer
- `backup_timer.pkl` - Comprehensive timer with all options

**Socket Units:**
- `socket.pkl` - Basic socket
- `web_socket.pkl` - Advanced socket with multiple listeners

**Mount Units:**
- `mount.pkl` - Basic filesystem mount
- `network_mount.pkl` - NFS mount with full configuration
- `automount.pkl` - On-demand automount for network filesystems

**Other Unit Types:**
- `target.pkl` - Target unit for grouping services
- `path.pkl` - Path unit for monitoring configuration changes
- `swap.pkl` - Swap unit for swap file management
- `slice.pkl` - Resource management slice for applications
- `scope.pkl` - Scope unit for external process management

## Installation

Add to your `PklProject` dependencies:

```
dependencies {
    ["systemd"] = "package://pkl.declix.org/pkl-systemd@0.0.8"
}
```

## Contributing

This package closely follows the official systemd documentation. When adding new properties or unit types:

1. Reference the official systemd man pages
2. Use proper type constraints (Boolean, String, type aliases)
3. Support union types for properties accepting single/multiple values
4. Add comprehensive tests and examples
5. Update documentation

## Complete systemd Support

This package now provides **complete systemd unit file support** with all 11 unit types:

✅ **Service units** (`systemd.service`) - Process management  
✅ **Timer units** (`systemd.timer`) - Scheduled task execution  
✅ **Socket units** (`systemd.socket`) - Socket-based activation  
✅ **Mount units** (`systemd.mount`) - Filesystem mounting  
✅ **Target units** (`systemd.target`) - Unit grouping and synchronization  
✅ **Path units** (`systemd.path`) - File system monitoring  
✅ **Swap units** (`systemd.swap`) - Swap space management  
✅ **Automount units** (`systemd.automount`) - On-demand mounting  
✅ **Slice units** (`systemd.slice`) - Resource management and cgroups  
✅ **Scope units** (`systemd.scope`) - External process management  
✅ **Device units** (`systemd.device`) - Device representation  

## Future Enhancements

- [ ] pkldoc documentation generation
- [ ] Advanced Duration validation with systemd-specific units
- [ ] Byte size type constraints (e.g., `"1M"`, `"512K"`)
- [ ] Additional systemd specifier support
