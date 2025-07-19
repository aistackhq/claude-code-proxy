# @aistack/claude-code-proxy

A high-performance proxy server that translates Anthropic Claude API requests to LiteLLM format, enabling seamless integration with multiple LLM providers including OpenAI, Google Gemini, OpenRouter, and Anthropic.

## Features

- 🚀 **Multi-Provider Support**: Seamlessly switch between OpenAI, Gemini, OpenRouter, and Anthropic models
- 🔄 **Automatic Format Translation**: Converts Anthropic API format to LiteLLM format and back
- 🎯 **Smart Model Mapping**: Intelligently maps Claude models to equivalent models from other providers
- 📡 **Streaming Support**: Full support for Server-Sent Events (SSE) streaming responses
- 🛡️ **Production Ready**: Comprehensive error handling, logging, and graceful shutdown
- ⚡ **High Performance**: Built with Go for optimal performance and low resource usage
- 🔧 **Configurable**: Flexible configuration via settings.json (Simialr to Claude Code)

## Installation

Install globally using npm:

```bash
npm install -g @aistack/claude-code-proxy # OR
npx @aistack/claude-code-proxy
```

OR 

```bash
npx ccproxy
bunx ccproxy
``` 

## Quick Start

1. **Initialize configuration**:

```bash
ccproxy init
```

This will prompt you to create a configuration file at either:
- User level: `$HOME/.ccproxy/settings.json`
- Project level: `./.ccproxy/settings.json`

2. **Edit the configuration file**:

```json
{
  "server": {
    "port": 8082,
    "host": "0.0.0.0"
  },
  "providers": {
    "openai": {
      "apiKey": "sk-...",
      "baseURL": "https://api.openai.com/v1",
      "responseStyle": "openai"
    },
    "anthropic": {
      "apiKey": "sk-ant-...",
      "baseURL": "https://api.anthropic.com/v1",
      "responseStyle": "anthropic"
    },
    "gemini": {
      "apiKey": "...",
      "baseURL": "https://generativelanguage.googleapis.com/v1beta/openai",
      "responseStyle": "openai"
    },
    "openrouter": {
      "apiKey": "sk-or-v1-...",
      "baseURL": "https://openrouter.ai/api/v1",
      "responseStyle": "openai"
    }
  },
  "models": {
    "bigModel": "moonshotai/kimi-k2@openrouter",
    "smallModel": "gpt-4.1@openai"
  },
  "logging": {
    "level": "info",
    "format": "json"
  },
  "claude": {
    "args": ""
  }
}
```

3. **Start the proxy**:

```bash
# Start proxy server and launch claude CLI (default behavior)
ccproxy

# Or start only the proxy server
ccproxy server
```

The server will start on `http://localhost:8082` (or your configured port).

## Commands

### Available Commands

- **`ccproxy`** (default) - Start proxy server (if needed) and launch claude CLI
- **`ccproxy server`** - Start only the proxy server
- **`ccproxy init`** - Initialize configuration file interactively
  - `--scope user` - Create user-level configuration
  - `--scope project` - Create project-level configuration
- **`ccproxy help`** - Show help message
- **`ccproxy version`** - Show version information

### Global Options

- **`--debug`** - Enable debug logging (applies to all commands)
- **`--port`** - Override server port (server command only)

### Examples

```bash
# Start server and launch claude CLI (default behavior)
ccproxy

# Start with debug logging enabled
ccproxy --debug

# Start only the proxy server
ccproxy server

# Start server on custom port
ccproxy server --port 8080

# Start server with debug logging
ccproxy server --debug

# Initialize configuration interactively
ccproxy init

# Create user-level configuration
ccproxy init --scope user

# Create project-level configuration
ccproxy init --scope project
```

## Usage

## Configuration

### Configuration Files

The proxy loads configuration from multiple sources in priority order:

1. **Command Line Arguments** (highest priority)
   - `--port`: Override server port
2. **JSON Configuration Files**:
   - `/etc/ccproxy/managed-settings.json` (managed/system-wide)
   - `$CWD/.ccproxy/settings.json` (project-level)
   - `$HOME/.ccproxy/settings.json` (user-level)

### Model Format

Models are specified in the format `modelname@provider`:
- `gpt-4@openai`
- `claude-4-opus@anthropic`
- `gemini-2.5-pro@gemini`
- `moonshotai/kimi-k2@openrouter`

