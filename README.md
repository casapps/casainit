# CasAInit - Local AI Stack Bootstrapper for Arch Linux

CasAInit is a fully self-contained, POSIX-compliant shell bootstrapper that installs, configures, and integrates a complete local AI stack on Arch Linux systems.

## Features

- **Fully Automated**: Single command installation of entire AI stack
- **POSIX Compliant**: Works with dash/sh, no bash dependencies
- **Hardware Aware**: Automatically detects CPU, RAM, GPU, and VRAM to install optimal models
- **Self-Repairing**: Built-in fix command to repair permissions and configurations
- **Service Management**: Unified interface for managing all AI services

## Supported Tools

1. **Ollama** - Local LLM runtime (binary + systemd)
2. **OpenCode** - Code-focused AI assistant (binary + systemd)
3. **Piper** - Text-to-speech engine (binary + systemd)
4. **OpenWebUI** - Web interface for AI models (Docker container)
5. **OpenDiffusion** - Image generation (Docker container)
6. **Continue** - VS Code AI extension
7. **MCP Tools** - Additional AI tools from registry

## Quick Start

One-line install (curl | sh safe):
```bash
curl -sL https://raw.githubusercontent.com/casapps/casainit/main/casainit | sh
```

Or download and run:
```bash
wget https://raw.githubusercontent.com/casapps/casainit/main/casainit
chmod +x casainit
./casainit
```

## Commands

```bash
casainit                           # Install everything (default)
casainit status                    # Show installation status
casainit gpu                       # Show GPU information
casainit service start             # Start all services
casainit service restart ollama    # Restart specific service
casainit models remote             # List available models
casainit models installed          # Show installed models
casainit fix                       # Repair permissions and configs
casainit update                    # Update all tools
casainit uninstall                 # Remove CasAInit (preserves models)
casainit uninstall --remove-data   # Remove everything including models
```

## Flags

- `--raw` - Disable colors and emojis (for scripting)
- `--interactive` / `-i` - Run in interactive mode (prompt for options)
- `--yes` / `-y` - Assume yes to all prompts (default)
- `--remove-data` - Remove all models and configurations during uninstall
- `--help` / `-h` - Show help message
- `--version` / `-v` - Show version
- `--type=TYPE` - Filter by type (ollama,tts,diffusion)

## System Requirements

- Arch Linux
- 8GB+ RAM recommended
- 50GB+ free disk space
- Optional: NVIDIA/AMD GPU for acceleration

## File Layout

```
$HOME/.local/bin/              # Binaries
$HOME/.local/share/casaicli/   # Archives and extracted tools
$HOME/.config/casaicli/        # Configurations
$HOME/.local/log/casaicli/     # Logs
/var/lib/casaicli/models/      # Shared model storage
```

## Model Selection

CasaICLI automatically scores your system and installs appropriate models:

- **Score = (CPU cores × 0.5) + RAM + VRAM**
- **Tier 1 (48+)**: High-end models (CodeLlama 13B, etc.)
- **Tier 2 (32-47)**: Mid-range models (Mistral 7B, etc.)
- **Tier 3 (<32)**: Lightweight models (Llama2 7B, etc.)

## Configuration

All settings stored in `~/.config/casaicli/settings.env`:

```bash
SYSTEM_CPU_CORES="8"
SYSTEM_RAM_GB="16"
SYSTEM_VRAM_GB="6"
SYSTEM_GPU_TYPE="nvidia"
INSTALLED_ollama="yes"
PORT_ollama="11434"
```

## Service Management

Binary services use systemd:
```bash
systemctl --user status casaicli-ollama
```

Container services use Docker:
```bash
docker ps | grep casaicli
```

## Troubleshooting

Fix permissions:
```bash
casainit fix
```

Check logs:
```bash
tail -f ~/.local/log/casainit/*.log
```

Reinstall specific tool:
```bash
casainit uninstall ollama
casainit install ollama
```

## License

MIT License - See LICENSE file for details

## Contributing

Contributions welcome! Please ensure:
- POSIX compliance (test with dash)
- No bashisms
- Follow existing code style
- Test on fresh Arch Linux install