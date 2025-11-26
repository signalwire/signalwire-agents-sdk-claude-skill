# Troubleshooting Guide

Common issues and solutions for SignalWire AI Agents.

## Quick Diagnostics

Run these commands first to diagnose issues:

```bash
# Verify SWML configuration
swaig-test my_agent.py --dump-swml

# List registered functions
swaig-test my_agent.py --list-tools

# Test a specific function
swaig-test my_agent.py --exec function_name --args '{"param": "value"}'

# Check for Python errors
python my_agent.py 2>&1 | head -50
```

## Common Issues

### Agent Doesn't Speak

**Symptom:** Agent answers but produces no audio.

**Cause:** No language/voice configured.

**Solution:**
```python
# Add language configuration
self.add_language("English", "en-US", "rime.spore")
```

**Verify:**
```bash
swaig-test my_agent.py --dump-swml | grep -A5 "languages"
```

---

### Functions Never Called

**Symptom:** AI doesn't call your SWAIG functions even when appropriate.

**Cause:** Function description is unclear.

**Solution:** Make descriptions explicit about when to use:

```python
# Bad - vague description
@AgentBase.tool(
    name="lookup",
    description="Look up data",  # Too vague
    parameters={}
)

# Good - clear description
@AgentBase.tool(
    name="lookup_order",
    description="Look up order status and tracking information using the order number",
    parameters={}
)
```

**Also check:**
1. Function is registered: `swaig-test --list-tools`
2. Prompt mentions the function
3. Parameters have clear descriptions

---

### Import Errors

**Symptom:** `ModuleNotFoundError: No module named 'signalwire_agents'`

**Solution:**
```bash
# Install the SDK
pip install signalwire-agents

# Or install from source
pip install -e /path/to/signalwire-agents
```

**Verify:**
```bash
python -c "from signalwire_agents import AgentBase; print('OK')"
```

---

### Port Already in Use

**Symptom:** `OSError: [Errno 48] Address already in use`

**Solutions:**

1. Use a different port:
```python
agent = MyAgent()
agent.port = 3001  # Or pass in constructor
agent.run()
```

2. Kill the existing process:
```bash
lsof -i :3000 | grep LISTEN
kill -9 <PID>
```

3. Use environment variable:
```bash
PORT=3001 python my_agent.py
```

---

### Transfer Fails

**Symptom:** Transfer action doesn't work or produces errors.

**Cause:** Invalid destination format.

**Solution:** Use correct URI format:

```python
# Phone number - must include country code
.add_action("transfer", {"dest": "tel:+15551234567"})

# SIP address
.add_action("transfer", {"dest": "sip:user@domain.com"})

# SignalWire resource
.add_action("transfer", {"dest": "sip:queue-name@company.sip.signalwire.com"})
```

**Wrong formats:**
```python
# Missing country code
{"dest": "tel:5551234567"}  # Wrong

# Missing protocol
{"dest": "user@domain.com"}  # Wrong

# Wrong protocol
{"dest": "http://domain.com"}  # Wrong
```

---

### Parameters Not Received

**Symptom:** Function handler receives empty or wrong parameters.

**Cause:** JSON Schema format is incorrect.

**Solution:** Use correct schema format:

```python
# Correct format
parameters={
    "order_number": {
        "type": "string",
        "description": "The order number to look up"
    },
    "include_items": {
        "type": "boolean",
        "description": "Include line items",
        "default": False
    }
}

# Wrong - missing type
parameters={
    "order_number": {
        "description": "Order number"  # Missing "type"
    }
}

# Wrong - wrong structure
parameters={
    "order_number": "string"  # Should be dict
}
```

---

### Skill Not Loading

**Symptom:** Skill functions not available.

**Cause:** Missing environment variables or wrong parameters.

**Solution:** Check skill requirements:

```python
# Check if skill loaded
if agent.has_skill("web_search"):
    print("Web search available")
else:
    print("Web search not loaded - check env vars")

# Verify environment
import os
print(os.getenv("GOOGLE_API_KEY"))
print(os.getenv("GOOGLE_SEARCH_ENGINE_ID"))
```

---

### SignalWire Can't Reach Agent

**Symptom:** Calls don't connect or timeout.

**Causes and solutions:**

1. **Agent not running:**
   ```bash
   curl -X POST http://localhost:3000/ -d '{}' -H "Content-Type: application/json"
   ```

2. **No public URL:** Use ngrok for development:
   ```bash
   ngrok http 3000
   ```

