<p align="center">
  <img src=https://github.com/deep-diver/auto-data-fountain/blob/main/assets/janus-logo.png?raw=true />
</p>

# Janus

The goal of Auto Data Fountain is to generate synthetic data in order to fine-tune a large language model for arbitrary situations in a systematic way. Auto Data Fountain generates synthetic data by leaveraging the power of Large Language Mode(LLM). Currently supported LLM is Google's Gemini Pro, and GPT4 will be supported soon. 

## Motivation

There are many well-known and well-documented contents on how to fine-tune large language models. However, it is hard to collect hundreds of thousands of data for the fine-tuning. To this end, there have been various methods proposed to generate synthetic dataset such as [Stanford Alpaca](https://crfm.stanford.edu/2023/03/13/alpaca.html), [WizardLM's Evol-Instruct](https://github.com/nlpxucan/WizardLM), and so on. 

Those synthetic data generation methods define seed prompts, then LLM(mostly GPT4) continues to generate (prompt, output) pairs derived from it in various directions. The fine-tuned LLMs on the datasets generated by these methods has shown good results so far. However, there are two main concerns of these methods, and Auto Data Fountain tries to solve them in the follwoing manner.

1. The seed prompts are manually hand crafted. The seed prompts are very important since all the derived (prompt, output) pairs have dependency on them. If you look into some of the examples (such as [Alpaca](https://github.com/tatsu-lab/stanford_alpaca/blob/main/seed_tasks.jsonl)), it is hard to capture how each prompt is designed and weather there are implicit biases or ethical violations. 
    - ⛲️ _**Auto Data Foundation**_ uses [Mermaid](https://mermaid.js.org/) syntax to represent entities and their relationship. Also, Mermaid allows us to define detailed attributes of each entity and the relationship with dedicated syntax and comments. Underlying assumption is that certain situations could be depicted better and clearer in structured format than plain natural human language. Based on the Mermaid definition, the seed prompts are automatically generated.
      
2. These methods were applied to build up a sort of general purpose fine-tuned LLMs. General purpose doesn't mean the fine-tuned LLMs are in the level of expert to solve problems on any arbitrary tasks. Rather it tries to give you general answers on various tasks. This is why some modern datasets (in 2023~2024) are constructed by merging lots of different datasets from various sources. However, we can not find possible dataset on every single possible scenarios.
    - Since ⛲️ _**Auto Data Foundation**_ leverages the Mermaid syntax, we can describe a certain situations or task in more sophisticated manner. Further, Auto Data Foundation let us define the evolving directions that guide how derived prompts from the seed prompts should be generated.

## Basics (example)

Let's say we want to build a counseling chatbot for marriage guidance.

1. Create a directory named `marriage_counsel`
```bash
$ mkdir -p marriage_counsel
```

2. Create `setup.yaml` and `diagram.mermaid` in the `marriage_counsel` directory.
    - `setup.yaml` defines basic information such as initial prompt to generate seed prompts, derivational prompts to derive diverse prompts based on the seed prompts, roles of a user and an assistant.
    - `diagram.mermaid` defines entities and their relationship along with attributes.

Below shows the basic structure of the two files. If you are curious more, take a look at the actual file contents [setup.yaml](https://github.com/deep-diver/auto-data-fountain/blob/main/samples/marriage_counsel/setup.yaml), [diagram.mermaid](https://github.com/deep-diver/auto-data-fountain/blob/main/samples/marriage_counsel/diagram.mermaid). Note that you should open up the `diagram.mermaid` file in raw format to see the hidden comments.

```yaml
# setup.yaml
initial_prompt: |
  ....

derivational_prompt: |
  ....

output_format: | 
  The generated conversations are recorded in a valid JSON as
  {"conversations":[{"user": text, "assistant": text},...]}.  

delimiter: "------------------------------"

user_role: COUNSELEE
assistant_role: COUNSELOR

seed_evolving_directions:
  - general

derivational_evolving_directions:
  - general
  - in-depth
```

```
# diagram.mermaid
erDiagram
    COUNSELOR ||--|{ COUNSELEE : "provides counseling to"

    %% Comments for relationship attributes
    %% Start date: 2024-02-14
    %% Frequency: Weekly
    %% Topic: marriage guidance
```

```mermaid
erDiagram
    COUNSELOR ||--|{ COUNSELEE : "provides counseling to"

    %% Comments for relationship attributes
    %% Start date: 2024-02-14
    %% Frequency: Weekly
    %% Topic: marriage guidance
```

4. install the dependencies
```bash
$ pip install -r requirements.txt
```

5. run the `main.py` script

Below shows the CLI arguments to run the `main.py` script. Currently Gemini Pro is the only supported backend LLM. If you don't have the API key yet, grasp one at [Google AI Studio](https://cloud.google.com/generative-ai-studio). It's free. GPT4 support will be added later.

```bash
$ python src/main.py --help
...

options:
  -h, --help            show this help message and exit
  --gemini-api-key ➡️ Gemini API key from Google AI Studio
  --target-folder ➡️ In which folder to look up for setup.yaml and diagram.mermaid
  --target-filename ➡️ Filename to store the generated outputs. The file will be created in the same folder as target-folder
  --type ➡️ Multi-turn conversations or single turn instruction & response
  --d-factor ➡️ How many times to generate outputs on a single direction
  --retry-num ➡️ How many times to retry when failing at calling Gemini API or parsing JSON
```

For the example of marriage counsel, you can run the following CLI with the prepared setup. Then, it will generate 40 (prompt, output) pairs, and you can see the actual output in the [output.json](https://github.com/deep-diver/auto-data-fountain/blob/main/samples/marriage_counsel/outputs.json) file.

```bash
$ python src/main.py \ 
--gemini-api-key "..." \ 
--target-folder samples/marriage_counsel
```
