# Serverless Deployment Reference

Deploy SignalWire AI Agents in serverless environments including AWS Lambda, Google Cloud Functions, Azure Functions, and CGI.

## Import

```python
from signalwire_agents import AgentBase
```

## Automatic Mode Detection

The SDK automatically detects the execution environment:

| Environment | Detection Method |
|-------------|------------------|
| AWS Lambda | `AWS_LAMBDA_FUNCTION_NAME` env var |
| Google Cloud Functions | `FUNCTION_TARGET` env var |
| Azure Functions | `FUNCTIONS_WORKER_RUNTIME` env var |
| CGI | `GATEWAY_INTERFACE` env var |
| HTTP Server | Default (no env vars) |

## Usage Methods

### Method 1: run() with Auto-Detection

The simplest approach - let the SDK detect the environment:

```python
if __name__ == "__main__":
    agent = MyAgent()
    agent.run()  # Automatically uses serverless mode if detected
```

### Method 2: run() with Force Mode

Force a specific execution mode:

```python
agent.run(force_mode="lambda")    # Force Lambda mode
agent.run(force_mode="cgi")       # Force CGI mode
agent.run(force_mode="google_cloud_function")  # Force GCF mode
agent.run(force_mode="azure_function")  # Force Azure mode
```

### Method 3: handle_serverless_request()

Directly handle serverless requests:

```python
def handle_serverless_request(
    self,
    event=None,      # Lambda/Cloud Function event
    context=None,    # Lambda/Cloud Function context
    mode=None        # Override execution mode
) -> Any
```

---

## AWS Lambda

### Lambda Handler

```python
# agent.py
from signalwire_agents import AgentBase
from signalwire_agents.core.function_result import SwaigFunctionResult


class MyAgent(AgentBase):
    def __init__(self):
        super().__init__(name="lambda-agent")
        self.add_language("English", "en-US", "rime.spore")
        self.prompt_add_section("Role", "You are a helpful assistant.")

    @AgentBase.tool(
        name="get_info",
        description="Get information",
        parameters={"query": {"type": "string"}}
    )
    def get_info(self, args, raw_data):
        return SwaigFunctionResult(f"Info for: {args.get('query')}")


# Create singleton agent instance
agent = MyAgent()


def lambda_handler(event, context):
    """AWS Lambda entry point."""
    return agent.handle_serverless_request(event, context, mode="lambda")
```

### Lambda Response Format

The SDK returns Lambda-compatible responses:

```python
{
    "statusCode": 200,
    "headers": {"Content-Type": "application/json"},
    "body": "..."  # JSON string
}
```

### Lambda Configuration

**API Gateway Setup:**
- Use proxy integration (`{proxy+}`)
- Enable CORS if needed
- Set appropriate timeout (30s+ recommended)

**Environment Variables:**
```
SIGNALWIRE_SPACE_NAME=your-space
SIGNALWIRE_PROJECT_ID=your-project-id
SIGNALWIRE_TOKEN=your-token
SWML_PROXY_URL_BASE=https://your-api-gateway-url.execute-api.region.amazonaws.com/stage
```

**Authentication:**
```python
agent = MyAgent(basic_auth=("user", "password"))
```

---

## Google Cloud Functions

### Cloud Function Handler

```python
# main.py
from signalwire_agents import AgentBase


class MyAgent(AgentBase):
    def __init__(self):
        super().__init__(name="gcf-agent")
        self.add_language("English", "en-US", "rime.spore")
        self.prompt_add_section("Role", "You are a helpful assistant.")


# Create singleton agent instance
agent = MyAgent()


def agent_handler(request):
    """Google Cloud Function entry point.

    Args:
        request: Flask request object

    Returns:
        Flask response
    """
    return agent.handle_serverless_request(request, mode="google_cloud_function")
```

### Deployment

```bash
gcloud functions deploy agent_handler \
    --runtime python39 \
    --trigger-http \
    --allow-unauthenticated \
    --timeout 300
```

### Environment Variables

Set in Cloud Console or `env.yaml`:

```yaml
SIGNALWIRE_SPACE_NAME: your-space
SIGNALWIRE_PROJECT_ID: your-project-id
SIGNALWIRE_TOKEN: your-token
SWML_PROXY_URL_BASE: https://your-region-your-project.cloudfunctions.net/agent_handler
```

---

## Azure Functions

### Azure Function Handler

```python
# __init__.py
import azure.functions as func
from signalwire_agents import AgentBase


class MyAgent(AgentBase):
    def __init__(self):
        super().__init__(name="azure-agent")
        self.add_language("English", "en-US", "rime.spore")
        self.prompt_add_section("Role", "You are a helpful assistant.")


# Create singleton agent instance
agent = MyAgent()


def main(req: func.HttpRequest) -> func.HttpResponse:
    """Azure Function entry point."""
    return agent.handle_serverless_request(req, mode="azure_function")
```

### function.json

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get", "post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    }
  ]
}
```

### Environment Variables

Set in Application Settings:

```
SIGNALWIRE_SPACE_NAME=your-space
SIGNALWIRE_PROJECT_ID=your-project-id
SIGNALWIRE_TOKEN=your-token
SWML_PROXY_URL_BASE=https://your-function.azurewebsites.net/api/agent
```

---

## CGI Mode

For traditional CGI deployments:

```python
#!/usr/bin/env python3
from signalwire_agents import AgentBase


