# Examples

## Basic Usage

### Simple Configuration

```python
from config_manager import ConfigManager, ConfigSchema

# Define your configuration schema
schema = ConfigSchema()
schema.add_field("debug", bool, default=False)
schema.add_field("host", str, default="localhost")
schema.add_field("port", int, default=8080)
schema.add_field("allowed_hosts", list, default=["localhost"])

# Create config manager
config_manager = ConfigManager(
    config_paths=["config.toml"],
    env_prefix="MYAPP",
    schema=schema
)

# Load configuration
config = config_manager.load()

# Access values using get method or dictionary access
print(f"Debug mode: {config_manager.get('debug')}")
print(f"Server running on {config_manager.get('host')}:{config_manager.get('port')}")

# Or access the loaded config dictionary directly
print(f"Debug mode: {config['debug']}")
print(f"Server running on {config['host']}:{config['port']}")
```

### Nested Configuration

```python
from config_manager import ConfigManager, ConfigSchema

# Define nested schemas
database_schema = ConfigSchema()
database_schema.add_field("host", str, default="localhost")
database_schema.add_field("port", int, default=5432)
database_schema.add_field("name", str, required=True)
database_schema.add_field("user", str, required=True)
database_schema.add_field("password", str, required=True)

# Main schema
schema = ConfigSchema()
schema.add_field("app_name", str, default="My App")
schema.add_nested_schema("database", database_schema)

# Create and load config
config_manager = ConfigManager(
    config_paths=["config.toml"],
    env_prefix="APP",
    schema=schema
)
config = config_manager.load()

# Access nested values using dot notation
db_host = config_manager.get("database.host")
db_port = config_manager.get("database.port")
db_user = config_manager.get("database.user")

# Or access nested values from the config dictionary
db_config = config['database']
connection_string = f"postgresql://{db_config['user']}@{db_config['host']}:{db_config['port']}/{db_config['name']}"
```

## Configuration File Examples

### TOML Configuration

```toml
# config.toml
debug = true
host = "0.0.0.0"
port = 8080
allowed_hosts = ["localhost", "example.com"]

[database]
host = "db.example.com"
port = 5432
name = "myapp"
user = "appuser"
password = "secretpassword"

[cache]
enabled = true
ttl = 300
backend = "redis"
```

### Environment Variables

```bash
# Override configuration with environment variables
export MYAPP_DEBUG=false
export MYAPP_PORT=9000
export MYAPP_DATABASE__HOST=prod-db.example.com
export MYAPP_DATABASE__PASSWORD=production-secret
export MYAPP_CACHE__ENABLED=true
```

## Advanced Usage

### Multiple Configuration Files

```python
# Load configuration from multiple TOML sources
config_manager = ConfigManager(
    config_paths=[
        "config/default.toml",    # Base configuration
        "config/production.toml", # Environment-specific overrides
    ],
    env_prefix="APP",
    schema=schema
)
```

**Note**: Only TOML files are currently supported. Environment files (.env) are not supported by the file loaders.

### Custom Validation

```python
from config_manager import ConfigManager, ConfigSchema, ConfigError

def validate_port(port):
    if not 1 <= port <= 65535:
        raise ConfigError(f"Port {port} is out of valid range (1-65535)")
    return port

schema = ConfigSchema()
schema.add_field("port", int, default=8080)

config_manager = ConfigManager(schema=schema)
config = config_manager.load()

# Custom validation after loading
try:
    validated_port = validate_port(config_manager.get("port"))
    config_manager.set("port", validated_port)
except ConfigError as e:
    print(f"Configuration error: {e}")
```

### Dynamic Configuration Updates

```python
# Update configuration at runtime
config_manager.set("debug", True)
config_manager.set("database.host", "new-db.example.com")

# Get configuration source
source = config_manager.get_source("database.host")
print(f"database.host was loaded from: {source}")

# Check if key exists
if config_manager.has("feature_flags.new_feature"):
    print("New feature is configured")
```

### Exporting Configuration

```python
# Get configuration as dictionary
config_dict = config_manager.to_dict()

# Save current configuration to file
import json
with open("current_config.json", "w") as f:
    json.dump(config_dict, f, indent=2)
```

## Integration Examples

### Flask Integration

```python
from flask import Flask
from config_manager import ConfigManager, ConfigSchema

# Define schema
schema = ConfigSchema()
schema.add_field("debug", bool, default=False)
schema.add_field("secret_key", str, required=True)
schema.add_field("database_uri", str, required=True)

# Load configuration
config_manager = ConfigManager(
    config_paths=["config.toml"],
    env_prefix="FLASK",
    schema=schema
)
config = config_manager.load()

# Create Flask app
app = Flask(__name__)

# Update Flask config from our config manager
app.config['DEBUG'] = config_manager.get('debug')
app.config['SECRET_KEY'] = config_manager.get('secret_key')
app.config['SQLALCHEMY_DATABASE_URI'] = config_manager.get('database_uri')

@app.route("/")
def home():
    return "Hello, World!"

@app.route("/config")
def show_config():
    # Show current configuration (excluding secrets)
    return {
        "debug": config_manager.get('debug'),
        "database_configured": config_manager.has('database_uri')
    }

if __name__ == "__main__":
    app.run(debug=config_manager.get('debug'))
```

### FastAPI Integration

```python
from fastapi import FastAPI
from config_manager import ConfigManager, ConfigSchema

# Define configuration schema
schema = ConfigSchema()
schema.add_field("app_name", str, default="FastAPI App")
schema.add_field("debug", bool, default=False)
schema.add_field("allowed_origins", list, default=["http://localhost:3000"])
schema.add_field("host", str, default="0.0.0.0")
schema.add_field("port", int, default=8000)

# Load configuration
config_manager = ConfigManager(
    config_paths=["config.toml"],
    env_prefix="API",
    schema=schema
)
config = config_manager.load()

# Create FastAPI app
app = FastAPI(
    title=config_manager.get('app_name'),
    debug=config_manager.get('debug')
)

# CORS middleware
from fastapi.middleware.cors import CORSMiddleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=config_manager.get('allowed_origins'),
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
def read_root():
    return {"message": f"Welcome to {config_manager.get('app_name')}"}

@app.get("/config")
def get_config():
    return {
        "app_name": config_manager.get('app_name'),
        "debug": config_manager.get('debug'),
        "host": config_manager.get('host'),
        "port": config_manager.get('port')
    }

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        app, 
        host=config_manager.get('host'), 
        port=config_manager.get('port'),
        debug=config_manager.get('debug')
    )
```

## Best Practices

1. **Use Schema Validation**: Always define a schema to ensure type safety and provide defaults
2. **Environment Variable Prefixes**: Use prefixes to avoid conflicts with system variables
3. **Layered Configuration**: Use multiple TOML files for different environments (base, dev, prod)
4. **Secure Secrets**: Never commit sensitive values to version control - use environment variables
5. **Document Configuration**: Include example configuration files and schemas in your project
6. **Validate Early**: Load and validate configuration at application startup
7. **Use Defaults**: Provide sensible defaults for optional configuration values
8. **Source Tracking**: Use `get_source()` to understand where configuration values originated
9. **Runtime Updates**: Use `set()` and `has()` for dynamic configuration changes
10. **Error Handling**: Always wrap configuration loading in try-catch blocks to handle ConfigError