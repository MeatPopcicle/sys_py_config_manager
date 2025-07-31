# API Reference

## ConfigManager

The main class for managing application configurations.

```python
from config_manager import ConfigManager
```

### Constructor

```python
ConfigManager(config_class: Type[BaseModel])
```

**Parameters:**
- `config_class`: A Pydantic model class that defines your configuration schema

### Methods

#### load_config

```python
load_config(
    config_files: List[Union[str, Path]] = None,
    config_dict: Dict[str, Any] = None,
    validate: bool = True,
    watch: bool = False,
    reload_callback: Callable[[BaseModel], None] = None
) -> BaseModel
```

Load configuration from multiple sources.

**Parameters:**
- `config_files`: List of configuration file paths to load (optional)
- `config_dict`: Dictionary of configuration values (optional)
- `validate`: Whether to validate the configuration (default: True)
- `watch`: Enable file watching for auto-reload (default: False)
- `reload_callback`: Function to call when configuration is reloaded

**Returns:**
- Configured instance of your config class

**Example:**
```python
config = config_manager.load_config(
    config_files=["config.yaml", ".env"],
    validate=True
)
```

#### validate_config

```python
validate_config(config: BaseModel) -> None
```

Validate a configuration object.

**Parameters:**
- `config`: Configuration object to validate

**Raises:**
- `ValidationError`: If configuration is invalid

#### to_dict

```python
to_dict(
    config: BaseModel,
    include_secrets: bool = False,
    exclude_none: bool = True
) -> Dict[str, Any]
```

Convert configuration to dictionary.

**Parameters:**
- `config`: Configuration object
- `include_secrets`: Include fields marked as secrets (default: False)
- `exclude_none`: Exclude fields with None values (default: True)

**Returns:**
- Dictionary representation of configuration

#### save_config

```python
save_config(
    config: BaseModel,
    filepath: Union[str, Path],
    format: str = None,
    include_secrets: bool = False
) -> None
```

Save configuration to file.

**Parameters:**
- `config`: Configuration object to save
- `filepath`: Path where to save the configuration
- `format`: File format (yaml, json, toml). If None, inferred from extension
- `include_secrets`: Include secret fields in saved file (default: False)

## ConfigField

Enhanced Pydantic field for configuration management.

```python
from config_manager import ConfigField
```

### Function Signature

```python
ConfigField(
    default: Any = PydanticUndefined,
    *,
    env: str = None,
    secret: bool = False,
    description: str = None,
    **kwargs
)
```

**Parameters:**
- `default`: Default value for the field
- `env`: Environment variable name to read value from
- `secret`: Mark field as containing sensitive data
- `description`: Field description
- `**kwargs`: All other Pydantic Field parameters

**Example:**
```python
class Config(BaseModel):
    api_key: str = ConfigField(env="API_KEY", secret=True)
    port: int = ConfigField(default=8000, env="PORT", description="Server port")
```

## Configuration Classes

### BaseConfigModel

Base class for configuration models with additional features.

```python
from config_manager import BaseConfigModel
```

**Features:**
- Automatic environment variable loading
- Nested configuration support
- Custom validation methods

**Example:**
```python
class MyConfig(BaseConfigModel):
    app_name: str
    debug: bool = False
    
    class Config:
        env_prefix = "MYAPP_"  # Prefix for environment variables
```

## Validators

### Built-in Validators

#### validate_url

```python
from config_manager.validators import validate_url

class Config(BaseModel):
    api_url: str = ConfigField()
    
    @validator("api_url")
    def validate_api_url(cls, v):
        return validate_url(v)
```

#### validate_port

```python
from config_manager.validators import validate_port

class Config(BaseModel):
    port: int = ConfigField(default=8000)
    
    @validator("port")
    def validate_port_number(cls, v):
        return validate_port(v)
```

#### validate_email

```python
from config_manager.validators import validate_email

class Config(BaseModel):
    admin_email: str = ConfigField()
    
    @validator("admin_email")
    def validate_admin_email(cls, v):
        return validate_email(v)
```

### Custom Validators

You can create custom validators using Pydantic's validator decorator:

```python
from pydantic import validator

class Config(BaseModel):
    percentage: float = ConfigField()
    
    @validator("percentage")
    def validate_percentage(cls, v):
        if not 0 <= v <= 100:
            raise ValueError("Percentage must be between 0 and 100")
        return v
```

## Loaders

### FileLoader

Base class for file loaders.

```python
from config_manager.loaders import FileLoader
```

### YAMLLoader

Load configuration from YAML files.

```python
from config_manager.loaders import YAMLLoader

loader = YAMLLoader()
config_dict = loader.load("config.yaml")
```

### JSONLoader

Load configuration from JSON files.

```python
from config_manager.loaders import JSONLoader

loader = JSONLoader()
config_dict = loader.load("config.json")
```

### TOMLLoader

Load configuration from TOML files.

```python
from config_manager.loaders import TOMLLoader

loader = TOMLLoader()
config_dict = loader.load("config.toml")
```

### EnvLoader

Load configuration from environment files.

```python
from config_manager.loaders import EnvLoader

loader = EnvLoader()
config_dict = loader.load(".env")
```

## Exceptions

### ConfigurationError

Base exception for configuration errors.

```python
from config_manager import ConfigurationError

try:
    config = config_manager.load_config(["config.yaml"])
except ConfigurationError as e:
    print(f"Configuration error: {e}")
```

### ValidationError

Raised when configuration validation fails.

```python
from config_manager import ValidationError

try:
    config = config_manager.validate_config(config_obj)
except ValidationError as e:
    print(f"Validation error: {e}")
```

## Environment Variable Resolution

Environment variables are resolved in the following order:

1. Direct environment variable (if `env` is specified in ConfigField)
2. Prefixed environment variable (if `env_prefix` is set in Config class)
3. Default value
4. Required field validation

**Example:**
```python
class DatabaseConfig(BaseModel):
    host: str = ConfigField(env="DB_HOST", default="localhost")
    port: int = ConfigField(env="DB_PORT", default=5432)
    
    class Config:
        env_prefix = "MYAPP_"

# These environment variables will be checked:
# 1. DB_HOST (direct)
# 2. MYAPP_DB_HOST (prefixed)
# 3. "localhost" (default)
```

## Type Conversions

Config Manager automatically converts string values from environment variables and config files to the appropriate Python types:

- `bool`: "true", "1", "yes", "on" → True; "false", "0", "no", "off" → False
- `int`: Numeric strings → integers
- `float`: Numeric strings → floats
- `list`: Comma-separated values → list
- `dict`: JSON strings → dictionaries

**Example:**
```python
# Environment: DEBUG=true PORT=8080 FEATURES=auth,api,admin

class Config(BaseModel):
    debug: bool = ConfigField(env="DEBUG")  # True
    port: int = ConfigField(env="PORT")     # 8080
    features: List[str] = ConfigField(env="FEATURES")  # ["auth", "api", "admin"]
```