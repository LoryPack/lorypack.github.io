---
layout: post
title: Finetuning GPT3
date: 2022-11-26 15:09:00
description: Using the OpenAI API to finetune GPT3 on a custom dataset
---

Large Language Models have quite extraordinary performance. They can generate text, translate languages, and even answer questions. 
However, they are not perfect (fun fact: this last sentence was actually suggested by Github copilot, which is powered by a language model :laughing:). If you have a specific task, you may want to finetune the model on your own dataset. 

OpenAI API allows you to do that. You can finetune GPT3 and then using it as you wish. In the notebook below, I show how to 
finetune GPT3 to predict the title of an arxiv paper from its abstract. Hope you enjoy it!

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/gist/LoryPack/c53b41e7041f77ac450f7394455d2d2e#scrollTo=GwQeLZWFGs7_)

One disclaimer: to use the OpenAI API you need to register and using it has a monetary cost.




