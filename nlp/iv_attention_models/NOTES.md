# [Natural Language Processing with Attention Models](https://www.coursera.org/learn/attention-models-in-nlp?specialization=natural-language-processing)

## 4.1. Neural Machine Translation

### Seq2Seq

A Google model introduced in 2014.

- Inputs and outputs can have different lengths.
- LSTMs and GRUs are used to avoid vanishing and exploding gradients.

**The information bottleneck:** The model's shortcoming is that longer sequences can be problematic though because seq2seq relies on a fixed-length memory.

Therefore, as sequence size increases, model performance decreases.

### Seq2Seq with [Attention](https://arxiv.org/abs/1706.03762)

The sequential nature of models you learned in the previous course (RNNs, LSTMs, GRUs) does not allow for parallelization within training examples, which becomes critical at longer sequence lengths, as memory constraints limit batching across examples.

In other words, if you rely on sequences and you need to know the beginning of a text before being able to compute something about the ending of it, then you can not use parallel computing. You would have to wait until the initial computations are complete.

This is not good, because if your text is too long, then (1) it will take a long time for you to process it and (2) you will lose a good amount of information mentioned earlier in the text as you approach the end.

*Attention allows you to focus on the parts that matter more, and helps with the information bottleneck.*

### Queries, Keys, Values, and Attention

Scaled dot-product attention:

- Scaled using the root of the key vector size
- Scaled scores converted to weights using the softmax function such that the weights for each query sum to one

$$
Context vector = \text{softmax}(\frac{QK^T}{\sqrt{d_k}})V
$$

When the attention mechanism assigns a higher attention score to a certain word in a sequence, then the next word in the decoder's output will be more strongly influenced by this word than by other words in the sequence; i.e. *the attention mechanism matches queries and keys to weigh values. The output of this process is used to select the most likely output word.*

Because attention is just two matrix multiplications and a softmax, it is computationally very efficient.

```python
def softmax(x, axis=0):
    """ Calculate softmax function for an array x

        axis=0 calculates softmax across rows which means each column sums to 1
        axis=1 calculates softmax across columns which means each row sums to 1
    """
    y = np.exp(x)

    return y / np.expand_dims(np.sum(y, axis=axis), axis)

def calculate_weights(queries, keys):
    """ Calculate the weights for scaled dot-product attention"""
    dot = np.matmul(queries, keys.T) / np.sqrt(keys.shape[1])
    weights = softmax(dot, axis=1)

    assert weights.sum(axis=1)[0] == 1, "Each row in weights must sum to 1"

    return weights

def attention_qkv(queries, keys, values):
    """ Calculate scaled dot-product attention from queries, keys, and values matrices """
    weights = calculate_weights(queries, keys)

    return np.matmul(weights, values)
```

### [Teacher Forcing](https://towardsdatascience.com/what-is-teacher-forcing-3da6217fed1c)

Teacher forcing uses the actual output from the training dataset at time step $y^{(t)}$ as input in the next time step $X^{(t+1)}$, instead of the output generated by your model.

### Neural Machine Translation with Attention

### BiLingual Evaluation Understudy (BLEU) Score

```(Sum of unique n-gram counts, overlapping in the candidate and reference) / (Total # of n-grams in the candidate)```

The most widely used evaluation metric for machine translation.

Compares candidate translations to reference (human) translations. The closer to 1, the better; the closer to 0, the worse.

Downsides: A model that always outputs common words will do great, because BLEU score does not consider

- semantic meaning
- sentence structure

### Recall-Oriented Understudy for Gisting Evaluation (ROUGE-N) Score

Precision is defined as:

```(Sum of overlapping unigrams in model and reference)/(total # of words in model)```

Recall is defined as:

```(Sum of overlapping unigrams in model and reference)/(total # of words in reference)```

Compares candidates with reference (human) translations. There are multiple variations of this metric.

$$
F1 = 2 \times \frac{Precision \times Recall}{Precision \times Recall}
$$

$$
F1 = 2 \times \frac{BLEU \times ROUGE-N}{BLEU \times ROUGE-N}
$$

### Sampling and Decoding

**Greedy decoding** selects the most probable word at each step; although the best word at each step may not be the best for longer sequences.

