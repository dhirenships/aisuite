# Reasoning Tokens Support in Langfuse Integration

This document describes the support for reasoning tokens in the aisuite Langfuse integration.

## Overview

Reasoning tokens are internal tokens used by AI models (e.g., OpenAI's o1, o3-mini) to perform complex reasoning tasks before generating a final response. These tokens contribute to the model's processing and are included in token usage for billing purposes but are not part of the visible output.

## Changes Made

The Langfuse integration in `aisuite/langfuse_integration.py` has been updated to properly extract and track reasoning tokens from LLM responses.

### New Method: `_extract_usage_data()`

This method extracts usage data from LLM responses, including:
- `input_tokens` (mapped from `prompt_tokens`)
- `output_tokens` (mapped from `completion_tokens`)
- `total_tokens`
- `input_tokens_details` (including `cached_tokens`)
- `output_tokens_details` (including `reasoning_tokens`)

### Updated Method: `update_trace_with_response()`

The trace update method now:
1. Extracts usage data including reasoning tokens using `_extract_usage_data()`
2. Adds usage information to the Langfuse metadata
3. Properly formats usage data to match OpenAI's API response format

## Usage Format

The integration now captures usage data in the following format:

```json
{
    "input_tokens": 75,
    "input_tokens_details": {
        "cached_tokens": 0
    },
    "output_tokens": 1186,
    "output_tokens_details": {
        "reasoning_tokens": 1024
    },
    "total_tokens": 1261
}
```

## API Response Formats Supported

The implementation handles multiple API response formats:

### OpenAI Format
- `input_tokens` / `prompt_tokens`
- `output_tokens` / `completion_tokens`
- `input_tokens_details` / `prompt_tokens_details`
- `output_tokens_details` / `completion_tokens_details`

### Pydantic Models
- Automatically converts Pydantic model objects to dictionaries
- Handles `CompletionUsage` and `CompletionTokensDetails` models

## Benefits

1. **Accurate Token Tracking**: Properly tracks all tokens including reasoning tokens
2. **Cost Monitoring**: Enables accurate cost calculations for reasoning-heavy models
3. **Performance Insights**: Provides insights into model reasoning behavior
4. **Compatibility**: Works with both OpenAI API responses and aisuite's internal models

## Example Usage

The reasoning tokens are automatically tracked when using any aisuite provider with Langfuse enabled:

```python
from aisuite import Client

client = Client(provider="openai")

response = client.chat.create(
    model="o3-mini",
    messages=[
        {"role": "user", "content": "Explain quantum computing"}
    ]
)

# Reasoning tokens are automatically tracked in Langfuse
# Check the Langfuse dashboard for detailed usage breakdown
```

## Monitoring

In the Langfuse dashboard, you can now see:
- Total tokens used
- Breakdown of input vs output tokens
- Number of reasoning tokens (when available)
- Cached tokens used
- Cost implications of reasoning tokens

## References

- [OpenAI Reasoning API Documentation](https://platform.openai.com/docs/guides/reasoning#managing-the-context-window)
- [OpenAI Token Details](https://platform.openai.com/docs/api-reference/chat/creating#chat/completions/usage)

