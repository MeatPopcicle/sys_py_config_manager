# py-config-manager

A standalone, reusable configuration management system for Python applications.

## Features

- **TOML Configuration**: Load configuration from TOML files with nested structures
- **Environment Variable Override**: Override any configuration value with environment variables using dot notation
- **Type Validation**: Automatic type validation and coercion for configuration values
- **Schema Validation**: Define configuration schemas with defaults and required fields
- **Multi-source Configuration**: Merge configuration from multiple TOML files and environment variables
- **Runtime Updates**: Dynamically update configuration values at runtime
- **Source Tracking**: Track where each configuration value originated from
- **Nested Access**: Access nested configuration values using dot notation

## Installation

This is currently a standalone module. Copy the `config_manager` package into your project.

## Quick Start

```python
from config_manager import ConfigManager, ConfigSchema

# Define your configuration schema
schema = ConfigSchema()
schema.add_field("debug", bool, default=False)
schema.add_field("host", str, default="localhost")
schema.add_field("port", int, default=8080)

# Define nested database configuration
db_schema = ConfigSchema()
db_schema.add_field("host", str, default="localhost")
db_schema.add_field("port", int, default=5432)
db_schema.add_field("name", str, required=True)
db_schema.add_field("user", str, required=True)
schema.add_nested_schema("database", db_schema)

# Create config manager
config_manager = ConfigManager(
    config_paths=["config.toml"],
    env_prefix="MYAPP",
    schema=schema
)

# Load configuration
config = config_manager.load()

# Access values
print(f"Debug mode: {config_manager.get('debug')}")
print(f"Server: {config_manager.get('host')}:{config_manager.get('port')}")
print(f"Database: {config_manager.get('database.host')}")
```

## Configuration Sources

Config Manager supports multiple configuration sources with the following precedence (highest to lowest):

1. Environment variables (with prefix)
2. Configuration files (merged in order specified)
3. Schema defaults

### Supported File Formats

- **TOML** (`.toml`) - Primary supported format
- **Environment variables** - With configurable prefix

**Note**: YAML and JSON support are not currently implemented.

### Environment Variable Naming

Environment variables use the following naming convention:
- Top-level fields: `{PREFIX}_{FIELD_NAME}`
- Nested fields: `{PREFIX}_{SECTION}__{FIELD_NAME}`

Example:
```bash
# For prefix "MYAPP"
export MYAPP_DEBUG=true
export MYAPP_PORT=9000
export MYAPP_DATABASE__HOST=prod-db.example.com
export MYAPP_DATABASE__USER=produser
```

## Configuration File Example

```toml
# config.toml
debug = false
host = "0.0.0.0"
port = 8080

[database]
host = "db.example.com"
port = 5432
name = "myapp"
user = "appuser"

[cache]
enabled = true
ttl = 300
```

## Advanced Usage

### Multiple Configuration Files

```python
# Load from multiple TOML files (merged in order)
config_manager = ConfigManager(
    config_paths=[
        "config/default.toml",
        "config/production.toml"
    ],
    env_prefix="MYAPP",
    schema=schema
)
```

### Runtime Configuration Updates

```python
# Update configuration at runtime
config_manager.set("debug", True)
config_manager.set("database.host", "new-db.example.com")

# Check if key exists
if config_manager.has("feature_flags.new_feature"):
    print("New feature is configured")

# Get source of configuration value
source = config_manager.get_source("database.host")
print(f"database.host loaded from: {source}")
```

### Custom Validation

```python
from config_manager import ConfigError

def validate_port(port):
    if not 1 <= port <= 65535:
        raise ConfigError(f"Port {port} is out of valid range")
    return port

# Apply validation after loading
config = config_manager.load()
validated_port = validate_port(config_manager.get("port"))
config_manager.set("port", validated_port)
```

## API Reference

### ConfigManager

**Constructor:**
```python
ConfigManager(config_paths=None, env_prefix=None, schema=None, auto_load=True)
```

**Key Methods:**
- `load()` - Load configuration from all sources
- `get(key, default=None)` - Get configuration value (supports dot notation)
- `set(key, value)` - Set configuration value (supports dot notation)
- `has(key)` - Check if configuration key exists
- `get_source(key)` - Get source where value was loaded from
- `to_dict()` - Export configuration as dictionary

### ConfigSchema

**Methods:**
- `add_field(name, field_type, default=None, required=False)` - Add a field to schema
- `add_nested_schema(name, nested_schema)` - Add nested schema section

### Type Conversion

Automatic type conversion is supported for:
- `bool`: "true", "1", "yes", "on" → True; "false", "0", "no", "off" → False
- `int`: Numeric strings → integers
- `float`: Numeric strings → floats  
- `list`: Comma-separated values → list

## Error Handling

```python
from config_manager import ConfigError

try:
    config = config_manager.load()
except ConfigError as e:
    print(f"Configuration error: {e}")
```

## Contributing

This is a standalone module designed to be copied into projects. Improvements and bug fixes are welcome.

## License

This project is licensed under the MIT License - see the LICENSE file for details.