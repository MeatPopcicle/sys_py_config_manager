# Config Manager

A flexible and type-safe configuration management library for Python applications.

## Features

- **Type Safety**: Built on Pydantic for automatic validation and type checking
- **Multiple Formats**: Support for YAML, JSON, TOML, and environment files
- **Environment Variable Override**: Override any configuration value with environment variables
- **Nested Configuration**: Support for deeply nested configuration structures
- **Validation**: Built-in and custom validators for configuration values
- **Auto-reload**: Watch configuration files for changes and reload automatically
- **Secret Management**: Mark fields as secrets to prevent logging
- **Multiple Environments**: Easy management of development, staging, and production configs

## Installation

```bash
pip install py-config-manager
```

Or using Poetry:

```bash
poetry add py-config-manager
```

## Quick Start

```python
from config_manager import ConfigManager, ConfigField
from pydantic import BaseModel

class DatabaseConfig(BaseModel):
    host: str = ConfigField(default="localhost", env="DB_HOST")
    port: int = ConfigField(default=5432, env="DB_PORT")
    username: str = ConfigField(env="DB_USER")
    password: str = ConfigField(env="DB_PASSWORD", secret=True)

class AppConfig(BaseModel):
    app_name: str = "My Application"
    debug: bool = ConfigField(default=False, env="DEBUG")
    database: DatabaseConfig = DatabaseConfig()

# Load configuration
config_manager = ConfigManager(AppConfig)
config = config_manager.load_config(
    config_files=["config.yaml", ".env"],
    validate=True
)

# Access configuration values
print(config.app_name)
print(config.database.host)
```

## Configuration Sources

Config Manager supports multiple configuration sources with the following precedence (highest to lowest):

1. Environment variables
2. Configuration files (in the order they are specified)
3. Default values

### Supported File Formats

- **YAML** (`.yaml`, `.yml`)
- **JSON** (`.json`)
- **TOML** (`.toml`)
- **Environment files** (`.env`)

## Advanced Usage

### Custom Validators

```python
from config_manager import ConfigField, validator

class AppConfig(BaseModel):
    port: int = ConfigField(default=8000, env="PORT")
    
    @validator("port")
    def validate_port(cls, v):
        if not 1 <= v <= 65535:
            raise ValueError("Port must be between 1 and 65535")
        return v
```

### Nested Configuration with Environment Variables

```python
class RedisConfig(BaseModel):
    host: str = ConfigField(default="localhost", env="REDIS_HOST")
    port: int = ConfigField(default=6379, env="REDIS_PORT")

class CacheConfig(BaseModel):
    enabled: bool = ConfigField(default=True, env="CACHE_ENABLED")
    ttl: int = ConfigField(default=300, env="CACHE_TTL")
    redis: RedisConfig = RedisConfig()

# Environment variables:
# CACHE_ENABLED=true
# CACHE_TTL=600
# REDIS_HOST=redis.example.com
# REDIS_PORT=6380
```

### Configuration Reloading

```python
config_manager = ConfigManager(AppConfig)

# Enable auto-reload
config = config_manager.load_config(
    config_files=["config.yaml"],
    watch=True,
    reload_callback=lambda new_config: print("Config reloaded!")
)
```

### Multiple Environments

```python
import os

env = os.getenv("APP_ENV", "development")

config = config_manager.load_config(
    config_files=[
        "config/default.yaml",
        f"config/{env}.yaml"
    ]
)
```

## API Reference

### ConfigManager

The main class for managing configurations.

**Methods:**
- `load_config(config_files, validate=True, watch=False, reload_callback=None)`: Load configuration from files
- `validate_config(config)`: Validate a configuration object
- `to_dict(config, include_secrets=False)`: Convert config to dictionary
- `save_config(config, filepath, format=None)`: Save configuration to file

### ConfigField

A Pydantic field with additional configuration management features.

**Parameters:**
- `default`: Default value for the field
- `env`: Environment variable name to override this field
- `secret`: Mark field as secret (won't be included in logs/exports)
- `description`: Field description for documentation
- All standard Pydantic field parameters

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- Built on top of [Pydantic](https://pydantic-docs.helpmanual.io/)
- Inspired by various configuration management best practices in the Python ecosystem