<p align="center">
  <img src="https://github.com/deep-diver/auto-data-fountain/blob/main/assets/janus-logo.png?raw=true" height="70%" />
</p>

# Janus

The goal of Janus is to generate synthetic data in order to fine-tune a large language model for arbitrary situations in a systematic way. Janus generates synthetic data by leaveraging the power of Large Language Mode(LLM). Currently supported LLMs are Google's Gemini Pro 1.0 and OpenAI's GPT4, and other open source local LLMs will be supported later. 

## Motivation

There are many well-known and well-documented contents on how to fine-tune large language models. However, it is hard to collect hundreds of thousands of data for the fine-tuning. To this end, there have been various methods proposed to generate synthetic dataset such as [Stanford Alpaca](https://crfm.stanford.edu/2023/03/13/alpaca.html), [WizardLM's Evol-Instruct](https://github.com/nlpxucan/WizardLM), and so on. 

Those synthetic data generation methods define seed prompts, then LLM(mostly GPT4) continues to generate (prompt, output) pairs derived from it in various directions. The fine-tuned LLMs on the datasets generated by these methods has shown good results so far. However, there are two main concerns of these methods, and Janus tries to solve them in the follwoing manner.

1. The seed prompts are manually hand crafted. The seed prompts are very important since all the derived (prompt, output) pairs have dependency on them. If you look into some of the examples (such as [Alpaca](https://github.com/tatsu-lab/stanford_alpaca/blob/main/seed_tasks.jsonl)), it is hard to capture how each prompt is designed and weather there are implicit biases or ethical violations. 
    - _**Janus**_ uses [Mermaid](https://mermaid.js.org/) syntax to represent entities and their relationship. Also, Mermaid allows us to define detailed attributes of each entity and the relationship with dedicated syntax and comments. Underlying assumption is that certain situations could be depicted better and clearer in structured format than plain natural human language. Based on the Mermaid definition, the seed prompts are automatically generated.
    - If you are not familiar with Mermaid yet
      - The official doc comes with straight forward explanation to understand its syntax, and it is not difficult to draw diagrams manually.
      - However, there are various tools around it as well. For instance, the official website comes with [Mermaid Live Editor](https://mermaid.live/), or [Mermaid Flow](https://www.mermaidflow.app/) provides [Visual Editor](https://www.mermaidflow.app/flowchart), or there are VSCode extensions as well.
      - Also, if you are familar with SQL syntax, there are conversion tools such as [sql2mermaid](https://github.com/nkato/sql2mermaid) that turn SQL to Mermaid relational diagram as well.
      
2. These methods were applied to build up a sort of general purpose fine-tuned LLMs. General purpose doesn't mean the fine-tuned LLMs are in the level of expert to solve problems on any arbitrary tasks. Rather it tries to give you general answers on various tasks. This is why some modern datasets (in 2023~2024) are constructed by merging lots of different datasets from various sources. That means it is hard to find a universally complete dataset on every single possible scenarios.
    - Since _**Janus**_ leverages the Mermaid syntax, we can describe a certain situations or task in more sophisticated manner. Further, Janus let us define the evolving directions that guide how derived prompts from the seed prompts should be generated.

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

Below shows the CLI arguments to run the `main.py` script. Currently Gemini Pro is the only supported backend LLM. If you don't have the API key yet, grasp one at [Google AI Studio](https://cloud.google.com/generative-ai-studio). It's free. If you want to use OpenAI's Chat Completion API w/ GPT4, feel free to use do so with `--backend-llm gpt` option.

```bash
$ python src/main.py --help
...

options:
  --backend-llm {gemini,gpt}
  --api-key ➡️ API key for backend LLM
  --target-folder ➡️ In which folder to look up for setup.yaml and diagram.mermaid
  --target-filename ➡️ Filename to store the generated outputs. The file will be created in the same folder as target-folder
  --type ➡️ Multi-turn conversations or single turn instruction & response
  --s-factor ➡️ How many times to generate outputs on a single direction in seed generation
  --d-factor ➡️ How many times to generate outputs on a single direction in derivational generation
  --retry-num ➡️ How many times to retry when failing at calling Gemini API or parsing JSON

  --hf-hub-repo-id ➡️ Repository ID of Hugging Face Hub (Dataset)
  --hf-token ➡️ Hugging Face Hub Access Token
  --hf-append ➡️ Whether to append outputs to existing data on the Hugging Face Hub
```

For the example of marriage counsel, you can run the following CLI with the prepared [setup](https://github.com/deep-diver/janus/tree/main/samples/marriage_counsel). Then, it will generate 40 (prompt, output) pairs, and you can see the actual output in the [output.json](https://github.com/deep-diver/auto-data-fountain/blob/main/samples/marriage_counsel/outputs.json) file.

```bash
$ python src/main.py \
--backend-llm gemini \
--api-key "..." \
--target-folder samples/marriage_counsel \
--type conversation \
--hf-hub-repo-id chansung/test-ds \ 
--hf-token "..." \
--hf-append # append to the existing dataset on the hub
```

For the same example of marriage counsel but in the single turn based data generation ([setup](https://github.com/deep-diver/janus/tree/main/samples/marriage_counsel_instruct)), you can run the following CLI. Then you can check out the example output in the [output.json](https://github.com/deep-diver/janus/blob/main/samples/marriage_counsel_instruct/outputs.json) file.

```bash
$ python src/main.py \
--backend-llm gemini \
--api-key "..." \
--target-folder samples/marriage_counsel_instruct \
--type instruct
```

## UI to filter out unsatisfied outputs

This tool is under development, and the dev version is hosted on the [Hugging Face Space](https://huggingface.co/spaces/chansung/janus-filtering). The purpose of this tool is to let users interact with generated outputs and decide to keep or discard what.

<p align="center">
  <img src="https://github.com/deep-diver/auto-data-fountain/blob/main/assets/filtering-screen.png?raw=true" width="70%" />
</p>


## TODOs

- [ ] More documnets on the generating process
- [ ] Push to Hugging Face Hub
- [ ] UI and Hugging Face Hub integration