---
layout: project
title: Building TinyGPT from Scratch
date: 2026-04-04
tech: [PyTorch, Transformers, Deep Learning]
excerpt: "Implementation of a minimal GPT model from scratch to deeply understand how transformers work.(In progress ...)"
image: /assets/images/tinygpt.png
---

## Overview
I implemented a minimal GPT model from scratch with a Pytorch high level API. Not to say it's from scratch, I just want to show all process we can use to optimize model for training and inference with high constraints GPU ressources, and memory requirements. Just stay connected!

## What about ?
Before starting, I know, you usually hear about LLM, Agent and you ask yourself perhaps why this article, I stop you immediately, see as a toy project , as a proof you can create your own llm with your own constraints, your own will. You can start also from an existing , architecture, or model to a new. My main goal throughout this project is to give you skills to create and optimize an llm from model, architecture to real world problem and Cloud.For this you must undestand architecture mechanism that I implemented here.

## Data Preparation
*[Section to fill: You can explain your tokenization choices (word-level vs character-level) and how you built your vocabulary here]*

## Architecture

The architecture of this model is based on the classic Transformer foundations (decoder-only), implemented from scratch to master every tensor flow:

- **Embeddings**: Converting tokens into dense vectors. The technical challenge here was implementing **Positional Encoding** to give the model a sense of time and word order, while managing complex broadcasting issues between sequences and batches.
- **Transformer Bloc**: The core of the model. It integrates a **Multi-Head Self-Attention** mechanism coupled with a strict **Causal Mask**, ensuring the model cannot "cheat" by looking at future words during training.
- **Normalization**: Using **Layer Normalization** before attention and Feed-Forward networks. This step is crucial for stabilizing gradients when training on resource-constrained hardware.
- **Linear / Output**: The final projection layer that transforms latent representations into dimensions equal to our vocabulary size, generating the raw **logits** needed for probability calculation.

## Training Results

Final loss: [FILL HERE: e.g., 0.15]  

Example generation:

> [FILL HERE: Gutenberg text example]

## Key Challenges

- **Data Fetching & Processing**: Moving from a "character" to a "word" approach requires rigorous management. Using **Regex** to filter special characters and numbers was vital to reduce noise and keep the vocabulary size manageable for our GPU.
- **GPU Memory Optimization & Training**: 
    - **Context Management**: Implementing a strict sliding window (**slicing**) to ensure the input tensor never exceeds the defined `block_size` (e.g., 64), avoiding *IndexError* and RAM spikes.
    - **Gradient Flow**: Identifying and fixing "silent bugs," such as accidental gradient disconnection (`.detach()`) in encoding functions, which prevented the network from updating its weights.
    - **Data Type Safety**: Shifting from raw string manipulation to integer ID lists to ensure data integrity between CPU and GPU.
- **Inference**: *See dedicated section below.*

## Inference

*[Section to be written later]*

## Links

- GitHub repo: [Kazeo57/tinyGPT](https://github.com/Kazeo57/tinyGPT.git)

## Resources
- [Implementing nanoGPT from scratch using PyTorch: A guide with pride and prejudice](https://medium.com/@kelly.nguyen01/implementing-nanogpt-from-scratch-using-pytorch-a-guide-with-pride-and-prejudice-58f3b51f9a84)