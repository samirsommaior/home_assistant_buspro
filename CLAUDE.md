# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant custom integration for HDL Buspro building automation systems. It enables control of HDL devices (lights, switches, sensors, climate) through Home Assistant.

## Architecture

The integration follows Home Assistant's platform-based architecture:

```
custom_components/buspro/
├── __init__.py          # Core integration and service registration
├── config_flow.py       # UI configuration flow
├── [platform].py        # Platform implementations (light, switch, sensor, etc.)
└── pybuspro/           # Internal HDL Buspro protocol library
    ├── buspro.py       # Main Buspro class and communication
    ├── core/           # Protocol implementation
    ├── devices/        # Device-specific logic
    └── transport/      # UDP network layer
```

Key architectural patterns:
- Each platform module (light.py, switch.py, etc.) implements Home Assistant entity classes
- The pybuspro library handles all HDL protocol communication via UDP
- Device addressing uses format: `<subnet_id>.<device_id>.<channel>`
- Asynchronous operations using asyncio throughout

## Development Commands

This integration has no build process or external dependencies. For development:

1. **Testing in Home Assistant**:
   ```bash
   # Copy integration to HA config directory
   cp -r custom_components/buspro /path/to/homeassistant/config/custom_components/
   
   # Restart Home Assistant to load changes
   ```

2. **Checking for common issues**:
   ```bash
   # Check for Python syntax errors
   python -m py_compile custom_components/buspro/*.py
   python -m py_compile custom_components/buspro/pybuspro/*.py
   
   # Check imports work correctly
   python -c "from custom_components.buspro import __init__"
   ```

## Key Development Areas

### Adding New Device Support
1. Check `pybuspro/devices/` for existing device implementations
2. Add device class in appropriate file or create new one
3. Update corresponding platform file (e.g., `switch.py` for switches)
4. Follow existing patterns for entity creation and state updates

### Protocol Communication
- All HDL protocol logic is in `pybuspro/`
- Message parsing/creation in `pybuspro/core/parser.py`
- Device commands in `pybuspro/devices/`
- Network communication in `pybuspro/transport/udp.py`

### Configuration Flow
- UI configuration handled in `config_flow.py`
- Strings for UI in `strings.json` and `translations/`
- Supports both UI and YAML configuration

### Service Implementation
- Services defined in `__init__.py` 
- Three main services: send_message, activate_scene, set_universal_switch
- Service schemas use Home Assistant's voluptuous validation

## Important Considerations

1. **No External Dependencies**: The pybuspro library is bundled internally - do not try to pip install it
2. **Home Assistant Compatibility**: Check HA deprecation warnings and update accordingly (see recent commits for examples)
3. **Device Testing**: Physical HDL hardware or gateway required for testing
4. **Async Operations**: All I/O operations must be async - use `async`/`await` patterns
5. **Entity Updates**: Use `async_write_ha_state()` for state updates, not `schedule_update_ha_state()`

## Common Tasks

### Adding a New Platform
1. Create `custom_components/buspro/[platform].py`
2. Implement entity class inheriting from appropriate HA base class
3. Add platform to `PLATFORMS` list in `__init__.py`
4. Implement `async_setup_entry` function

### Debugging Communication
- Enable debug logging in HA configuration.yaml:
  ```yaml
  logger:
    logs:
      custom_components.buspro: debug
  ```
- Check `pybuspro/transport/udp.py` for network communication
- Monitor `_telegram_received_callback` in platform files for incoming messages

### Updating Translations
- Edit `strings.json` for default (English) strings
- Update `translations/[lang].json` for other languages
- Keep keys consistent across all translation files