**Random sampling** is often a little too random for accurate translation. Solution: Assign more weight to more probable words, and less weight to less probable words.

**Temperature** can control for more or less randomness in prediction:

- Lower temperature setting: More confident, conservative network
- Higher temperature setting: More excited, random network

### Beam Search

Since the most probable translation is not the one with the most probable word at each step, *calculate the probability of multiple possible sequences.*

Beam width $B$ determines the number of sequences you keep.

Donwsides:

- Penalises long sequences, so should normalise by the sentence length
- Computationally expensive and consumes a lot of memory

### Minimum Bayes Risk

Minimum Bayes Risk achieves better performance than random sampling and greedy decoding:

1. Generate several candidate translations
2. Assign a similarity to every pair using a similarity score (such as ROUGE)
3. Select the sample with the highest average similarity

$$
\argmax_E{\frac{1}{n} \sum_{E'}{ROUGE(E, E')}}
$$

## 4.2. Text Summarisation

Issues with Seq2Seq:

- Loss of information
- Vanishing gradients

Advantages of Transformers:

- With transformers, the vanishing gradient problem isn't related with the length of the sequences because we have access to all word positions at all times.
- Transformers are able to take more advantage from parallel computing than other RNN architectures.
- Even RNN architectures like GRUs and LSTMs don't work as well as transformers for really long sequences.

### Transformers vs RNNs

RNNs are sequential - parallel computations aren't possible with this architecture.

Vanishing gradient problems and loss of information arise with longer sequences. LSTMs and GRUs address this problem but also lose information at some point.

Attention offers a solution to theses problem.

Transformers do not use RNNs, LSTMs, etc.; only attention - *attention is all you need*.

- The Encoder provides a contextual representation of each item in the input sequence.
- The Decoder

### Masked Self Attention

There are three main types of attention:

- Encoder-Decoder attention: Queries from one sentence, keys and values from another
- Self-attention: Queries, keys and values come from the same sentence
- Masked self-attention: Queries, keys and values come from the same sentence; queries don't attend to future positions, i.e. weights assigned to future positions are equal to zero

In self-attention, queries and keys come from the same sentence.

In masked self-attention, queries cannot attend to the future.

### Multi-Head Attention

Multi-headed models attend to information from different representations:

Scaled dot-product attention multiple times in parallel, and concatenate results.

Usual choice of dimensions: $d_k = d_v = d_{model} / h$ ($d_{model}$ is the embedding size)

### Transformer Decoder

Three main layers:

- Decoder
- Feed-forward blocks
- Cross-entropy loss

Between them, there is an attention layer that helps the decoder focus on the relevant parts of the input sentence.

A residual connection around each attention layer followed by a layer normalisation step in the decoder network speeds up training, and significantly reduces the overall processing time.

### Transformer Summariser

The structure of the text input when implementing a summarisation task is: `ARTICLE TEXT <EOS> SUMMARY <EOS> <pad>`

1. Provide article
2. Generate summary word-by-word until the final `EOS`
3. Pick the *next word* by random sampling each time you get a different summary

A weighted loss is optimised.

## 4.3. Question Answering

### Transfer Learning in NLP

- Reduce training time
- Improve predictions
- Small datasets

#### Finetuning

Finetuning means taking existing weights and tweaking them a little bit to get a desired output, usually better results, on some specific task.

There are two types of fine-tuning:

- **Feature-based:** Extract features from a previous layer of the model as embeddings, and feed these feature embeddings to the new model.
- **Fine-tuning:** A finetuning **adapter layer** allows you to add a new layer and then you only fine-tune this new layer.

### ELMo, GPT, BERT, T5

In CBOW, you want to encode a word as a vector. To do this we used the context before the word and the context after the word and we use that model to learn and create features for the word. CBOW however uses a fixed window C (for the context).

What ElMo does is, it uses a bi-directional LSTM, which is another version of an RNN and you have the inputs from the left and the right.

Then Open AI introduced GPT. GPT unfortunately is uni-directional but it makes use of transformers. Although ElMo was bi-directional, it suffered from some issues such as capturing longer-term dependencies. Transformers help with that, but since GPT was still unidirectional, BERT was then introduced which stands for the Bi-directional Encoder Representation from Transformers. Last but not least, T5 was introduced which makes use of transfer learning and uses the same model to predict on many tasks. Here is an illustration of how it works.

