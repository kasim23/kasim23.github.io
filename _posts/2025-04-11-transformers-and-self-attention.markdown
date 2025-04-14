---
layout: post
comments: true
title:  "Understanding Self-Attention and the Unreasonable Effectiveness of a Transformer"
excerpt: "We'll understand the self-attention mechanism that powers Transformer neural networks and go through an example by hand to understand these concepts. You can follow along!"
date:   2025-04-11 11:00:00
mathjax: true
---

There’s something almost magical about how Transformers understand human language. Over the past few years, Large Language Models (LLMs) have captured a lot of attention (pun intended), especially with breakthroughs like OpenAI’s GPT-2 and ChatGPT. Recently, during a research meeting, my colleague shared something really interesting: she borrowed slides from another lab member who had presented on how language models work, and then walked her parents through it. To her surprise, by the end, they had a pretty solid understanding of how all this "magic" happens. Inspired by that, I want to take you behind the scenes of how these models operate, breaking it down in a way that's easy to grasp, even for those with no background in the field.

### Timeline and Foundational Context
To truly understand how a Large Language Model works, it’s essential to first explore the events that led to the development of the Transformer architecture by Google Research in 2017. [Attention Is All You Need](https://arxiv.org/pdf/1706.03762), the groundbreaking paper behind this innovation, marked a pivotal shift in natural language processing. Prior to the Transformer, we relied on earlier techniques like Bag of Words and Word2Vec, which laid the groundwork for understanding language in computational terms. While these methods were valuable, they lacked the ability to capture the nuanced relationships between words and sentences as effectively as the Transformer does today. However, understanding these earlier methods is key to appreciating how far we’ve come in language AI.
<div class="imgcap">
<img src="/assets/transformer/timeline-nlp.png">
<div class="thecap" style="text-align:justify"></div></div>

**Bag of Words** and **Word2Vec** are both methods to represent language in a way that computers can understand. We will walk through both of these methods to get a firm understanding of how words are represented to machines. Let's say you want to represent the following sentences:

- That is a cute dog
- My cat is cute

First, you'd create your vocabulary - essentially, all the unique words in the documents you have. Document 1 has the words "That," "is," "a," "cute," "dog," and Document 2 has "My," "cat," "is," "cute." Your vocabulary will often be smaller than the total number of words across all your documents since you're only picking out the unique words. In this case, the vocabulary looks like this: ["that", "is", "a", "cute", "dog", "my", "cat"]. Now, if we want to represent the sentence "My cat is cute," we'd map it to the vector: `[0, 1, 0, 1, 0, 1, 1]`. The order in this numerical representation is important as it allows us to compare different sentences to one another, in practice this is called the **vector representation** - a list of numerical values that represents the input. In this example, these values are counts and have an explicit meaning: they represent how many times a word in the vocabulary appears in the input. Vector representations in more advanced models typically do not have such an intuitive meaning and have values between 0 and 1.
<div class="imgcap">
<img src="/assets/transformer/bagofwords.png">
<div class="thecap" style="text-align:justify"></div></div>

Let’s walk through how we can create the vocabulary and represent the sentence as a vector in Python:

```python
# Define the sentences
sentence1 = "That is a cute dog"
sentence2 = "My cat is cute"

# Step 1: Create the vocabulary
vocabulary = list(set(sentence1.lower().split() + sentence2.lower().split()))

# Step 2: Create a function to generate the vector representation
def vector_representation(sentence, vocabulary):
    vector = [0] * len(vocabulary)
    for word in sentence.lower().split():
        if word in vocabulary:
            vector[vocabulary.index(word)] += 1
    return vector

# Step 3: Create the vector representation for sentence 2
vector = vector_representation(sentence2, vocabulary)

# Print the results
print(f"Vocabulary: {vocabulary}")
print(f"Vector Representation of '{sentence2}': {vector}")
```

```Vocabulary: ['that', 'is', 'a', 'cute', 'dog', 'my', 'cat']```

```Vector Representation of 'My cat is cute': [0, 1, 0, 1, 0, 1, 1]```

Let’s now explore how more complex word embeddings are created. For the remainder of the article, I will use the terms **vector embeddings** or **word embeddings**, both of which refer to the numerical representations of words. Bag of words although an elegant approach has a flaw, can you guess? It considers language to be nothing more than a literal bag of words and ignores the semantic relation between words. Word2vec was one of the first successful attempts at encoding said semantic relation in word embeddings. I will not go in the details of the word2vec algorithm and training process, that deserves a whole post of itself, however, if you are interested, you can read [The Illustrated Word2Vec By Jay Alammar](http://jalammar.github.io/illustrated-word2vec/). This is a really good resource if you want to dive deep into word2vec.
