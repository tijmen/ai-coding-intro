# AI Coding

Date: March 21, 2025

Author: Tijmen de Haan
## Motivation
My programming speed has increased 10x over the last 3 years and is still rapidly increasing thanks to the advent of LLMs. I know many people aren't yet coding with LLMs, so my goal is to enable you to get the most out of LLM assistance. These notes are intended as a companion to my 1hr hands-on tutorial.

## Very brief intro to LLMs
At training time, LLMs have ingested most of the publicly available internet and more, including stackoverflow, all software package documentation and source code, github, etc. As such they are likely familiar with many of your tasks. Increasingly, they can generalize to unseen problems. 

But what do I mean by "ingested" and "familiar"? Well, the first stage of training is called pretraining. The LLM is a model that takes in a sequence of tokens and outputs a probability vector over the next token. During pretraining, the LLM is asked: for each token, what is the probability $P$ of this token given the previous tokens? The loss function is $<-\ln(P)>$ where the angled brackets indicate averaging over tokens. Through gradient descent, we minimize the loss function over model parameters. This is typically done on $\mathcal{O}(10^{14})$ tokens, expending millions and millions of dollars.

This pretraining stage can be thought of as lossily compressing the world's knowledge into the model parameters. This compression is done by learning all kinds of subtle statistical patterns between tokens. One school of thinking is these statistical patterns are just that: a large number of shallow correlations that trick us into thinking the model understands something. Others believe that the model deeply understands the world as some kind of emergent property. All can agree it doesn't matter: the models are practically useful. 

After pretraining, the model is known as a "base model" or "foundation model" and has some limited use, as is. For example, when I'm writing I can sometimes get stuck ("writer's block"). A pretrained model is quite good at continuing text, so I'll hand it what I have so far and it will continue the text in my style, giving me inspiration for what to write about next. Another way a pretrained model can be used directly is in "few-shot prompting" where you give a few examples of how you want the model to behave. For example, if you want the answer to some question $Q$, you could hand the model $[Q_1, A_1, Q_2, A_2, Q]$ and it'll continue with some answer similar to the answers you gave as examples. 

Now, let's consider post-training. There can be multiple stages of post-training; the simplest being supervised fine-tuning (SFT). Here, the loss function stays the same, but the dataset is swapped out. Usually to make a chatbot, you would use a dataset that has many example conversations of the type [system prompt, user prompt, ideal assistant response, user prompt, ideal assistant response, ...]. For each of these example conversations, you don't just give the token sequence, but also a mask that tells the training procedure to only train the model to predict the ideal assistant responses. The system prompt might contain a variety of instructions which are following in the ideal assistant responses. After SFT, the model will retain the knowledge learned in pretraining, but will be able to hold a conversation and follow instructions. In addition to SFT, many modern models are trained with various flavors of reinforcement learning (RL). That's a fascinating topic for another day.

### Tokenization
Prior to feeding text into an LLM, it is "tokenized" meaning converted to a sequence of integers representing word chunks. The LLM outputs tokens, which are detokenized to turn the sequence back into readable text before you get to see it. Many of the stranger quirks of today's language models can be explained by understanding how tokenization works. I recommend Andrej Karpathy's YouTube video on tokenization to learn more. You don't need to understand this to use LLMs effectively.

### Sampling and Context
As discussed, at its core the LLM generates a probability distribution over what token comes next, given the previous tokens. We sample from this probability distribution with some sampling rules to get the next token. The previous tokens are known as the context, which has some maximum length. The process to generate a sentence, paragraph, or document requires successive passes through the model, each time taking the entire previous context (including what the model has already generated) as its input. Given that the model makes a prediction based on text sampled from its own previous predictions, LLMs are said to be "autoregressive".

### Thinking Models
Some LLM labs have made so-called "thinking" or "reasoning" models such as DeepSeek R1, OpenAI o1. These generate tokens which are typically hidden from you by default. The hidden tokens give the model a chance to spend computation prior to starting the final response. These thinking models were trained with RL, kind of letting the model teach itself to think, rewarding it for correct answers.  

## Third-party Services

LLMs are expensive both at training time and at inference time. 
> If you're not paying for the product, you are the product. — Common aphorism traced back to at least 1973

Please familiarize yourself with whether the service you're using will sell or train future models on your data. My work rarely has important secrets or privacy concerns, but I am aware that if I use ChatGPT, OpenAI gets to have that prompt and Anthropic does not. I'm ok with that, but if you're not, there are often settings you can choose that tell the model provider not to train on your data. For example, ChatGPT has "Improve the model for everyone" that you can switch to "Off". Cursor's documentation reads: *If you enable "Privacy Mode" in Cursor's settings: zero data retention will be enabled, and none of your code will ever be stored or trained on by us or any third-party.*

### LLM Services I Currently Use
  - Cursor Pro (20\$/month or 192\$/year)
  - ChatGPT Plus (20\$/month)
  - Claude Free
  - Google AI Studio (aistudio.google.com)
  - Chatbot Arena (lmarena.ai)
  - cosmosage (cosmosage.online)

### Other recommendations
 - Cursor Free 
 - VS Code + GitHub CoPilot Pro (10\$/month but free for educators/students)
 - ChatGPT Free (with an account)
 - Windsurf Pro (15\$/month)
 - Windsurf Free

