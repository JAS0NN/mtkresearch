## Special Tokens

- Inherited from llama3.2:
  - `<|begin_of_text|>`: Specifies the start of the prompt.
  - `<|end_of_text|>`: Model will cease to generate more tokens. This token is generated only by the base models.
  - `<|start_header_id|>` and `<|end_header_id|>`: These tokens enclose the role for a particular message. The possible roles are: [system, user, assistant, and ipython]
  - `<|eom_id|>`: End of message. A message represents a possible stopping point for execution where the model can inform the executor that a tool call needs to be made. This is used for multi-step interactions between the model and any available tools. This token is emitted by the model when the Environment: ipython instruction is used in the system prompt, or if the model calls for a built-in tool.
  - `<|eot_id|>`: End of turn. Represents when the model has determined that it has finished interacting with the user message that initiated its response. This token signals to the executor that the model has finished generating a response.
  - `<|python_tag|>`: Special tag used in the model’s response to signify a tool call.
- MediaTek Research proposed:
  - `<|use_tool|>` and `<|answer|>`: decision tokens
  - `<|start_img|>`, `<|img|>`, and `<|end_img|>`
  - `<|start_bbox|>` and `<|end_bbox|>`


## Prompt format for base models


For base models, the prompt format for a simple completion is as follows


##### Input Prompt Format
```
<|begin_of_text|>The color of the sky is blue but sometimes it can also be
```

##### Model Response Format
```
 gray or even purple. The color of the sky can change depending on the time of day, the weather, and the amount of pollution in the air. The color of the sky can also be affected by the presence of dust, smoke, and other particles in the air.

## Step 1: Identify the factors that
```

## Prompt format for base models with Image


For base models, here is an example of how the text completion format looks with an image,


##### Input Prompt Format
```
<|begin_of_text|><|start_img|><|img|><|img|><|img|><|img|><|img|><|end_img|>If I had to write a haiku for this one
```

##### Model Response Format
```
, it would be: A skateboarder's delight, a puppy on a board, a furry little thrill-seeker. This puppy is a true skateboarding enthusiast, always eager to hit the streets and show off his skills. He's a master of the board, gliding effortlessly across the pavement with grace and style.
```

## User and assistant conversation

Here is a regular multi-turn user assistant conversation and how its formatted.

##### Input Prompt Format
```
<|begin_of_text|><|start_header_id|>system<|end_header_id|>

You are a helpful assistant.<|eot_id|><|start_header_id|>user<|end_header_id|>

Who are you?<|eot_id|><|start_header_id|>assistant<|end_header_id|>


```

##### Model Response Format
```
I'm an AI assistant, which means I'm a computer program designed to simulate conversation and answer questions to the best of my ability. I'm here to help you with any questions or tasks you may have, and I'll do my best to provide you with accurate and helpful information.

I don't have a personal name, but I'm often referred to as a "virtual assistant" or a "chatbot." I'm a machine learning model, which means I've been trained on a large dataset of text and can generate responses based on patterns and context.

I can help with a wide range of topics, from general knowledge and trivia to more specialized subjects like science, history, and technology. I can also assist with tasks like language translation, text summarization, and even generating creative content like stories or poetry.

So, what can I help you with today?<|eot_id|>
```

## Zero shot function calling

All available functions can be provided in the system message.
Here is an example for the same,


##### Input Prompt Format

```
<|begin_of_text|><|start_header_id|>system<|end_header_id|>

Customized Functions: [{'name': 'get_weather', 'description': 'Get the current_weather', 'parameters': {'type': 'object', 'properties': {'location': {'type': 'string', 'description': 'The city and state, e.g. San Francisco, CA'}, 'unit': {'type': 'string', 'enum': ['celsius', 'fahrenheit']}}, 'required': ['location']}}]

---
You are a helpful assistant.<|eot_id|><|start_header_id|>user<|end_header_id|>

What is the weather in SF and Seattle?<|eot_id|><|start_header_id|>assistant<|end_header_id|>


```

##### Model Response Format

```
<|use_tool|>[get_weather(city='San Francisco', metric='celsius'), get_weather(city='Seattle', metric='celsius')]<|eot_id|>
```


##### Notes

- The output supports multiple tool calls


## Zero shot function calling E2E format


Here is an example of the e2e cycle of tool calls with the model in a muti-step way.


##### Input Prompt Format


```
<|begin_of_text|><|start_header_id|>system<|end_header_id|>

Customized Functions: [{'name': 'get_weather', 'description': 'Get the current_weather', 'parameters': {'type': 'object', 'properties': {'location': {'type': 'string', 'description': 'The city and state, e.g. San Francisco, CA'}, 'unit': {'type': 'string', 'enum': ['celsius', 'fahrenheit']}}, 'required': ['location']}}]

---
You are a helpful assistant.<|eot_id|><|start_header_id|>user<|end_header_id|>

What is the weather in SF and Seattle?<|eot_id|><|start_header_id|>assistant<|end_header_id|>

<|python_tag|>[get_weather(city='San Francisco', metric='celsius'), get_weather(city='Seattle', metric='celsius')]<|eom_id|><|start_header_id|>ipython<|end_header_id|>

[{'temperature': '25 celsius'}, {'temperature': '21 celsius'}]<|eot_id|><|start_header_id|>assistant<|end_header_id|>


```

##### Model Response Format
```
The weather is 25 C in San Francisco and 21 C in Seattle.<|eot_id|>
```


##### Notes

- The output of the function call is provided back to the model as a tool response ( in list of json format ).
- Notice `<|start_header_id|>ipython<|end_header_id|>` as the header message preceding the tool response.
- The model finally summarizes the information from the tool response and returns the result to the user.


## User and assistant conversation with Images

This example shows how to pass and image to the model as part of the messages.

##### Input Prompt Format
```
<|begin_of_text|><|start_header_id|>system<|end_header_id|>

You are a helpful assistant.<|eot_id|><|start_header_id|>user<|end_header_id|>

<|start_img|><|img|><|img|><|img|><|img|><|img|><|img|><|img|><|end_img|>\nDescribe this image in two sentences<|eot_id|><|start_header_id|>assistant<|end_header_id|>


```

##### Model Response Format
```
The image depicts a small dog standing on a skateboard, with its front paws firmly planted on the board and its back paws slightly raised. The dog's fur is predominantly brown and white, with a distinctive black stripe running down its back, and it is wearing a black collar around its neck.<|eot_id|>
```


##### Notes

- The `<|img|>` tag is used to indicate presence of the image
- The model is an early fusion model so actually translate an image into several tokens (multiply `<|img|>` tokens)
- The <|img|> tag is part of the user message body, implying that it should only come after the header `<|start_header_id|>{role}<|end_header_id|>` in the message body
- We recommend using a single image in one prompt

## User and assistant conversation with Images and Bounding Boxes

This example shows how to pass and image to the model as part of the messages.

##### Input Prompt Format
```
<|begin_of_text|><|start_header_id|>system<|end_header_id|>

You are a helpful assistant.<|eot_id|><|start_header_id|>user<|end_header_id|>

<|start_img|><|img|><|img|><|img|><|img|><|img|><|img|><|img|><|end_img|>What kind of animal is shown in the region <|start_bbox|>[[0, 0, 500, 500]]<|end_bbox|> ? <|eot_id|><|start_header_id|>assistant<|end_header_id|>


```

##### Model Response Format
```
The animal is a dog.<|eot_id|>
```

##### Notes

- The coordinates of bounding boxes are normalized between 0 and 1000.