| CBOW | ELMo | GPT | BERT | T5 |
| ---- | ---- | --- | ---- | -- |
| Context window | Full sentence | Transformer: Decoder | Transfomer: Encoder | Transformer: Encoder - Decoder |
| FFNN | Bi-directional context | Uni-directional context | Bi-directional context | Bi-directional context |
|  | RNN |  | Multi-mask | Multi-task |
|  |  |  | Next sentence prediction |  |

### [Bi-Directional Encoder Representations from Transformers (BERT)](https://arxiv.org/abs/1810.04805)

BERT is a multi-layer bi-directional transformer with positional embeddings.

BERT_base:

- 12 layers (12 transformer blocks)
- 12 attention heads
- 110 million parameters

It is designed to pretrain deep bi-directional representations from unlabeled text.

The idea is that as it is trained bidirectionally it may have a deeper sense of context and language flow than unidirectional models.

There are two steps in the BERT framework: pre-training and fine-tuning. During pre-training, the model is trained on unlabeled data over different pre-training tasks.  For fine tuning, the BERT model is first initialized with the pre-trained parameters, and all of the parameters are fine-tuned using labeled data from the downstream tasks.

BERT pre-training with **Masked language modelling (MLM):** Choose 15% of the tokens at random:

- mask them 80% of the time
- replace them with a random token 10% of the time
- keep as is 10% of the time
- with the goal of predicting the masked token
- using cross-entropy loss over $V$ classes

There could be multiple masked spans in a sentence.

The input embeddings are the sum of the token embeddings, the segmentation embeddings and the position embeddings.

The input embeddings: you have a CLS token to indicate the beginning of the sentence and a sep to indicate the end of the sentence

The segment embeddings: allows you to indicate whether it is sentence a or b.

Positional embeddings: allows you to indicate the word's position in the sentence.

| BERT Objective | Multi-Mask LM | Next Sentence Prediction |
| --- | --- | --- |
| Loss: | Cross-Entropy Loss | Binary Loss |

### T5

Encoder / decoder architecture with 12 transformer blocks each, and 220 million parameters, previously trained in a multi-tasking mix.

T5 makes use of transfer learning, and the same model could be used for several applications. This implies that other tasks could be used to learn information beneficial for different tasks.

### Multi-Task Training Strategy

The main objective of the multitasking training strategy is to improve the performance of the various tasks by learning them together.

It is possible to formulate most NLP tasks in a “text-to-text” format – that is, a task where the model is fed some text for context or conditioning and is then asked to produce some output text. This framework provides a consistent training objective both for pre-training and fine-tuning. Specifically, the model is trained with a maximum likelihood objective (using “teacher forcing” ) regardless of the task.

#### Training data strategies

- **Examples-proportional mixing:** sample in proportion to the size of each task’s dataset

- **Temperature scaled mixing:** adjust the “temperature” of the mixing rates. This temperature parameter allows you to weight certain examples more than others. To implement temperature scaling with temperature T, we raise each task’s mixing rate rm to the power of 1⁄T and renormalize the rates so that they sum to 1. When T = 1, this approach is equivalent to examples-proportional mixing and as T increases the proportions become closer to equal mixing

- **Equal mixing:** In this case, you sample examples from each task with equal probability. Specifically, each example in each batch is sampled uniformly at random from one of the datasets you train on.

#### Fine-Tuning

- Gradual unfreezing
- Adapter layers

### General Language Understanding Evaluation (GLUE) Benchmark

GLUE contains

- a collection of resources for training, evaluating and analysing natural language understanding (NLU) systems
- datasets for different genres, and of different sizes and difficulties

The GLUE bench mark is used for research purposes, it is model agnostic, and relies on models that make use of transfer learning.

### Question Answering

- Load a pre-trained model
- Process data to get the required inputs and outputs: "question: Q context: C" as input and "A" as target
- Fine tune your model on the new task and input
- Predict using your own model

### HuggingFace

**Pipelines:**

1. Pre-processing the inputs
2. Running the model
3. Post-processing the outputs

