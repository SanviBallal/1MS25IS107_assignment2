# ReAct Agent with Google Gemini

A Python-based **ReAct (Reason + Act) agent** built on Google Gemini. The agent searches a small collection of AI-related documents using a custom tool and uses what it finds to produce context-aware answers.

The notebook shows the core mechanism behind an AI agent: the model decides to call a tool, receives the result, and keeps reasoning in a loop until it can answer.

## How It Works

1. The user asks a question.
2. Gemini decides whether it needs to call the `search_documents` tool.
3. The tool searches the document collection and returns the best-matching passages.
4. The result goes back to the model, which either searches again or writes a final answer.
5. The loop stops when the model answers without a tool call, or after the maximum number of steps (default: 8).

```
Question -> Gemini -> tool call? --yes--> search_documents -> result -> Gemini (repeat)
                          |
                          no
                          v
                    Final answer
```

## Features

- **Custom search tool:** keyword-based search over a set of in-memory documents. It scores each document by how many query words it contains and returns the top two matches with source labels.
- **Manual agent loop:** automatic function calling is disabled, so every reasoning and tool-call step is visible in the output.
- **Retry handling:** handles temporary `429` and `503` API errors with increasing wait times (up to 5 attempts).
- **Secure key handling:** the API key is read from Colab Secrets, with a hidden prompt as a fallback. No key is stored in the code.
- **Grounded answers:** the system prompt requires the model to call the search tool before answering questions about the documents.

## Document Collection

The knowledge base is a Python dictionary (`DOCUMENTS`) with four short passages:

| Key | Topic |
|---|---|
| `fill_in1` | Artificial intelligence |
| `fill_in2` | Machine learning (supervised, unsupervised, reinforcement) |
| `fill_in3` | Generative AI and large language models |
| `fill_in4` | AI agents and ReAct agents |

You can add or replace entries to point the agent at your own content.

## Tech Stack

- Python 3
- Google Colab
- `google-genai` (Gemini SDK)
- Model: `gemini-3.5-flash-lite`

## Getting Started

### 1. Get a Gemini API key
1. Go to [Google AI Studio](https://aistudio.google.com/).
2. Sign in and click **Get API key** -> **Create API key**.
3. Copy the key (it starts with `AIza...`).

The free tier allows about 10 requests per minute and 250 per day, which is enough for this notebook.

### 2. Run the notebook
1. Click the **Open in Colab** badge above.
2. In Colab, open the **Secrets** panel (key icon) and add a secret named `GOOGLE_API_KEY` with your key. Turn on notebook access.
3. Run the cells from top to bottom.

### 3. Run locally (optional)
```bash
pip install google-genai
export GOOGLE_API_KEY="your-key-here"
```
Replace the Colab `userdata.get(...)` calls with `os.environ["GOOGLE_API_KEY"]`.

## Usage

```python
run_agent("What are the different types of machine learning?")
```

Example output:

```
Question: What are the different types of machine learning?
--------------------------------------------------
Step 1: model calls search_documents({'query': 'types of machine learning'})
Search result preview: [Source: fill_in2] ...

Final answer:
Machine learning includes supervised learning (labeled data), unsupervised
learning (unlabeled data), and reinforcement learning (rewards and penalties).
```

## Test Questions

The notebook runs the agent on five questions:

1. What is artificial intelligence? *(answered from the documents)*
2. What are the different types of machine learning? *(answered from the documents)*
3. What is generative AI and what can it create? *(answered from the documents)*
4. How does quantum teleportation work? *(not covered by the documents)*
5. Who invented the World Wide Web? *(general knowledge)*

Questions 4 and 5 test how the agent behaves when the search tool has no relevant information.
