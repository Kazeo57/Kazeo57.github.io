---
layout: project
title: Building TinyGPT from Scratch (Part 1)
date: 2026-04-04
tech: [PyTorch, Transformers, Deep Learning]
excerpt: "Implementation of a minimal GPT model from scratch to deeply understand how transformers work."
image: /assets/images/tinygpt.png
---

## Overview
I implemented a minimal GPT model from scratch with a PyTorch high level API. Not to say it's from scratch, I just want to show all the process we can use to optimize a model for training and inference with high constraints GPU resources, and memory requirements. Just stay connected!

## What about?
Before starting, I know, you usually hear about LLM, Agent and you ask yourself perhaps why this article. I stop you immediately, see it as a toy project, as a proof you can create your own LLM with your own constraints, your own will. You can start also from an existing architecture, or model to a new one. My main goal throughout this project is to give you the skills to create and optimize an LLM from model, architecture to real world problem and Cloud. For this you must understand the architecture mechanism that I implemented here.

## Data Preparation
I followed the Medium article on ![Gutenberg](https://www.gutenberg.org/files/1342/1342-0.txt) use case for data collection. If you want to use another dataset, you can just put it and write a custom code to download it, nothing too hard. My long-term goal is to use low-resource languages instead of English here to work with.

## Architecture

The architecture of this model is based on the Transformer foundations (decoder-only), implemented from scratch to master every tensor flow:

- **Embeddings**: Converting tokens into dense vectors. The technical challenge here was implementing **Positional Encoding** to give the model a sense of time and word order, while managing complex broadcasting issues between sequences and batches.
- **Transformer Block**: The core of the model. It integrates a **Multi-Head Self-Attention** mechanism coupled with a strict **Causal Mask**, ensuring the model cannot "cheat" by looking at future words during training.
- **Normalization**: Using **Layer Normalization** before attention and Feed-Forward networks. This step is crucial for stabilizing gradients when training on resource-constrained hardware.
- **Linear / Output**: The final projection layer that transforms latent representations into dimensions equal to our vocabulary size, generating the raw **logits** needed for probability calculation.

If you don't keep the steps in your mind, calm down, it takes time to know what to do at each step to get the result I had. My roadmap, whatever I must code, is to know the final goal of the part. Unfortunately, the article I followed implemented a GPT based on BERT encoder logic. Maybe, finally, it was a good thing — when I understood that, I found the best way to implement it with causal attention using a high level API, and I'm sorry to say that even basic attention has been implemented from scratch by me. When we say "from scratch", my first thought is: is there another way to build this?

If I lost you when I said "BERT", just keep in mind that there are many types of Attention. In BERT, attention sees the words next to the word you want to predict and the previous ones, while in a standard LLM based on a decoder-only architecture like GPT, the attention must see only the past, i.e. the previous words before the one you predict. To ensure this, during the process, we write a causal mask — it can be a matrix of score size, with 1s and 0s, and we apply it on the target score. The 0 part hides the next words every time and 1 represents the previous words. If you understand what I'm trying to say, the first concern you could think of is: why 0 if it removes our operation? Yes, 0 is destructive; in practice, we use negative infinity. I'll tell you right away, infinity here has a very small value near 0 but not 0 by convention. So while you compute with that, the value tends to become smaller, going near infinity. That is the center of our work. If you understand the mask effect, you understand 50% of the work. If you have never implemented a basic attention to imagine where you must put the mask, just check it out below:

```python
class SelfAttention:
    def __init__(self, dim_model):
        self.Wq = nn.Linear(in_features=dim_model, out_features=dim_model)
        self.Wk = nn.Linear(in_features=dim_model, out_features=dim_model)
        self.Wv = nn.Linear(in_features=dim_model, out_features=dim_model)

    def forward(self, X):
        q = self.Wq(X)
        k = self.Wk(X)
        v = self.Wv(X)
        first = torch.matmul(q, torch.transpose(k, -2, -1))
        first = first / (q.size(-1))**0.5
        first = torch.softmax(first, dim=-1)
        score = torch.matmul(first, v)
        return score
```

To help you imagine: you have a sentence, every word becomes an integer. This is the whole integer stack that is put into the neural network, so you can focus on the attention part. The integer becomes a vector with size "vocabulary size" × "embedding dimension". In this case, each integer is associated with a random vector whose weights will be updated during the training process. The embedding is the element entered in the attention part to create a "Causal Attention". Let's try it:

"The cat sleeps" --> [2, 3, 4] 
Here is tokenization — the integers are just values associated with words in a dictionary-like structure. You can have something like this: `{"The": 2, "cat": 3, "sleeps": 4}`, and you use it to replace the sentence with its integer stack equivalent.

When you finish tokenization you transform, as I said above, into embeddings. You'll have something like this for instance:
2 --> [0.1,0.2,0.6,0.9] 
There, our embedding has a dimension of 4, meaning there are 4 elements to represent a single integer in our work. Don't forget an integer is a word, so finally you represent the word "The" with this vector. You will have N tensors at this step, with N equal to `vocab_size`. This is how you can represent each word of your vocabulary. The big problem is when you test on a new sentence with words not in the vocabulary — you'll get an error like `index out of range`, meaning there is no integer associated with an embedding to represent your word. That's the reason why the tokenization part is really important. For a next release I'll go deeper on this and show you how it can improve your model performance. For now let's focus on attention.

This vector goes from a simple embedding to a contextual embedding. But why "contextual"? Just to say that attention finds a link between each word of your sentence — yeah, literally as you understand a sentence. Astonishing, right?

You could have something like this: [0.1,0.2,0.6,0.9] --> [0.3,0.15,0.5,0.23]. Yeah, that's all! In attention history, many people worked with implementations of attention, from Yoshua Bengio in *Bahdanau Attention* (Additive attention) in 2014, *Luong Attention* in 2015 to *Attention Is All You Need*. I know, as many people, you thought perhaps that the attention of transformers is the only one — no, attention is just a concept, you can invent your own. The main goal of an attention mechanism is to find relationships between words, and at the beginning it was especially used to build Machine Translation models. But today, attention is everywhere, because relationship is what defines the world.

Here is an implementation of basic Masked Attention found in LLMs:

```python
class MaskedAttention:
    def __init__(self, embed_dim):
        self.Wq = nn.Linear(embed_dim, embed_dim)
        self.Wk = nn.Linear(embed_dim, embed_dim)
        self.Wv = nn.Linear(embed_dim, embed_dim)

    def forward(self, X):
        q = self.Wq(X)
        k = self.Wk(X)
        v = self.Wv(X)

        X = torch.matmul(q, k.transpose(-2, -1)) / torch.sqrt(q.size(-1))
        mask = torch.tril(torch.ones_like(X), diagonal=0)
        mask = torch.masked_fill(X, mask == 0, float('-inf'))
        weights = torch.softmax(mask, dim=-1)
        score = torch.matmul(weights, v)
        return score
```

`score` is a matrix — your contextual embedding. After that, it's often just a softmax layer or an intermediate layer + softmax to project, for each word, a probability over the vocabulary. The max can be your next token. You can find all my implementation for understanding in `test/scratch.py`. If you read the code carefully, you'll see it all starts with Q, K, V vectors — nothing complicated, just a projection of your entry stack vector, maybe a sentence. See them as copies of your entry. You compute a score between them to see which words are strongly linked to another, or just linked — the score matrix lets you see this. That's how you get a contextual representation of your words.

First, a dot product between Q and K, followed by a softmax to transform scores into probabilities, itself followed by a dot product between the weights obtained and the V vector. That's the core of modern attention.

I know it's not possible to talk about all components and elements of attention — I concede that — but I think what I'm giving you here is the base you need to introduce yourself to the attention concept. That's all I wanted to show you today. Sorry if not everything was covered, there will be other articles to go deeper into every concept around the LLM life cycle.

## Training Results

Final loss: 0.77978 at 49/50 epochs  

Example generation:

> all this was acknowledged here composure good attachment had the business on which some of your dislike of pride he scarcely felt made and however in town four thousand pounds may be more lucky if i were to herself silent and it my sister i had not often declare that he first by his good inquiries or thoughtlessness and sometimes with him she believed his manners were so much better than any feeling a case of his behaviour or had known him except but once been used to pursue painful fixed for its warmth are the strength of lottery tickets of giving happiness of

## Key Challenges

- **Data Fetching & Processing**: Moving from a "character" to a "word" approach requires rigorous management. Using **Regex** to filter special characters and numbers was vital to reduce noise and keep the vocabulary size manageable for our GPU. (cf `data/get_data.py`)
- **GPU Memory Optimization & Training**:
    - **Context Management**: Implementing a strict sliding window (**slicing**) to ensure the input tensor never exceeds the defined `block_size` (e.g., 64), avoiding *IndexError* and RAM spikes.
    - **Gradient Flow**: Identifying and fixing "silent bugs," such as positional embedding computation and shape mismatches.
    - **Data Type Safety**: Shifting from raw string manipulation to integer ID lists to ensure data integrity between CPU and GPU.
- **Inference**:
"Multinomial is all you need" but one thing I learnt is: the most probable next token is not necessarily the right token to put next.

## Inference
In a next article, I'll show how inference can improve your generation. Another challenge with a level like training optimization.

## Links

- GitHub repo: [Kazeo57/tinyGPT](https://github.com/Kazeo57/tinyGPT.git)

## Resources
- [Implementing nanoGPT from scratch using PyTorch: A guide with pride and prejudice](https://medium.com/@kelly.nguyen01/implementing-nanogpt-from-scratch-using-pytorch-a-guide-with-pride-and-prejudice-58f3b51f9a84)
- [Attention Is All You Need](https://arxiv.org/abs/1706.03762) (The original Transformer paper)
- [Evolution of Attention Mechanisms](https://arxiv.org/abs/1409.0473) (From Bahdanau to Luong)
- Other resources on LLM implementation and fine-tuning:
  - [Building a Large Language Model (LLM) from Scratch](http://medium.com/@raufpokemon00/building-a-large-language-model-llm-from-scratch-61fed0570ea5)
  - [Building Your Own LLM from Scratch: A Comprehensive Guide](https://medium.com/@palanikalyan27/building-your-own-llm-from-scratch-a-comprehensive-guide-7e38d9624d47)
  - [Understanding LLMs from Scratch Using Middle School Math](https://medium.com/data-science/understanding-llms-from-scratch-using-middle-school-math-e602d27ec876)