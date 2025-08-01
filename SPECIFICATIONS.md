# CasAInit Technical Specifications

## Overview

CasAInit is a POSIX-compliant shell bootstrapper for Arch Linux that installs and manages a complete local AI stack. It follows strict guidelines for safety, compatibility, and user experience.

## Core Principles

### 1. POSIX Compliance
- **Shell**: Pure POSIX sh (dash-safe)
- **No Bashisms**: No arrays, no `[[`, no `function` keyword
- **Compatibility**: Must work with `/bin/sh` on any POSIX system
- **String handling**: Use parameter expansion, not arrays

### 2. Curl | sh Safety
- **Self-contained**: Single file deployment
- **No external dependencies**: All scripts embedded
- **Idempotent**: Safe to run multiple times
- **Non-destructive**: Never overwrites without confirmation
- **Embedded resources**: Model cache script created on-demand

### 3. Non-Interactive by Default
- **Default behavior**: `FORCE_YES=true`
- **No prompts**: Runs without user interaction
- **Override flag**: `--interactive` or `-i` for prompts
- **Automated deployment**: Suitable for CI/CD

## Architecture

### Directory Structure
```
~/.config/casainit/
├── settings.env          # Environment configuration
├── models.toml          # Model registry cache
└── .bandwidth_test      # Bandwidth test cache

~/.local/share/casainit/
├── ollama/              # Ollama installation
├── piper/               # Piper TTS installation
├── opencode/            # OpenCode installation
└── downloads/           # Downloaded archives

~/models/
├── ollama/              # Ollama models
├── tts/                 # TTS voice models
└── diffusion/           # Stable Diffusion models
```

### Service Architecture
- **Binaries**: Managed via systemd (`casainit-{service}.service`)
- **Containers**: Managed via Docker (`{service}-{user}`)
- **Ports**: Dynamically assigned, stored in settings.env
- **Logs**: Centralized in `~/.local/log/casainit/`

## Color and Output System

### Printf Functions
```sh
# Core functions
__printf_reset_color()      # Reset color codes
__printf_reset_nc()         # No-op for raw mode
__printf_newline_color()    # Newline with color reset
__printf_newline_nc()       # Plain newline

# Progress functions
__printf_percent_color()    # Colored progress bar [=====>   ] 45%
__printf_percent_nc()       # Plain progress [45%] Task
__print_progress()          # Smart wrapper
__print_progress_done()     # Complete progress display

# Monitoring functions
__monitor_progress()        # Monitor command output
__show_spinner()           # Animated spinner with emojis
__show_spinner_custom()    # Custom icon spinner
```

### Color Definitions
```sh
COLOR_RESET="\033[0m"
COLOR_RED="\033[31m"
COLOR_GREEN="\033[32m"
COLOR_YELLOW="\033[33m"
COLOR_BLUE="\033[34m"
COLOR_MAGENTA="\033[35m"
COLOR_CYAN="\033[36m"
COLOR_BOLD="\033[1m"
```

### Emoji Icons
```sh
ICON_SUCCESS="✅"    # Success messages
ICON_ERROR="❌"      # Error messages
ICON_WARN="⚠️"      # Warning messages
ICON_INFO="💡"      # Information messages
ICON_PACKAGE="📦"   # Package operations
ICON_PROGRESS="⏳"  # In-progress tasks
ICON_DOWNLOAD="⬇️"  # Download operations
ICON_INSTALL="🔧"   # Installation tasks
```

### Output Rules
1. **Always use printf**, never echo
2. **Check RAW_MODE** for color/emoji output
3. **Use `%b`** for escape sequences
4. **Reset colors** after each colored output
5. **Icons in color mode only**

## Progress Monitoring

### Monitor Progress Types
```sh
__monitor_progress docker_pull <image>      # Docker image pulls
__monitor_progress curl_download <url> <output>  # File downloads
__monitor_progress ollama_pull <model>      # Ollama model pulls
__monitor_progress apt_install <packages>   # APT installations
__monitor_progress yay_install <packages>   # AUR installations
__monitor_progress generic <task> <steps>   # Generic progress
```

### Progress Bar Format
- **Color mode**: `[======>    ] 65% Downloading model`
- **Raw mode**: `[65%] Downloading model`
- **Width**: 50 characters default
- **Update**: Carriage return (`\r`) for in-place updates

## SSL Certificate Handling

### Let's Encrypt Support
- **Path**: `/etc/letsencrypt/live/domain/` (literal path)
- **Validation**: Check cert/key match
- **Mismatch handling**: Warning only, not error
- **User feedback**: Clear instructions for renewal

### Certificate Functions
```sh
__check_letsencrypt_cert            # Validate LE certificates at /etc/letsencrypt/live/domain/
__print_check "ok|fail" <message>  # Show check status
```

## Hardware Detection

### System Scoring Formula
```
Score = (CPU cores × 0.5) + RAM (GB) + VRAM (GB)
```

### Tier Classification
- **Enterprise**: Score ≥ 64
- **Advanced**: Score ≥ 32
- **Standard**: Score ≥ 16
- **Basic**: Score ≥ 8
- **Minimal**: Score < 8

### Detection Functions
```sh
__detect_cpu_cores()    # CPU core count
__detect_ram_gb()       # System RAM in GB
__detect_gpu_type()     # nvidia|amd|intel|none
__detect_vram_gb()      # GPU VRAM in GB
__calculate_score()     # System score calculation
```

