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
*[Section à remplir par tes soins : Tu pourras expliquer ici tes choix de tokenisation (mots vs caractères) et la création de ton vocabulaire]*

## Architecture

L'architecture de ce modèle repose sur les fondations classiques d'un Transformer (décodeur uniquement), mais implémentée de zéro pour maîtriser chaque flux de tenseur :

- **Embeddings** : Transformation des tokens en vecteurs denses. Le défi technique ici a été l'implémentation de l'encodage de position (*Positional Encoding*) pour donner au modèle la notion du temps et de l'ordre des mots, tout en gérant les problèmes complexes de *broadcasting* (dimensions entre séquences et batchs).
- **Transformer Bloc** : Le cœur du modèle. Il intègre le mécanisme d'attention multi-têtes (*Multi-Head Self-Attention*) couplé à un masque causal (Causal Mask) strict, garantissant que le modèle ne triche pas en regardant les mots futurs pendant l'entraînement.
- **Normalization** : Utilisation de *Layer Normalization* avant l'attention et les réseaux *Feed-Forward*. Cette étape est cruciale pour stabiliser les gradients lors de l'apprentissage sur des machines à ressources limitées.
- **Linear / Output** : La couche de projection finale qui transforme les représentations latentes en dimensions égales à la taille de notre vocabulaire, générant ainsi les *logits* bruts nécessaires au calcul des probabilités.

## Training Results

Final loss: [À REMPLIR : ex: 0.15]  

Example generation:

> [À REMPLIR : Guterberg text example]

## Key Challenges

- **Data fetching & Processing** : Passer d'une approche par "caractère" à une approche par "mot" nécessite une gestion rigoureuse. L'utilisation d'expressions régulières (Regex) pour filtrer les caractères spéciaux et les chiffres a été vitale pour réduire le bruit et ramener la taille du vocabulaire à un niveau gérable pour notre GPU.
- **GPU memory optimization & Training** : 
  - **Gestion du contexte** : Implémentation d'une fenêtre glissante (slicing) stricte pour s'assurer que le tenseur d'entrée ne dépasse jamais le `block_size` défini (ex: 64), évitant ainsi les erreurs de type *IndexError* et l'explosion de la RAM.
  - **Gradient Flow** : La détection et correction de bugs "silencieux", comme la coupure accidentelle du gradient (`.detach()`) dans les fonctions d'encodage, qui empêchait le réseau de mettre à jour ses poids.
  - **Sécurité des types de données** : Le passage de la manipulation de texte brut (Strings) à des listes d'identifiants numériques (IDs) pour garantir l'intégrité des données entre le CPU et le GPU.
- **Inference** : *Voir section dédiée ci-dessous.*

## Inference

*[Section à rédiger plus tard]*

## Links

- GitHub repo: [Kazeo57/tinyGPT](https://github.com/Kazeo57/tinyGPT.git)

## Ressources
- [Implementing nanoGPT from scratch using PyTorch: A guide with pride and prejudice](https://medium.com/@kelly.nguyen01/implementing-nanogpt-from-scratch-using-pytorch-a-guide-with-pride-and-prejudice-58f3b51f9a84)