class MyAgent(AgentBase):
    def __init__(self):
        super().__init__(name="cgi-agent")
        self.add_language("English", "en-US", "rime.spore")
        self.prompt_add_section("Role", "You are a helpful assistant.")


if __name__ == "__main__":
    agent = MyAgent()
    agent.run(force_mode="cgi")
```

### CGI Configuration (Apache)

```apache
ScriptAlias /agent /var/www/cgi-bin/agent.py
<Directory /var/www/cgi-bin>
    Options +ExecCGI
    AddHandler cgi-script .py
</Directory>
```

---

## Authentication in Serverless

All serverless modes support basic authentication:

```python
agent = MyAgent(basic_auth=("username", "password"))
```

Authentication is checked automatically for each request. Failed auth returns:
- Lambda: `401` status with `WWW-Authenticate` header
- GCF/Azure: Flask response with `401` status
- CGI: Appropriate HTTP headers

---

## URL Configuration

### Proxy URL Base

Set the external URL where your function is accessible:

```python
# Via environment variable (recommended)
export SWML_PROXY_URL_BASE="https://your-function-url.com"

# Or programmatically
agent.manual_set_proxy_url("https://your-function-url.com")
```

This URL is used for SWAIG webhook callbacks.

---

## Request Flow

1. **Root request** (no path) → Returns SWML document
2. **Path request** (e.g., `/function_name`) → Executes SWAIG function

### SWML Endpoint

```
POST https://your-function-url.com/
```

Returns the SWML document for SignalWire to consume.

### SWAIG Endpoints

```
POST https://your-function-url.com/function_name
Body: {
    "function": "function_name",
    "argument": {"parsed": [{"param": "value"}]},
    "call_id": "..."
}
```

---

## Complete AWS Lambda Example

```python
# agent.py
import os
from signalwire_agents import AgentBase
from signalwire_agents.core.function_result import SwaigFunctionResult


class CustomerServiceAgent(AgentBase):
    """Customer service agent for Lambda deployment."""

    def __init__(self):
        super().__init__(
            name="customer-service",
            basic_auth=(
                os.environ.get("AGENT_USER", "agent"),
                os.environ.get("AGENT_PASSWORD", "secret")
            )
        )

        # Configure voice
        self.add_language("English", "en-US", "rime.spore")

        # Build prompt
        self.prompt_add_section("Role", "You are a customer service agent.")
        self.prompt_add_section(
            "Guidelines",
            bullets=[
                "Be helpful and professional",
                "Use lookup_order for order inquiries",
                "Transfer complex issues to human agents"
            ]
        )

        # Add skills
        self.add_skill("datetime", {"timezone": "America/New_York"})

        # Configure post-prompt
        self.set_post_prompt(json_schema={
            "type": "object",
            "properties": {
                "resolved": {"type": "boolean"},
                "summary": {"type": "string"}
            }
        })

    @AgentBase.tool(
        name="lookup_order",
        description="Look up an order by number",
        parameters={
            "order_number": {
                "type": "string",
                "description": "Order number to look up"
            }
        },
        fillers=["Let me check that order for you"]
    )
    def lookup_order(self, args, raw_data):
        order_num = args.get("order_number")
        # Query DynamoDB or other backend
        return SwaigFunctionResult(f"Order {order_num}: Shipped, arriving Friday")

    @AgentBase.tool(
        name="transfer_to_human",
        description="Transfer to a human agent",
        parameters={}
    )
    def transfer_to_human(self, args, raw_data):
        return (SwaigFunctionResult("Transferring to a specialist")
                .add_action("transfer", {"dest": "sip:support@company.com"}))


# Create singleton instance
agent = CustomerServiceAgent()


def lambda_handler(event, context):
    """AWS Lambda entry point."""
    return agent.handle_serverless_request(event, context, mode="lambda")
```

### SAM Template (template.yaml)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
  AgentFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: agent.lambda_handler
      Runtime: python3.9
      Timeout: 300
      MemorySize: 512
      Environment:
        Variables:
          SIGNALWIRE_SPACE_NAME: !Ref SignalWireSpace
          SIGNALWIRE_PROJECT_ID: !Ref SignalWireProjectId
          SIGNALWIRE_TOKEN: !Ref SignalWireToken
          SWML_PROXY_URL_BASE: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod"
          AGENT_USER: agent
          AGENT_PASSWORD: !Ref AgentPassword
      Events:
        Root:
          Type: Api
          Properties:
            Path: /
            Method: ANY
        Proxy:
          Type: Api
          Properties:
            Path: /{proxy+}
            Method: ANY
```

---

## Best Practices

1. **Use singleton pattern** - Create agent instance outside handler
2. **Set appropriate timeouts** - Voice calls need 60s+ for complex interactions
3. **Configure authentication** - Always use basic_auth in production
4. **Set SWML_PROXY_URL_BASE** - Required for SWAIG callbacks
5. **Handle cold starts** - Minimize initialization time
6. **Use environment variables** - Don't hardcode credentials

## See Also

- [Agent Base Reference](agent-base.md) - Core agent documentation
- [Environment Variables](environment-variables.md) - Configuration options
- [Dynamic Configuration](dynamic-configuration.md) - Per-request customization
