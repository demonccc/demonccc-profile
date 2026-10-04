# Qwen Web-Search Agent on Hugging Face

---
id: project-huggingface-qwen-web-search-agent
type: project
category: research
period: 2026-present
status: active
capabilities:
  - generative-ai
  - agents
  - llm-inference
  - tool-calling
  - python
---

## Overview & Motivation

Public prototype exploring how an open-weight language model can combine local/hosted inference with external web-search tools to answer questions that require current information.

## Architecture & Implementation

The application runs as a Hugging Face Space with a Gradio interface. It loads quantized GGUF models through a llama.cpp-based runtime and uses agent/workflow logic to decide when an external search is required before synthesizing the final answer.

## My Contribution

I designed, implemented and deployed the application, including model loading, GPU configuration, agent flow, web-search integration and the conversational UI.

## Key Capabilities Demonstrated

LLM runtime engineering, quantized inference, agent workflows, tool calling, asynchronous orchestration and public AI application deployment.

## Evidence

- Hugging Face Space: https://huggingface.co/spaces/demonccc/Qwen_Uncensored_Chat_with_Web_Search
- Hugging Face profile: https://huggingface.co/demonccc

## Tech Stack

Python, Hugging Face Spaces, Gradio, LlamaIndex, llama.cpp, GGUF, GPU inference and web-search APIs.