## Model Management

### Model Selection
- **Automatic**: Based on system tier
- **Manual**: User-specified models
- **Filtering**: By type (ollama, tts, diffusion)
- **Caching**: models.toml updated daily

### Model Commands
```sh
casainit models remote [--type=TYPE]    # List available
casainit models installed               # Show installed
casainit models refresh                 # Update cache
casainit models install <model>         # Install model
```

## Service Management

### Systemd Services
```ini
[Unit]
Description=CasAInit {Service}
After=network.target

[Service]
Type=simple
User={user}
ExecStart={binary} {args}
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### Docker Containers
- **Naming**: `{service}-{user}`
- **Restart**: always
- **Network**: host or bridge
- **Volumes**: Bind mounts to local paths
- **GPU**: `--gpus all` when available

## Environment Management

### Settings File Format
```sh
# ~/.config/casainit/settings.env
# ─────────────────────────────────────
# 🚀 CasAInit Environment Configuration
# ─────────────────────────────────────

# Installation Status
INSTALLED_ollama="yes"
VERSION_ollama="0.1.17"

# Service Ports
PORT_openwebui="3000"
PORT_opendiffusion="7860"

# Model Configuration
DEFAULT_OLLAMA_MODEL="llama2:7b"
DEFAULT_TTS_VOICE="en_US-amy-medium"
```

### Environment Functions
```sh
__env_init()            # Initialize config directory
__env_load()            # Load settings into environment
__env_set <key> <val>   # Set and persist value
__env_get <key> [def]   # Get value with default
__env_save()            # Write current environment
```

## Command Line Interface

### Argument Parsing
```sh
# Position 1: Command
install|uninstall|update|fix|gpu|status|service|models

# Position 2: Subcommand (context-dependent)
service start|stop|restart|status [service]
models remote|installed|refresh|install [model]

# Flags (any position)
--raw                   # Disable colors/emojis
--interactive, -i       # Enable prompts
--yes, -y              # Assume yes (default)
--remove-data          # Remove data on uninstall
--type=TYPE            # Filter by type
--help, -h             # Show help
--version, -v          # Show version
```

### Default Behavior
- **No arguments**: Run install command
- **Non-interactive**: No prompts by default
- **Safe mode**: Confirm destructive operations

## Installation Process

### Prerequisites
1. Check root/sudo access
2. Update package database
3. Install core packages:
   - curl, git, base-devel
   - docker, docker-compose
   - yay (AUR helper)

### Tool Installation Order
1. **Ollama**: Binary + systemd service
2. **Piper**: Binary + voice model
3. **OpenWebUI**: Docker container
4. **OpenDiffusion**: Docker container
5. **Continue**: VS Code extension
6. **MCP Tools**: From registry

### Post-Installation
1. Create cron jobs
2. Set up log rotation
3. Update PATH in shell RC
4. Fix permissions
5. Show status

## Cron Jobs

### Schedule
```cron
# Model cache refresh (2 AM daily)
0 2 * * * root /usr/local/bin/casainit-model-cache.sh

# Fix permissions (6 AM, 2 PM, 10 PM)
0 6,14,22 * * * root find /var/lib/casainit/models -type d -exec chmod 777 {} +

# Log cleanup (Monday 1 AM)
0 1 * * 1 root find ~/.local/log/casainit -name "*.log" -mtime +7 -delete
```

## Error Handling

### Exit Codes
- `0`: Success
- `1`: General error
- `2`: Missing dependencies
- `3`: Permission denied
- `4`: Network error
- `5`: Invalid arguments

### Error Recovery
- **Idempotent operations**: Safe to retry
- **Automatic fixes**: `casainit fix` command
- **Clear messages**: Actionable error output
- **Fallback options**: Alternative approaches

## Security Considerations

### Permissions
- **Model directories**: 777 (shared access)
- **Config files**: 644 (user read/write)
- **Binaries**: 755 (executable)
- **Services**: Run as user, not root

### Network Security
- **Local only**: Services bound to localhost
- **No external access**: Firewall recommended
- **API keys**: Stored in environment
- **HTTPS**: For external model downloads

## Testing Guidelines

### Unit Tests
- Test each function independently
- Mock system commands
- Verify POSIX compliance
- Check error handling

### Integration Tests
- Full installation flow
- Service management
- Model operations
- Uninstall/cleanup

### Compatibility Tests
- Arch Linux (primary)
- Other distros (future)
- Different shells (sh, dash, bash)
- Container environments

## Contributing Rules

### Code Style
1. POSIX sh only
2. 4-space indentation
3. Function names: `__function_name`
4. Constants: `UPPER_CASE`
5. Comments for complex logic

### Pull Request Requirements
1. Pass shellcheck
2. Update documentation
3. Test on fresh Arch system
4. Maintain backward compatibility
5. Follow existing patterns

### Commit Messages
```
type: brief description

Longer explanation if needed.

Fixes #123
```

Types: feat, fix, docs, style, refactor, test, chore

## Version History

### 0.1.0 (Current)
- Initial release
- Core functionality
- Basic tool support

### Roadmap
- [ ] Multi-distro support
- [ ] Web UI for management
- [ ] Plugin system
- [ ] Cloud model support
- [ ] Cluster deployment

## License

MIT License - See LICENSE file

## Repository

- **Organization**: casapps
- **Repository**: casainit
- **URL**: https://github.com/casapps/casainit

---

*This specification is authoritative. All code must conform to these rules.*