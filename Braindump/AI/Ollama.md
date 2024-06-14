#ai

[Ollama](https://www.ollama.com/) allows to run your own LLM locally on your device.
## Installation

* Download and install the latest version of Ollama from the [official website](https://www.ollama.com/).
* Run the `llama3:8b` model

```bash
ollama run llama3:8b
```

## Customization

### Change Storage Location of Models

By default Ollama is installed on your C:\ drive. You can change this if you set the `OLLAMA_MODELS` environment variable.

* Start -> Edit the system environment variables -> Environment Variables -> New -> `OLLAMA_MODELS` -> `D:\Software\Ollama`

### Use Models from Registry

Ollama has a registry where you can pull and push models from/to.

You can find the list of models in their registry via the [library](https://ollama.com/library).  
To run a model from a registry you can specify the `run` command.

```bash
ollama run rouge/daybreak-kunoichi-2dpo-7b
```
### Create Custom Model

* Create a modelfile based on an existing model

```bash
ollama show llama3:8b --modelfile  >.\storyteller.modelfile
```

* Add your custom instructions to the modelfile

```txt
SYSTEM """
You are a medieval storyteller. You will create an immersive and detailed story from every message you receive. Use a maximum of 6 sentences.
Respond to my messages and craft a visual and engaging story of them.
"""
```

* Create a new model based on the modelfile

```bash
ollama create storyteller --file .\storyteller.modelfile
```

* Test your new model

```bash
ollama run storyteller
```

[Reference](https://medium.com/@sumudithalanz/unlocking-the-power-of-large-language-models-a-guide-to-customization-with-ollama-6c0da1e756d9)

## Running Models from Hugging Face

[Hugging Face](https://huggingface.co/Orenguteng/Llama-3-8B-Lexi-Uncensored) is a website where people can upload models. You can download these models and run them locally.

Download [the model](https://huggingface.co/Orenguteng/Llama-3-8B-Lexi-Uncensored-GGUF/blob/main/Lexi-Llama-3-8B-Uncensored_F16.gguf) the website. You need to use a `gguf` extension. At this moment I have no clue why.

Create a ``modelfile``.

```bash
touch lexi-llama3-uncensored.modelfile
```

Add the following and make sure that your `FROM` points to the location of your downloaded model.

```bash
# Modelfile  
FROM "./Lexi-Llama-3-8B-Uncensored_F16.gguf"  
  
PARAMETER stop "<|im_start|>"  
PARAMETER stop "<|im_end|>"  
  
TEMPLATE """  
<|im_start|>system  
{{ .System }}<|im_end|>  
<|im_start|>user  
{{ .Prompt }}<|im_end|>  
<|im_start|>assistant  
"""
```

Create a model in `ollama` from your `modelfile`.

```bash
ollama create lexi-llama3-uncensored -f lexi-llama3-uncensored.modelfile
```

Run the model.

```bash
ollama run lexi-llama3-uncensored
```

[Ollama + HuggingFace ✅🔥. Create Custom Models From Huggingface… | by Sudarshan Koirala | Medium](https://medium.com/@sudarshan-koirala/ollama-huggingface-8e8bc55ce572)
## API

Once your Ollama application is [[Ollama#Installation|running]], you can query it via the [API](https://github.com/ollama/ollama/blob/main/docs/api.md).  
The default port is `11434` but this can be changed using the `OLLAMA_HOST` environment variable.

### Completion

You can query a completion via the following commands.

```bash
curl http://localhost:11434//api/generate -d '{
  "model": "storyteller",
  "prompt": "The Duke of Cambridge enters my court hall. He kneels in front of me."
}'
```

Alternatively in Powershell:
```powershell
Invoke-RestMethod -Uri "http://localhost:11434/api/generate" -Method Post -Body '{"model": "storyteller", "prompt": "The Duke of Cambridge enters my court hall. He kneels in front of me."}' -ContentType "application/json"
```

Note by default the response is streamed. If you want to receive a single response with the full text you can specify the `stream` parameter and set it to `false`.

```powershell
Invoke-RestMethod -Uri "http://localhost:11434/api/generate" -Method Post -Body '{"model": "storyteller", "prompt": "The Duke of Cambridge enters my court hall. He kneels in front of me.", "stream": false}' -ContentType "application/json"
```

Optionally you can overwrite the `system` message in the prompt to specify what instructions the model should follow.
```powershell
Invoke-RestMethod -Uri "http://localhost:11434/api/generate" -Method Post -Body '{"model": "storyteller", "prompt": "The Duke of Cambridge enters my court hall. He kneels in front of me.", "stream": false, "system": "You are a dog and you only bark"}' -ContentType "application/json"

model                : storyteller
created_at           : 2024-06-11T22:52:39.9366121Z
response             : RUFF RUFF RUFF! *ears perked up* WOOOOO! *barks excitedly, trying to get the Duke's attention* RUFF RUFF RUFF! *wags tail* WOOOO! *tries to lick the Duke's face* RUFF RUFF RUFF!
done                 : True
done_reason          : stop
context              : {128006, 9125, 128007, 881...}
total_duration       : 1003303100
load_duration        : 2494700
prompt_eval_count    : 36
prompt_eval_duration : 375542000
eval_count           : 65
eval_duration        : 624148000
```

### Chat

More often you want to perform a chat operation, where you keep a set of messages as history that can be used as context.

```powershell
Invoke-RestMethod -Uri "http://localhost:11434/api/chat" -Method Post ` -Body '{"model": "storyteller", "messages": [ {"role": "user", "content": "The Duke of Cambridge enters my court hall. He kneels in front of me."}, {"role": "user", "content": "I take my knife and cut off his ear"}, {"role": "user", "content": "Lex the dog enters the room and starts rolling around in the blood."}], "stream": false}' ` -ContentType "application/json"
```

## Open WebUI

[GitHub - open-webui/open-webui: User-friendly WebUI for LLMs (Formerly Ollama WebUI)](https://github.com/open-webui/open-webui)

