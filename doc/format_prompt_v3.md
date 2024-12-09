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
[get_weather(city='San Francisco', metric='celsius'), get_weather(city='Seattle', metric='celsius')]<|eot_id|>
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