### Response Styles

Providers can have different response styles:
- `"responseStyle": "anthropic"` - Native Anthropic format (no transformation)
- `"responseStyle": "openai"` - OpenAI format (requires transformation)

This allows the proxy to skip unnecessary transformations for providers that natively support Anthropic's API format.

## Model Mapping

The proxy automatically maps Claude models to configured models:

| Claude Model | Mapped To | Description |
|--------------|-----------|-------------|
| claude-*-haiku-* | `smallModel` | Maps to the configured small model |
| claude-*-sonnet-* | `bigModel` | Maps to the configured big model |
| Other models | As specified | Uses the model@provider format |

## Configuration Options

### JSON Configuration

| Field | Description | Default |
|-------|-------------|---------|
| `server.port` | Server port | `8082` |
| `server.host` | Server host | `0.0.0.0` |
| `providers.<name>.apiKey` | Provider API key | - |
| `providers.<name>.baseURL` | Provider base URL | - |
| `providers.<name>.responseStyle` | Response format style | `openai` |
| `models.bigModel` | Model for large requests | `gpt-4@openai` |
| `models.smallModel` | Model for small requests | `gpt-4-mini@openai` |
| `logging.level` | Log level | `info` |
| `logging.format` | Log format | `json` |


## Default Behavior

When you run `ccproxy` without arguments, it will:
1. Check if the proxy server is already running
2. Start the server if it's not running
3. Launch the claude CLI with ANTHROPIC_BASE_URL set to the proxy

This means you can use `ccproxy` as a drop-in replacement for the `claude` command while automatically getting multi-provider support.

### Adding Custom Providers

You can add custom providers with native Anthropic support:

```json
{
  "providers": {
    "custom-llm": {
      "baseURL": "https://your-llm-api.com",
      "apiKey": "your-api-key",
      "responseStyle": "anthropic"
    }
  }
}
```

Then use it with: `custom-model@custom-llm`

## GitHub Actions Integration

Use claude-code-proxy in your GitHub Actions workflows to enable multi-provider LLM support for automated code analysis, reviews, and assistance.

### Quick Setup

```yaml
- name: Setup Claude Code Proxy
  uses: ./.github/actions/setup-ccproxy
  with:
    openai_api_key: ${{ secrets.OPENAI_API_KEY }}
    big_model: 'gpt-4@openai'
    small_model: 'gpt-4-mini@openai'

- name: Run Claude Code
  uses: anthropics/claude-code-action@beta
  env:
    ANTHROPIC_BASE_URL: http://localhost:8082
    ANTHROPIC_API_KEY: dummy-key
  with:
    task: 'Review this code and suggest improvements'
```

### Key Features
- 🔄 **Multi-Provider Support**: Switch between OpenAI, Anthropic, Gemini, OpenRouter
- 🚀 **Easy Setup**: One-step proxy configuration with secrets management
- 🔧 **Flexible**: Support for issue comments, PR reviews, scheduled analysis
- 📊 **Cost Optimization**: Route different tasks to appropriate model sizes
- 🛡️ **Secure**: Proper API key management via GitHub Secrets

See the [GitHub Actions Documentation](docs/github-actions.md) for complete setup instructions, examples, and best practices.

## Platform Support

The CLI is available for the following platforms:

- macOS (Intel & Apple Silicon)
- Linux (x64 & ARM64)
- Windows (x64)

## Troubleshooting

### Binary Not Found

If you see "Binary not found" error, it means your platform is not supported. Please [open an issue](https://github.com/aistackhq/claude-code-proxy/issues) with your platform details.

### Permission Denied

On Unix systems, if you get a permission error:

```bash
sudo npm install -g @aistack/claude-code-proxy
```

### API Key Issues

Ensure your API keys are correctly set in your `settings.json` file and that they have the necessary permissions.

## Development

For development or contributing, visit the [GitHub repository](https://github.com/aistackhq/claude-code-proxy).

## License

MIT License - see the [LICENSE](https://github.com/aistackhq/claude-code-proxy/blob/main/LICENSE) file for details.

## Support

- 📚 [Documentation](https://github.com/aistackhq/claude-code-proxy)
- 🐛 [Issue Tracker](https://github.com/aistackhq/claude-code-proxy/issues)