Broadly, LLM-assisted coding breaks into two possible workflows. One is to use a web-based chat interface such as ChatGPT or Claude and to copy-paste your code in and out of the browser window. The other is to use an IDE with LLM features built in. I **highly** recommend the latter workflow, choosing one of Cursor, Windsurf, or VS Code + GitHub CoPilot. I currently like Cursor, so let's focus on it for now. That being said, this is a volatile field, so I'm ready to switch tooling at the drop of a hat.

## Cursor-specific Features

Cursor is a VS Code fork with added AI features. It has most of the VS Code features like VS Code extensions and connecting to remote servers via SSH or Remote Tunnel. 

### Cursor Tab
Cursor Tab is autocomplete on steroids. It not only suggests how to complete the current line of code you're writing, it also
 - Suggests moving your cursor (hit Tab)
 - Suggests multiple lines (hit Tab)
 - Suggests fill-in-the-middle completions (hit Tab)
 - Suggests multiple fill-in-the-middle completions at the same time (hit Tab)

One common workflow I use is to write a brief description of what I want as a comment and then just tab-tab-tab the code into place.

Another example of where Cursor Tab comes in handy is to change a variable name. It'll suggest changing that variable name further down in your code (yes, with Tab).

### AI Pane
The Cursor IDE comes with several customizable panes. My window usually has a file explorer pane on the left, a terminal pane on the bottom, and an AI pane on the right. The AI Pane can be used in three modes, with a number of possible LLMs. Let's first cover the three modes:
 - **Ask**: This is similar to typing into a web-based chatbot, except that the model is given some of your codebase in its context, including the code you're currently working on. You will get an answer to the question you asked, perhaps with suggested code.
 -  **Edit**: Similar to **Ask**, but Cursor will go ahead and apply any suggested code to your codebase. This will neatly display the difference, allowing you to conveniently accept/reject all of or parts of the suggestion. This "apply" functionality is what convinced me to switch from copy-paste workflow to Cursor.
 - **Agent**: Similar to **Edit** but the LLM is given even more autonomy. The agent might search the web, grep the codebase, make changes to the code, run the code itself, iterate, etc. At your own risk, you can enable YOLO mode to allow the agent to run terminal commands without your approval. I use YOLO mode, but I have seen social media posts where the agent will do something crazy like `rm` the codebase.

Now, in Cursor you have some limited control over how to populate the LLM's context window. Cursor isn't clear on exactly what tokens it's handing to the LLM, but it lets you reference context with the `@` symbol. For example, you can `@foo.py` to tell the LLM that you are referring to that file or `@web` to suggest it search the web. In addition, you can add rules either globally or per project (in `.cursor/rules`) to be included with your queries. You can put preferences here that you always want the model to follow. For example, mine tells the model not to use type hints in python and to avoid breaking code out into unnecessary one-time-use functions. If you have some quirky preference like non-standard indentation or having the model write in a certain style, here's where to specify it.

Finally, there is a model selection dropdown. New and better models are being released frequently, so the following list will fall out of date very soon, but here's a snapshot of the models I'm using at the moment:
 - **claude-3.7-sonnet** is a safe bet for general use
 - **claude-3.7-sonnet-thinking** can sometimes solve problems that claude-3.7-sonnet cannot. I recommend it as a second pass if you can tell what you are asking is too complicated for claude-3.7-sonnet
 - **o3-mini** is strong for the kinds of problems you might get as homework in undergraduate-level physics
 - **gpt-4.5-preview** feels the "smartest" to me at the moment, but I don't use it through Cursor due to pricing

## What LLMs are good at and common pitfalls
 - If you're doing a boring, repetitive, or easy coding task, an LLM is likely to do a good job out of the box and save you time and effort. 
 - For medium-difficulty and hard tasks, it really helps the LLM if you specify *exactly* what you want. Give precise requirements. If you're unhappy with the response, try breaking the problem into smaller/simpler subproblems. One thing that I find helpful (for myself as well as for the LLM) is to write a software requirements specification before I start the project. In Cursor, I include it with `@srs.md`.
 - Don't put too much trust in the LLM. For one-off tasks, you can often verify the outputs of the code and trust that the code is likely doing what you want. For code that will be reused in the future, or code that you are sharing with others, read the code yourself and make sure you understand what it's doing. LLMs will sometimes hallucinate or misunderstand things in ways a human never would. **Do not commit code you haven't read.**
 - Don't argue with the LLM. It's not going to remember or learn from it and it certainly won't help your mood.
 - New task? Start a new conversation.
 - Many of your tasks are likely too difficult for the model. Just accept that and do it yourself :) The LLM can still help with line-by-line assistance and some of the side tasks such as making plots, writing unit tests, helping clarify an obscure error message, etc. 
 - Prompt engineering (details of how to write your query) has come to matter less and less as time goes on. In 2023 and into 2024, you could say magic phrases like "Think step by step." or "You are an extremely intelligent physicist and programmer." and improve model performance. As post-training has matured, these performance gains have been baked in. Just ask for what you want. If the model misunderstands your intent, I recommend rephrasing your query to be more precise and starting over (by editing the original prompt or starting a new conversation) rather than correcting the model.
 - Rarely, I have a task that benefits from very long context. Google's models currently excel at long context, so I use their AI Studio for such tasks.
 - The term "vibe coding" refers to writing only the instructions for the LLM and blindly using the result, interacting only indirectly with the code by asking the LLM for modifications. I don't recommend this for serious scientific programming, right now.
