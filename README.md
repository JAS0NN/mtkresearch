# MTK Research Package

# Setup

```
$ pip install mtkresearch
```

# MRPromptV3

`MRPromptV3` is a class designed to facilitate the creation and management of prompts for conversational AI systems. This class provides methods to generate prompts based on a series of conversations, including support for function calls and handling various types of user inputs such as text, images, and categories.

## Initialization
To initialize an instance of `MRPromptV3`, simply import the class and create an instance:

```python
from mtkresearch.llm.prompt import MRPromptV3
prompt = MRPromptV3()
```

## Methods

### `get_prompt`

Generates a prompt based on the provided conversations and optional functions.

```markdown
Parameters:
* `conversations` (list): A list of conversation dictionaries, each containing role and content.
* `functions` (list, optional): A list of function definitions that the assistant can use. Default is `None`.
* `add_bos_token` (bool, optional): Whether to add a beginning-of-sequence token. Default is `False`.
* `training` (bool, optional): Whether the prompt is for training purposes. Default is `False`.

Returns:
* `str`: The generated prompt.
```

Example:
```python
python
conversations = [
    {"role": "user", "content": "What is the weather of Boston?"}
]
functions = [
    {
        'name': 'get_current_weather',
        'description': 'Get the current weather',
        'parameters': {
            'type': 'object',
            'properties': {
                'location': {'type': 'string', 'description': 'The city and state, e.g. San Francisco, CA'},
                'unit': {'type': 'string', 'enum': ['celsius', 'fahrenheit']}
            },
            'required': ['location']
        }
    }
]

result = prompt.get_prompt(conversations, functions)
print(result)
```

### `parse_generated_str`

Parses a generated string to extract the role and content, including function calls.

```markdown
Parameters:
* `generated_str` (str): The generated string to parse.

Returns:
* `dict`: A dictionary containing the parsed role and content.
```

Example:

```python
generated_str = "<|use_tool|>[get_current_weather(location='Boston, MA')]<|eot_id|>"
parsed_result = prompt.parse_generated_str(generated_str)
print(parsed_result)
# Expected: 
# {
#     'role': 'assistant',
#     'tool_calls': [
#         {
#             'type': 'function',
#             'function': {
#                 'name': 'get_current_weather',
#                 'arguments': '{"location": "Boston, MA"}'
#             }
#         }
#     ]
# }
```
