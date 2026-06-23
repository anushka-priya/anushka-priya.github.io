+++
date = '2026-06-23T14:11:50+05:30'
draft = false
title = 'Attention Is All You Need- But What is Attention?'
+++

## Introduction- the problem with RNNs

The 2017 paper ["Attention Is All You Need"](https://arxiv.org/abs/1706.03762) quite literally transformed neural networks. Every chatbot, every LLM you interact with today: GPT, Gemini, Claude finds its roots in this elegant mechanism named attention.

But why did we require attention to begin with?

To understand why attention matters, we need to go back to what came before it. The dominant models for sequential tasks like language translation, text generation, and speech recognition were 
Recurrent Neural Networks (RNNs). The idea behind RNNs was simple, process one token at a time, left to right, carrying information forward through a hidden state.

The problem? RNNs had the memory of a goldfish.

This made learning long range dependencies extremely difficult. Think about reading a complex piece of literature, you simply cannot make sense of what's happening on page 50 without remembering the intricate details from page 1. If we struggle with this ourselves, how could we expect an RNN to handle it flawlessly?

By the time an RNN reached word 50 of a sentence, the information from word 1 had largely vanished due to the multiple computations that came in between.Gate mechanisms and cell state were used in LSTMs to mitigate this issue. However, it wasn't completely solved and since each word had to wait for the previous one to finish processing, you couldn't parallelize the computation, which made the training process extremely  slow.

Attention mechanisms had actually existed before this paper, introduced by [Bahdanau et al.](https://arxiv.org/abs/1409.0473) (2014) in "Neural Machine Translation by Jointly Learning to Align and Translate" but they remained limited only as a complement to RNNs, not as a replacement.

This paper changed that entirely. Vaswani et al. proposed a model architecture called **Transformers** based solely on attention, throwing out recurrence altogether.

<figure style="text-align: center;">
  <img src="/images/transformer-architecture.jpeg" alt="Transformer architecture diagram" style="max-width: 100%;">
  <figcaption style="text-align: center; font-style: italic; margin-top: 8px;"><strong>Figure drawn by me, referencing the original paper to illustrate the transformer architecture and its components.</strong></figcaption>
</figure>

## Embedding

Before we can apply any attention mechanism, we need to address a fundamental question: how does a model even read words?

Computers don't understand language the way we do. Every word needs to be converted into something a neural network can actually work with- numbers.

This is where embeddings come in. An embedding is a vector of fixed dimensions that represents a word's meaning numerically. In the Transformer, every word is mapped to a vector of dmodel = 512 numbers. This mapping is learned during training, the model figures out the best numerical representation for each word based on the task. In the Transformer, the entire input sentence becomes a matrix of embeddings: one row per word, each row a 512-dimensional vector.

This matrix is the raw input to the Transformer, but before it goes any further, something important is added to it.

Position.

## Positional Encoding

Attention processes all words simultaneously. This is exactly what makes it powerful but it also means the model has absolutely no idea what order the words appear in. If you jumble an entire sententce- the input would still look unchanged to a transformer, however the semantic meaning will change completely. 

RNNs never had this problem, because they processed words sequentially, preserving the position. But the Transformer threw away sequential processing entirely. So it needs another way to know where each word is present in the sentence.

The solution is positional encoding: a vector of the same dimension as the word embedding (dmodel = 512) that is added directly to the embedding before anything else happens:

Final Input = Word Embedding + Positional Encoding

The authors in the paper used sinusoidal functions of different frequencies to give every word a distinct position. Using sinusoids also enabled the model to relate the positions between any two words as well as generalize to sequence lengths longer than those seen during training.

## Understanding Q, K, V- the Heart of Attention

The three key components of the attention mechanism are the **Query (Q)**, **Key (K)**, and **Value (V)**. Let me explain it to you intuitively.

Since I mentioned literature earlier- let's keep at it.

Suppose you walk into a bookstore looking for a specific genre. You browse through the shelves, scanning titles and covers to get a brief sense of what each book is about. Eventually, one title catches your eye as it seems relevant, interesting, exactly what you were looking for. You pick it up and read it cover to cover.

In this analogy:
- The Query is what you were looking for: your genre, your question walking into the store.
- The Key is what each book communicated about itself through its title, cover i.e. how relevant it seemed to your query.
- The Value is the actual content of the book: what you got out of it once you decided it was worth reading.

Turns out, sometimes you do have to judge a book by its cover and in attention, that's exactly the point.

Consider the sentence:

"The trophy didn't fit in the bag because it was too big"

The most interesting word here is "it" — what does "it" refer to? The trophy or the bag?

As humans, we resolve this ambiguity in milliseconds, it comes completely intuitively to us. Much like how we can classify an image in seconds just by glancing at it, our brains are wired to extract context and meaning effortlessly. The challenge in deep learning has always been: how do we get a model to do what comes naturally to us? Through attention.

When processing "it", its Query is asking "which words in this sentence are most relevant to 
understanding what it refer to?"

"trophy" responds with a very high Key match because the context "too big to fit" points directly 
to the trophy

"bag" responds with a moderate Key match as it's related but not the answer.

"big" also responds with a high Key match.

The Values of "trophy" and "big" are pulled strongly into "it's" final representation, giving the model enough context to correctly resolve that "it" = "trophy", not "bag".

This is the core idea of self-attention: every word gets to look at every other word simultaneously and decide how much context to borrow from each of them, regardless of how far apart they are in the sentence.

Technically, Q, K, and V are learned linear projections of the embeddings obtained by multiplying the input embedding by three separate learned weight matrices:

Q = X · Wq
K = X · Wk
V = X · Wv

Where X is the input embedding and Wq, Wk, Wv are weight matrices learned during training. This means the model learns through backpropagation, the best way to project each word into its Query, Key, and Value representations for the task at hand.

## Scaled Dot Product Attention

Now that we've set up the base, let's take a look at computing attention itself!

The attention function is defined as:

{{< figure src="/images/attention-equation.png" alt="Attention formula" caption="From Vaswani et al. (2017)" >}}

The first operation is a matrix multiplication between Q and the transpose of K. This computes a dot product between every query and every key measuring how similar, or how relevant, each key is to each query.

The dot product is a measure of similarity. Two vectors pointing in the same direction produce a 
high dot product, while two unrelated vectors produce a low one.

The result is a matrix of raw similarity scores:one score for every pair of words in the sentence:

For example:

        "trophy" "didn't" "fit" "bag" "it" "big"
"it"   [  0.8      0.1    0.3   0.4  0.2   0.7  ]

High scores for "trophy" and "big" are exactly the words that resolve what "it" refers to.

**Scaling factor**: *Why do we divide by √dk?*

When dk is large, let's say 512, the dot products between queries and keys grow very large in magnitude. If each component of q and k has mean 0 and variance 1, their dot product has variance dk. So with 
dk = 512, your dot products can get enormous.

These large values are fed into softmax next. When you feed softmax a set of very large numbers, one value completely dominates and the model pays 100% attention to one word and completely ignores everything else. Even worse, in this region of softmax, the gradients become extremely small. We're back to the vanishing gradient problem we were trying to escape from in the first place.

Therefore the fix, Divide by √dk.

Why √dk specifically? Because if the variance of the dot product is dk, dividing by √dk brings the 
variance back to exactly 1, keeping the scores in a stable, well-behaved range regardless of how 
large dk is.

The authors also note that for larger values of dk, additive attention outperforms unscaled dot product attention further justifying the scaling factor.

After scaling, **softmax** is applied to convert the raw scores into a probability distribution — values between 0 and 1 that sum to 1. This gives us the attention weights or how much each word pays attention to every other word in the sentence- providing the context of the sentence.

Finally, the attention weights are multiplied by the Value matrix V. This produces a weighted sum of all value vectors and words with high attention weights contribute more to the output.
The result is a new, context-aware representation of all the words.

## Multi-head Attention

Single attention is powerful but it has a fundamental limitation.
With a single attention head, the model performs one attention function over the full dmodel = 512 
dimensional space. Every word attends to every other word but through a single, unified 
perspective. All the complexity of language gets compressed into one set of attention weights.

As the paper puts it directly:

"Multi-head attention allows the model to jointly attend to information from different representation subspaces at different positions. With a single attention head, averaging inhibits this."

The solution is to project Q, K, and V into h = 8 different lower-dimensional subspaces, run attention independently in each, and then combine the results:

headi = Attention(Q·Wqi, K·Wki, V·Wvi)

MultiHead(Q,K,V) = Concat(head1,...,head8) · Wo

Each head receives its own learned projection matrices Wqi, Wki, Wvi — meaning each head learns 
to project the input into a completely different subspace before computing attention. The model learns which subspace is useful for which type of relationship entirely through backpropagation.

***Representation Subspaces***

Think of the full 512-dimensional embedding space as containing all information about a word such as its meaning, its grammatical role, its position, its relationship to other words.

Different dimensions of this space encode different types of information. A single attention head 
operating on all 512 dimensions simultaneously must find one set of attention weights that 
satisfies all of these at once. However, by projecting into 8 separate 64-dimensional subspaces, each head gets to focus on a distinct portion of the representational space. One head 
might find that its 64 dimensions are most useful for tracking grammatical dependencies. Another 
might specialize in semantic similarity. Another in long range dependencies.

This was directly observed by the authors. The different attention heads clearly learned to perform distinct linguistic tasks: some tracking long distance dependencies across many tokens, others sharply resolving pronoun references, others capturing syntactic structure.

### The Computational Trick

Single head attention: 1 head × dk = 512 dimensions
Multi-head attention:  8 heads × dk = 64 dimensions

Total computation: identical.

By reducing each head's dimensionality by a factor of h, the total computational cost remains the same as single-head attention with full dimensionality while learning much more complex representations.

The outputs of all 8 heads are concatenated into a hdv = 512 dimensional vector and projected back to dmodel via the learned output matrix Wo producing a final representation that synthesizes 8 independent perspectives into one unified, richly contextualised output.

## Conclusion

The Transformer didn't just improve on RNNs, it replaced them entirely with one idea.
The result? BERT, GPT, Claude, Gemini, all of them, at their core, are doing exactly what this paper described in 2017.

***Attention really is all you need.***

***And now you know why.***
