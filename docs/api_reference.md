# API Reference

## ConfigManager

The main class for managing application configurations.

```python
from config_manager import ConfigManager
```

### Constructor

```python
ConfigManager(
    config_paths=None,
    env_prefix=None,
    schema=None,
    auto_load=True
)
```

**Parameters:**
- `config_paths`: List of configuration file paths to load (optional)
- `env_prefix`: Prefix for environment variables (optional)
- `schema`: ConfigSchema instance defining the configuration structure (optional)
- `auto_load`: Whether to automatically load configuration on initialization (default: True)

### Methods

#### load

```python
load() -> dict
```

Load configuration from all configured sources (files and environment variables).

**Returns:**
- Dictionary containing the merged configuration

**Example:**
```python
config = config_manager.load()
```

#### get

```python
get(key: str, default=None) -> Any
```

Get a configuration value by key. Supports dot notation for nested values.

**Parameters:**
- `key`: Configuration key (supports dot notation like "database.host")
- `default`: Default value if key is not found

**Returns:**
- Configuration value or default

**Example:**
```python
debug = config_manager.get("debug", False)
db_host = config_manager.get("database.host", "localhost")
```

#### set

```python
set(key: str, value: Any, source: str = "runtime") -> None
```

Set a configuration value by key. Supports dot notation for nested values.

**Parameters:**
- `key`: Configuration key (supports dot notation)
- `value`: Value to set
- `source`: Source identifier for tracking (default: "runtime")

**Example:**
```python
config_manager.set("debug", True)
config_manager.set("database.host", "prod-db.example.com")
config_manager.set("api_key", "secret123", source="manual_override")
```

#### has

```python
has(key: str) -> bool
```

Check if a configuration key exists.

**Parameters:**
- `key`: Configuration key (supports dot notation)

**Returns:**
- True if key exists, False otherwise

**Example:**
```python
if config_manager.has("feature_flags.new_feature"):
    print("New feature is enabled")
```

#### get_source

```python
get_source(key: str) -> Optional[str]
```

Get the source where a configuration value was loaded from.

**Parameters:**
- `key`: Configuration key

**Returns:**
- Source identifier (e.g., "config.toml", "environment", "schema_default") or None if key doesn't exist

**Example:**
```python
source = config_manager.get_source("database.host")
if source:
    print(f"database.host was loaded from: {source}")
else:
    print("database.host not found")
```

#### to_dict

```python
to_dict() -> Dict[str, Any]
```

Convert the entire configuration to a dictionary.

**Returns:**
- Dictionary representation of the configuration

#### add_config_path

```python
add_config_path(path: Union[str, Path]) -> None
```

Add a configuration file path to the search list.

**Parameters:**
- `path`: Path to configuration file to add

**Example:**
```python
config_manager.add_config_path("additional_config.toml")
config_manager.add_config_path(Path("configs/override.toml"))
```

## ConfigSchema

Class for defining configuration schemas with type validation and defaults.

```python
from config_manager import ConfigSchema
```

### Constructor

```python
ConfigSchema()
```

### Methods

#### add_field

```python
add_field(
    name: str,
    field_type: type,
    default=None,
    required: bool = False
) -> None
```

Add a field to the schema.

**Parameters:**
- `name`: Field name
- `field_type`: Python type for the field (str, int, bool, list, dict, etc.)
- `default`: Default value for the field
- `required`: Whether the field is required

**Example:**
```python
schema = ConfigSchema()
schema.add_field("debug", bool, default=False)
schema.add_field("port", int, default=8080)
schema.add_field("api_key", str, required=True)
```

#### add_nested_schema

```python
add_nested_schema(name: str, nested_schema: ConfigSchema) -> None
```

Add a nested schema for complex configuration structures.

**Parameters:**
- `name`: Name of the nested section
- `nested_schema`: ConfigSchema instance for the nested section

**Example:**
```python
db_schema = ConfigSchema()
db_schema.add_field("host", str, default="localhost")
db_schema.add_field("port", int, default=5432)

main_schema = ConfigSchema()
main_schema.add_nested_schema("database", db_schema)
```

## Loaders

### TOMLLoader

Load configuration from TOML files.

```python
from config_manager import TOMLLoader

loader = TOMLLoader()
config_dict = loader.load("config.toml")
```

**Methods:**
- `load(file_path: Path) -> Dict[str, Any]`: Load configuration from a TOML file

### EnvLoader

Load configuration from environment variables.

```python
from config_manager import EnvLoader

loader = EnvLoader(prefix="MYAPP")
config_dict = loader.load()
```

**Constructor Parameters:**
- `prefix`: Environment variable prefix (optional)

**Methods:**
- `load() -> dict`: Load configuration from environment variables

## Validators

### TypeValidator

Validates and converts values to specified types.

```python
from config_manager import TypeValidator

validator = TypeValidator()
value = validator.validate_type("8080", int)  # Returns 8080 as int
```

**Methods:**
- `validate_type(value: Any, expected_type: Type) -> Any`: Convert value to expected type

### SchemaValidator

Validates configuration against a schema.

```python
from config_manager import SchemaValidator

validator = SchemaValidator()
validated_config = validator.validate(config_dict, schema)
```

**Constructor Parameters:**
- None (parameterless constructor)

**Methods:**
- `validate(config: Dict[str, Any], schema: ConfigSchema) -> Dict[str, Any]`: Validate configuration against schema and return validated config

## Exceptions

### ConfigError

Base exception for configuration errors.

```python
from config_manager import ConfigError

try:
    config = config_manager.load()
except ConfigError as e:
    print(f"Configuration error: {e}")
```

## Environment Variable Resolution

Environment variables are resolved with the following naming convention:

1. For top-level fields: `{PREFIX}_{FIELD_NAME}`
2. For nested fields: `{PREFIX}_{SECTION}__{FIELD_NAME}`

**Example:**
```python
# With prefix "MYAPP"
# config.debug -> MYAPP_DEBUG
# config.database.host -> MYAPP_DATABASE__HOST
# config.database.port -> MYAPP_DATABASE__PORT
```

## Type Conversions

The TypeValidator automatically converts string values from environment variables and config files:

- `bool`: "true", "1", "yes", "on" → True; "false", "0", "no", "off" → False
- `int`: Numeric strings → integers
- `float`: Numeric strings → floats
- `list`: Comma-separated values → list
- `dict`: Not supported via environment variables

**Example:**
```bash
# Environment variables
export MYAPP_DEBUG=true
export MYAPP_PORT=8080
export MYAPP_ALLOWED_HOSTS=localhost,example.com
```

## File Format Support

Currently supported configuration file formats:

- **TOML**: `.toml` files using the TOMLLoader
- **Environment**: Environment variables using the EnvLoader

**Note**: YAML and JSON loaders are not currently implemented.