**Checkpoints:** Set of learned parameters for a model using a training procedure for some task.

```python
!pip install transformers datasets torch

from transformers import pipeline

# The task "question-answering" will return a QuestionAnsweringPipeline object
question_answerer = pipeline(task="question-answering", model="distilbert-base-cased-distilled-squad")

context = """
Tea is an aromatic beverage prepared by pouring hot or boiling water over cured or fresh leaves of Camellia sinensis,
an evergreen shrub native to China and East Asia. After water, it is the most widely consumed drink in the world.
There are many different types of tea; some, like Chinese greens and Darjeeling, have a cooling, slightly bitter,
and astringent flavour, while others have vastly different profiles that include sweet, nutty, floral, or grassy
notes. Tea has a stimulating effect in humans primarily due to its caffeine content.

The tea plant originated in the region encompassing today's Southwest China, Tibet, north Myanmar and Northeast India,
where it was used as a medicinal drink by various ethnic groups. An early credible record of tea drinking dates to
the 3rd century AD, in a medical text written by Hua Tuo. It was popularised as a recreational drink during the
Chinese Tang dynasty, and tea drinking spread to other East Asian countries. Portuguese priests and merchants
introduced it to Europe during the 16th century. During the 17th century, drinking tea became fashionable among the
English, who started to plant tea on a large scale in India.

The term herbal tea refers to drinks not made from Camellia sinensis: infusions of fruit, leaves, or other plant
parts, such as steeps of rosehip, chamomile, or rooibos. These may be called tisanes or herbal infusions to prevent
confusion with 'tea' made from the tea plant.
"""

result = question_answerer(question="Where is tea native to?", context=context)
print(result["answer"])
```

**Datasets:** Task-specific datasets or fine-tuning.

**Tokenizers:** Help with pre-processing data before training.

Pre-defined **evaluation metrics** like BLEU and ROUGE.

## 4.4. Chatbot

### Transformer Issues

The computational time increases quadratically with the sequence length L, which becomes problematic when dealing with longer sequences. Additionally, the time increases linearly with the number of layers:

- Attention on a sequence of length $L$ takes $L^2$ time and memory.
- $N$ layers take $N$ times as much memory.

The more layers a Transformer has, the more memory it needs in training. This is because you need to save the forward pass activations for backprop.

A way to overcome this memory constraint is to recompute activations.

### LSH Attention

What does Attention do? Select nearest neighbours (K, Q) and return corresponding V.

Attention computes $d(q, k_i)$ for $i$ from $1$ to $n$, which can be slow.

*Faster* approximation uses **locality sensitive hashing (LSH):** It allows us to not have to compare each query with each key. Instead we only compare the vectors that are found in the same bucket.

| Standard Attention | LSH Attention |
| --- | --- |
| $A(Q, K, V) = softmax(QK^T)V$ | - Hash $Q$ and $K$ |
|  | - Standard attention within same-hash bins |
|  | - Repeat a few times to increase probability of key in the same bin |

*The more hashes you have the more accurate your model is, but the slower it is.*

LSH Attention can take advantage of parallel computing by sorting and chunking the LSH buckets. Then you let each chunk attend within itself and adjacent chunks.

### Motivation for Reversible Layers: Memory

Reversible layers allow you to reconstruct activations. As a result, you don't need to save them, which saves memory.

### Reversible Residual Layers

Reversible residual layers allow you to reconstruct the forward layer from the end of the network. Usually you have two similar branches in the network that you use to compute the network.

Standard Transformer:

$$
y_a = x + Attention(x)
$$

$$
y_b = y_a + FF(y_a)
$$

Reversible:

$$
y_1 = x_1 + Attention(x_2)
$$

$$
y_2 = x_2 + FF(y_1)
$$

Recompute $x_1, x_2$ from $y_1, y_2$:

$$
x_1 = y_1 - Attention(x_2)
$$

$$
x_2 = y_2 - FF(y_1)
$$

### [Reformer - The Reversible Transformer](https://arxiv.org/abs/2001.04451)

Reformer is a transformer model designed to be memory efficient so it can handle very large context windows of upto 1 million words.

It utilises

- LSH attention: to reduce the complexity of attending over long sequences
- Reversible layers: to more efficiently use the memory available