3. **Firewall blocking:** Check firewall allows port 3000

4. **Wrong URL in SignalWire:** Verify the webhook URL in SignalWire dashboard

5. **Basic auth mismatch:** Check credentials match:
   ```bash
   curl -u user:pass https://your-agent.com/
   ```

---

### Handler Returns Wrong Type

**Symptom:** `TypeError` or function produces no response.

**Cause:** Not returning `SwaigFunctionResult`.

**Solution:**
```python
# Wrong - returning string
def my_function(self, args, raw_data):
    return "Hello"  # Wrong!

# Wrong - returning dict
def my_function(self, args, raw_data):
    return {"response": "Hello"}  # Wrong!

# Correct
def my_function(self, args, raw_data):
    return SwaigFunctionResult("Hello")
```

---

### Post-Process Not Working

**Symptom:** Hangup occurs before AI finishes speaking.

**Cause:** Missing `post_process=True`.

**Solution:**
```python
# Wrong - hangup interrupts speech
return SwaigFunctionResult("Goodbye!").add_action("hangup", {})

# Correct - AI finishes speaking first
return SwaigFunctionResult(
    "Goodbye!",
    post_process=True
).add_action("hangup", {})
```

---

### Debug Webhook Not Receiving Events

**Symptom:** No events at debug webhook URL.

**Solutions:**

1. Check debug level is set:
   ```python
   self.set_params({
       "debug_webhook_url": "https://webhook.site/your-id",
       "debug_webhook_level": 1  # Must be 1 or 2
   })
   ```

2. Verify URL is accessible:
   ```bash
   curl -X POST https://webhook.site/your-id -d '{"test": true}'
   ```

3. Check environment variables:
   ```bash
   echo $DEBUG_WEBHOOK_URL
   echo $DEBUG_WEBHOOK_LEVEL
   ```

---

### SWML Too Large

**Symptom:** Agent fails to load or responses are slow.

**Cause:** Prompt or function definitions too verbose.

**Solutions:**

1. Trim prompt sections
2. Remove unnecessary functions
3. Use DataMap for simple API calls (handled server-side)
4. Split into multiple specialized agents

---

## Environment Variable Issues

### Check All Required Variables

```python
import os

required = [
    "SIGNALWIRE_SPACE_NAME",
    "SIGNALWIRE_PROJECT_ID",
    "SIGNALWIRE_TOKEN"
]

for var in required:
    value = os.getenv(var)
    if value:
        print(f"{var}: {'*' * 8}...{value[-4:]}")
    else:
        print(f"{var}: NOT SET")
```

### Load from .env File

```python
from dotenv import load_dotenv
load_dotenv()  # Load from .env file

from signalwire_agents import AgentBase
# Now env vars are available
```

---

## Debugging Techniques

### Enable Verbose Logging

```bash
SWML_LOG_LEVEL=DEBUG python my_agent.py
```

### Print SWML on Startup

```python
if __name__ == "__main__":
    agent = MyAgent()

    # Print SWML for debugging
    import json
    print(json.dumps(agent.render_swml(), indent=2))

    agent.run()
```

### Inspect Function Calls

```python
def on_swml_request(self, request_data=None, callback_path=None, request=None):
    print(f"Request: {request_data}")
    print(f"Path: {callback_path}")
```

### Test Functions in Isolation

```python
# test_functions.py
from my_agent import MyAgent

agent = MyAgent()

# Test function directly
result = agent.lookup_order(
    {"order_number": "ORD-12345"},
    {"call_id": "test"}
)
print(result.to_dict())
```

---

## Getting Help

1. **Check SWML output:** `swaig-test --dump-swml`
2. **Verify functions:** `swaig-test --list-tools`
3. **Enable debug logging:** `SWML_LOG_LEVEL=DEBUG`
4. **Use debug webhooks:** Set `debug_webhook_level: 2`
5. **Test locally first:** Use ngrok before deploying
6. **Check SignalWire logs:** Dashboard > Logs

## Common Fixes Summary

| Issue | Quick Fix |
|-------|-----------|
| No speech | Add `add_language()` |
| Function not called | Improve description |
| Import error | `pip install signalwire-agents` |
| Port in use | Change port or kill process |
| Transfer fails | Use `tel:+1...` or `sip:...` format |
| Bad parameters | Check JSON Schema format |
| Skill missing | Check environment variables |
| Can't connect | Check public URL and firewall |
| Wrong return type | Return `SwaigFunctionResult` |
| Hangup interrupts | Use `post_process=True` |
