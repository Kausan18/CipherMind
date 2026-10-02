# CipherMind — Neural Network Cipher Classification & Decryption Lab

## 1. Project Overview

**Project name:** CipherMind

**One-line description:**  
An educational neural-network platform that first learns to classify classical/toy encryption methods and then uses a second neural network to learn ciphertext-to-plaintext decryption, with a dashboard that makes the training and inference process visible.

### Core pipeline

```text
Encrypted Message
       |
       v
+------------------------+
| Phase 1: NN Classifier |
| Cipher Identification  |
+-----------+------------+
            |
            v
     Detected Cipher
            |
            v
+------------------------+
| Phase 2: NN Decryptor  |
| Ciphertext + Type      |
|          -> Plaintext  |
+-----------+------------+
            |
            v
      Decrypted Text
```

The project is deliberately designed as both:

1. A working classical-cipher analysis/decryption application.
2. A hands-on neural-network learning laboratory.

The goal is not merely to call a machine-learning framework. We will understand the mathematics and mechanics behind neural networks and progressively implement increasingly capable models.

---

# 2. Educational Aim

The primary aim is to understand neural networks and sequence models by building a concrete project from first principles.

By the end of the project, the developer should understand and be able to explain:

- neurons
- weights
- biases
- weighted sums
- activation functions
- forward propagation
- loss functions
- gradients
- backpropagation
- gradient descent
- optimizers
- learning rate
- epochs
- batches
- training/validation/test sets
- overfitting and underfitting
- regularization
- classification
- softmax
- cross-entropy loss
- sequence modeling
- embeddings
- recurrent neural networks
- LSTMs
- attention
- queries, keys and values
- self-attention
- positional information
- Transformers
- sequence-to-sequence learning

The project should therefore be built in a way that makes the underlying concepts visible rather than hiding them behind a high-level API.

---

# 3. Scope and Safety Boundary

CipherMind is an **educational cryptanalysis project**.

## Initial supported domain

The system will work with classical/toy ciphers such as:

- Caesar cipher
- Vigenère cipher
- Affine cipher
- Monoalphabetic substitution

Additional classical ciphers may be added later.

## Data policy

Training data should primarily be generated locally by the project itself:

```text
Plaintext
   |
   v
Classical Cipher
   |
   v
Ciphertext + Ground Truth
```

This gives us complete control over:

- plaintext
- cipher type
- key
- ciphertext
- training labels

## Explicit limitation

This project is not intended to defeat modern real-world cryptographic systems such as:

- AES
- RSA
- ChaCha20
- modern elliptic-curve cryptography
- production authentication/encryption systems

The purpose is to learn neural networks, sequence modeling, and classical cryptanalysis.

---

# 4. Functional Requirements

## FR-1: Dataset Generation

The system must be able to generate synthetic training examples containing:

- plaintext
- cipher type
- key/parameters
- ciphertext

Example:

```text
Plaintext:  HELLOWORLD
Cipher:     Caesar
Key:        3
Ciphertext: KHOORZRUOG
```

## FR-2: Phase 1 Classification

The first neural network must accept a ciphertext representation and predict the cipher type.

Example:

```text
Input:
KHOORZRUOG

Output:
Caesar       96.8%
Vigenère      1.7%
Affine        1.1%
Substitution  0.4%
```

The system should expose:

- predicted class
- confidence/probability distribution
- evaluation metrics

## FR-3: Phase 2 Decryption

The second neural network must learn a sequence transformation:

```text
Ciphertext + Cipher Type
            |
            v
       Plaintext
```

Example:

```text
Ciphertext: KHOORZRUOG
Type:       Caesar

Predicted plaintext:
HELLOWORLD
```

The Phase 2 model will initially be deliberately simple and later upgraded to sequence models.

## FR-4: Dashboard

The dashboard must allow users to:

1. inspect training progress
2. inspect model performance
3. enter ciphertext
4. run Phase 1 classification
5. see the predicted cipher
6. run Phase 2 decryption
7. inspect the final plaintext
8. view useful model/training information

---

# 5. High-Level Architecture

```text
                        CIPHERMIND
                            |
                            v
                   +------------------+
                   |    Dashboard     |
                   +--------+---------+
                            |
              +-------------+-------------+
              |                           |
              v                           v
       Training Interface          Inference Interface
              |                           |
              v                           v
       +-------------+             +-------------+
       | Phase 1 NN  |             | Ciphertext  |
       | Classifier  |             +------+------+
       +------+------+                    |
              |                           v
              |                    +-------------+
              |                    | Phase 1 NN  |
              |                    | Classifier  |
              |                    +------+------+
              |                           |
              |                           v
              |                     Cipher Type
              |                           |
              |                           v
              |                    +-------------+
              |                    | Phase 2 NN  |
              |                    | Decryptor   |
              |                    +------+------+
              |                           |
              |                           v
              |                     Plaintext
              |
              v
       Training Artifacts
       - loss
       - accuracy
       - checkpoints
       - metrics
       - confusion matrix
```

---

# 6. Phase 1 — Cipher Classification

## Objective

Train Neural Network #1 to answer:

> Given a ciphertext, which supported cipher most likely produced it?

Mathematically:

```text
f(C) -> P(cipher_type | C)
```

where:

- `C` = ciphertext
- `P(...)` = predicted probability distribution

## Initial classes

```text
0 = Caesar
1 = Vigenère
2 = Affine
3 = Substitution
```

## Initial input representation

We will experiment with two representations.

### Approach A — Hand-crafted statistical features

Useful features can include:

- ciphertext length
- character frequency
- normalized character frequency
- n-gram statistics
- entropy
- index of coincidence
- repeated-pattern statistics

This approach is useful for learning traditional feature engineering.

### Approach B — Character sequence representation

Map characters to integers:

```text
A -> 0
B -> 1
...
Z -> 25
```

Then feed the sequence to a neural model.

This approach is useful for learning representation learning and sequence models.

## Recommended progression

Start with hand-crafted/statistical features and a simple dense network.

Then experiment with direct character-sequence input.

---

# 7. Phase 1 Model

Initial architecture:

```text
Input Features
      |
      v
Dense Layer
      |
      v
ReLU
      |
      v
Dense Layer
      |
      v
ReLU
      |
      v
Output Layer
      |
      v
Softmax
      |
      v
4 Cipher Classes
```

Starting point only:

```text
Input size: determined by feature set
Hidden layer 1: 64 neurons
Hidden layer 2: 32 neurons
Output: 4 classes
Activation: ReLU
Output activation: Softmax
Loss: Cross Entropy
Optimizer: Adam initially; also study SGD
```

These are starting points, not fixed requirements. Experiments should change them deliberately and record the effect.

---

# 8. Phase 1 Dataset

The dataset will be generated programmatically.

## Suggested initial dataset

Start small enough to iterate quickly:

```text
10,000–50,000 examples
```

distributed across the supported cipher classes.

Do not assume that more data automatically means a better model.

## Plaintext source

Initially use generated/template English sentences or a controlled local corpus that can legally be used for the project.

The generator should vary:

- sentence length
- word selection
- punctuation handling
- capitalization
- repeated characters
- message length

## Cipher parameters

Randomize parameters appropriately.

For Caesar:

```text
shift = random valid shift
```

For Vigenère:

```text
key = random alphabetic key
```

For Affine:

```text
a, b = valid parameters
```

For substitution:

```text
random valid alphabet permutation
```

## Dataset record

A generated record should conceptually contain:

```json
{
  "plaintext": "...",
  "cipher": "caesar",
  "key": 3,
  "ciphertext": "..."
}
```

The exact storage format can be CSV, JSONL, Parquet, or another appropriate format depending on implementation.

---

# 9. Dataset Splitting

Use separate:

```text
Training set
Validation set
Test set
```

Suggested initial split:

```text
70% training
15% validation
15% test
```

The test set must remain isolated until evaluation.

The generator should avoid accidental leakage between splits.

For example, do not simply create multiple almost-identical ciphertexts and distribute them across every split.

---

# 10. Phase 1 Evaluation

Track at least:

- accuracy
- precision
- recall
- F1 score
- confusion matrix
- per-class accuracy

The dashboard should visualize training and validation behavior.

Important questions:

- Does training accuracy rise?
- Does validation accuracy rise?
- Is validation performance substantially worse?
- Which cipher pairs are confused?
- Does increasing model complexity actually help?

---

# 11. Phase 2 — Neural Decryption

## Objective

Train Neural Network #2 to learn:

```text
(ciphertext, cipher_type) -> plaintext
```

This is fundamentally different from Phase 1.

Phase 1 is classification.

Phase 2 is sequence prediction / sequence transformation.

## Example

```text
Input ciphertext:
KHOORZRUOG

Cipher type:
Caesar

Target:
HELLOWORLD
```

The model should learn to generate the target sequence.

---

# 12. Phase 2 Progression

Do not jump immediately to a Transformer.

Build Phase 2 progressively.

## Phase 2A — Simple baseline

Start with a deliberately constrained model for understanding.

```text
Ciphertext representation
        |
        v
Dense / simple sequence representation
        |
        v
Output characters
```

This baseline may be limited. That is useful because it demonstrates why sequence models are needed.

## Phase 2B — RNN

```text
Ciphertext
    |
    v
RNN
    |
    v
Plaintext
```

Learn:

- hidden state
- recurrence
- sequence processing
- vanishing/exploding gradients

## Phase 2C — LSTM

```text
Ciphertext
    |
    v
LSTM
    |
    v
Plaintext
```

Learn:

- cell state
- forget gate
- input gate
- output gate

## Phase 2D — Attention

Introduce attention to understand why the model needs a mechanism to selectively focus on parts of the input.

## Phase 2E — Transformer

Final advanced version:

```text
Ciphertext
    |
    v
Token/Character Embedding
    |
    v
Positional Information
    |
    v
Transformer
    |
    v
Output Projection
    |
    v
Plaintext
```

This phase teaches:

- embeddings
- positional encoding/information
- query
- key
- value
- self-attention
- multi-head attention
- feed-forward networks
- residual connections
- layer normalization
- autoregressive/sequence generation concepts where appropriate

---

# 13. Phase 2 Output Representation

Start with character-level modeling.

Alphabet/token vocabulary can include:

```text
A-Z
space
basic punctuation if supported
special tokens:
<PAD>
<BOS>
<EOS>
<UNK>
```

The model predicts one output token at a time.

For sequence generation:

```text
<BOS>
   |
   v
H
   |
   v
E
   |
   v
L
   |
   v
L
   |
   v
O
   |
   v
<EOS>
```

The exact training strategy will be selected during implementation after the baseline is understood.

---

# 14. Phase 2 Evaluation

Track:

- character-level accuracy
- exact sequence accuracy
- token accuracy where applicable
- validation loss
- test loss
- examples of predicted vs expected plaintext

Example:

```text
Target:
THEQUICKBROWNFOX

Prediction:
THEQUICKBROWNF0X

Character accuracy:
94.1%

Exact match:
NO
```

Exact sequence accuracy is intentionally strict.

---

# 15. Dashboard Requirements

Recommended framework:

**Streamlit**

## Dashboard sections

### Section 1 — Project overview

Show:

- project description
- current model
- supported ciphers
- training status

### Section 2 — Phase 1 training

Display:

- epoch
- training loss
- validation loss
- training accuracy
- validation accuracy
- learning rate
- batch information
- model architecture

### Section 3 — Phase 1 visualization

Display:

- loss curve
- accuracy curve
- confusion matrix
- per-class metrics
- class probabilities

### Section 4 — Phase 2 training

Display:

- sequence loss
- character accuracy
- validation metrics
- training examples
- sample predictions

### Section 5 — Neural network visualization

Where practical, show:

```text
Input
  |
  v
Layer 1
  |
  v
Layer 2
  |
  v
Output
```

For selected neurons, expose:

- weights
- bias
- activation
- gradient information during training where available

Do not attempt to visualize every parameter in a huge model.

### Section 6 — Inference

Input:

```text
Encrypted message
```

Then:

```text
[Analyze & Decrypt]
```

Output:

```text
Detected cipher:
Caesar

Confidence:
96.8%

Decrypted text:
HELLOWORLD
```

Also show the Phase 1 probability distribution.

### Section 7 — Explainability / learning view

Show concise explanations of what the current model is doing.

Example:

```text
Phase 1:
The classifier converts the ciphertext into numerical features,
passes them through learned weighted transformations, and produces
a probability distribution over the four supported cipher classes.
```

---

# 16. Recommended Technology Stack

## Core language

**Python**

Reason:

- excellent ML ecosystem
- simple experimentation
- easy numerical programming
- strong educational value

## Numerical computing

**NumPy**

Use it initially to implement neural-network mechanics manually.

## Deep learning

**PyTorch**

Use after the from-scratch implementation is understood.

PyTorch will become the primary framework for the more advanced Phase 2 models.

## Data

**pandas**

Use for dataset inspection and analysis.

## Evaluation

**scikit-learn**

Use for:

- confusion matrices
- classification metrics
- train/test utilities where appropriate

## Visualization

**Matplotlib**

Use for learning/training plots.

Optionally use Plotly later if it makes the dashboard more interactive.

## Dashboard

**Streamlit**

Use for the interactive application.

## Environment

Recommended:

```text
Python virtual environment
requirements.txt
```

A `pyproject.toml` can be introduced later if the project grows.

---

# 17. Development Philosophy

The most important rule:

> Understand the mechanism before hiding it behind a framework.

Therefore:

```text
NumPy from scratch
        |
        v
Understand forward pass
        |
        v
Understand loss
        |
        v
Understand gradients
        |
        v
Implement backpropagation
        |
        v
Train simple model
        |
        v
Reimplement using PyTorch
        |
        v
Build advanced models
```

Do not start by writing a large Transformer.

---

# 18. Recommended Build Order

## Milestone 0 — Repository and environment

Create:

```text
Python environment
Git repository
requirements.txt
README.md
project structure
```

Verify imports and basic execution.

### Definition of done

A minimal Python program runs successfully.

---

## Milestone 1 — Classical ciphers

Implement encryption/decryption functions for:

```text
Caesar
Vigenère
Affine
Substitution
```

Each implementation should have tests.

### Definition of done

For every supported cipher:

```text
plaintext
 -> encrypt
 -> ciphertext
 -> decrypt
 -> original plaintext
```

---

## Milestone 2 — Dataset generator

Build a generator that creates labeled examples.

### Definition of done

The project can generate a reproducible dataset containing:

```text
plaintext
cipher
key
ciphertext
```

with configurable sample counts.

---

## Milestone 3 — Feature engineering

Implement initial Phase 1 features.

Investigate:

- character frequencies
- normalized frequencies
- entropy
- index of coincidence
- n-grams
- message length

### Definition of done

A ciphertext can be converted into a numerical feature vector.

---

## Milestone 4 — Neural network from scratch

Implement a minimal dense neural network using NumPy.

Required components:

```text
Layer
Dense
ReLU
Softmax
Loss
Backward pass
Optimizer/update
```

### Definition of done

The network can learn a small classification task.

---

## Milestone 5 — Phase 1 classifier

Train the classifier on CipherMind's generated dataset.

Track:

- loss
- accuracy
- validation performance
- confusion matrix

### Definition of done

The model can classify held-out classical-cipher examples.

Do not define a fixed target accuracy before running experiments; record the actual result and investigate failures.

---

## Milestone 6 — Phase 1 PyTorch version

Rebuild the classifier using PyTorch.

Compare:

```text
NumPy implementation
vs
PyTorch implementation
```

### Definition of done

Both versions solve the same classification task and their behavior is understood.

---

## Milestone 7 — Phase 2 baseline

Build the first decryptor.

Start with a constrained problem, such as short messages and a subset of ciphers.

### Definition of done

The model can learn a non-trivial ciphertext-to-plaintext mapping on held-out examples.

---

## Milestone 8 — RNN

Introduce sequence modeling.

### Definition of done

Understand why a recurrent model is more appropriate for variable-length sequential data.

---

## Milestone 9 — LSTM

Replace the basic RNN with an LSTM.

### Definition of done

Understand gates and long-range sequence handling.

---

## Milestone 10 — Transformer

Build or use a small Transformer-based sequence model.

### Definition of done

Understand:

```text
Embedding
Positional information
Q/K/V
Attention
Multi-head attention
Feed-forward layer
Residual connection
Layer normalization
```

---

## Milestone 11 — Dashboard

Integrate:

- Phase 1 training
- Phase 2 training
- plots
- metrics
- model architecture
- inference

### Definition of done

A user can train/view models and enter a ciphertext from the dashboard.

---

## Milestone 12 — Final integrated application

Final pipeline:

```text
User ciphertext
       |
       v
Phase 1 classifier
       |
       v
Cipher type
       |
       v
Phase 2 decryptor
       |
       v
Plaintext
```

### Definition of done

The full workflow works end-to-end on supported classical/toy ciphers.

---

# 19. Proposed Folder Structure

```text
ciphermind/
│
├── README.md
├── PROJECT_SPEC.md
├── requirements.txt
├── .gitignore
│
├── configs/
│   ├── phase1.yaml
│   └── phase2.yaml
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── generated/
│
├── notebooks/
│   ├── 01_cipher_experiments.ipynb
│   ├── 02_neuron_from_scratch.ipynb
│   ├── 03_backpropagation.ipynb
│   ├── 04_phase1_features.ipynb
│   └── 05_phase2_sequence_experiments.ipynb
│
├── src/
│   └── ciphermind/
│       │
│       ├── __init__.py
│       │
│       ├── ciphers/
│       │   ├── __init__.py
│       │   ├── caesar.py
│       │   ├── vigenere.py
│       │   ├── affine.py
│       │   └── substitution.py
│       │
│       ├── data/
│       │   ├── __init__.py
│       │   ├── generator.py
│       │   ├── preprocessing.py
│       │   ├── vocabulary.py
│       │   └── datasets.py
│       │
│       ├── features/
│       │   ├── __init__.py
│       │   ├── frequency.py
│       │   ├── entropy.py
│       │   ├── coincidence.py
│       │   └── ngrams.py
│       │
│       ├── nn_from_scratch/
│       │   ├── __init__.py
│       │   ├── layers.py
│       │   ├── activations.py
│       │   ├── losses.py
│       │   ├── network.py
│       │   └── optimizers.py
│       │
│       ├── phase1/
│       │   ├── __init__.py
│       │   ├── model.py
│       │   ├── train.py
│       │   ├── evaluate.py
│       │   └── predict.py
│       │
│       ├── phase2/
│       │   ├── __init__.py
│       │   ├── baseline.py
│       │   ├── rnn.py
│       │   ├── lstm.py
│       │   ├── transformer.py
│       │   ├── train.py
│       │   ├── evaluate.py
│       │   └── generate.py
│       │
│       ├── training/
│       │   ├── callbacks.py
│       │   ├── metrics.py
│       │   ├── checkpoints.py
│       │   └── logging.py
│       │
│       └── utils/
│           ├── config.py
│           ├── reproducibility.py
│           └── device.py
│
├── models/
│   ├── phase1/
│   └── phase2/
│
├── experiments/
│   ├── phase1/
│   └── phase2/
│
├── tests/
│   ├── test_ciphers.py
│   ├── test_dataset.py
│   ├── test_features.py
│   ├── test_nn_from_scratch.py
│   ├── test_phase1.py
│   └── test_phase2.py
│
└── app/
    ├── app.py
    ├── components/
    │   ├── training_view.py
    │   ├── classifier_view.py
    │   ├── decryptor_view.py
    │   ├── metrics_view.py
    │   └── network_view.py
    └── assets/
```

This structure is intentionally modular. We can simplify it during the earliest milestones rather than creating empty files for everything immediately.

---

# 20. Module Responsibilities

## `ciphers/`

Pure cryptographic transformations.

Must not depend on PyTorch or Streamlit.

Example conceptual API:

```python
encrypt(plaintext, key) -> ciphertext
decrypt(ciphertext, key) -> plaintext
```

---

## `data/`

Responsible for:

- generating examples
- splitting data
- encoding
- decoding
- batching
- dataset loading

---

## `features/`

Responsible for traditional ciphertext analysis features.

---

## `nn_from_scratch/`

Educational implementation of neural-network fundamentals.

This is where we explicitly implement:

- weights
- biases
- forward propagation
- activation functions
- loss
- gradients
- backpropagation
- parameter updates

---

## `phase1/`

Everything specific to cipher classification.

---

## `phase2/`

Everything specific to neural decryption.

The advanced sequence models belong here.

---

## `training/`

Shared training infrastructure:

- metrics
- checkpointing
- logging
- callbacks
- reproducibility

---

## `app/`

Streamlit dashboard only.

The dashboard should call model/data services rather than containing core ML logic itself.

---

# 21. Data Flow

## Training

```text
Plaintext corpus/templates
        |
        v
Cipher generator
        |
        v
Labeled ciphertext dataset
        |
        +----------------------+
        |                      |
        v                      v
 Phase 1 dataset          Phase 2 dataset
        |                      |
        v                      v
 Classifier training      Decryptor training
        |                      |
        v                      v
 Phase 1 checkpoint       Phase 2 checkpoint
```

## Inference

```text
User ciphertext
       |
       v
Preprocessing
       |
       v
Phase 1 model
       |
       v
Predicted cipher type
       |
       v
Phase 2 model
       |
       v
Predicted plaintext
       |
       v
Dashboard
```

---

# 22. Model Interfaces

The rest of the application should not need to know internal neural-network details.

Conceptually:

```python
classifier.predict(ciphertext)
```

should return something like:

```python
{
    "cipher": "caesar",
    "probabilities": {
        "caesar": 0.968,
        "vigenere": 0.017,
        "affine": 0.011,
        "substitution": 0.004
    }
}
```

The decryptor should conceptually expose:

```python
decryptor.predict(ciphertext, cipher_type)
```

returning:

```python
{
    "plaintext": "HELLOWORLD",
    "confidence": ...,
    "token_scores": [...]
}
```

The exact APIs can evolve during implementation.

---

# 23. Reproducibility

Experiments should record:

- random seed
- dataset version/configuration
- model architecture
- hyperparameters
- optimizer
- learning rate
- batch size
- number of epochs
- training/validation/test metrics
- model checkpoint

The same configuration should be reproducible as closely as practical.

---

# 24. Suggested Starting Hyperparameters

These are **starting points for experimentation**, not final values.

## Phase 1

```text
Hidden layers: 2
Hidden sizes: 64 -> 32
Activation: ReLU
Output: 4 classes
Loss: Cross Entropy
Optimizer: Adam
Learning rate: 1e-3
Batch size: 64
Epochs: 20–100
```

## Phase 2 baseline

Start smaller:

```text
Embedding dimension: 32
Hidden dimension: 64
Batch size: 32–64
Learning rate: 1e-3 initially
```

Then experiment.

For RNN/LSTM/Transformer models, choose architecture size based on dataset size and observed training behavior rather than blindly using large models.

---

# 25. Training Visualization

The dashboard should expose at least:

## Loss

```text
Epoch
  |
  v
Training Loss
Validation Loss
```

## Accuracy

```text
Epoch
  |
  v
Training Accuracy
Validation Accuracy
```

## Phase 1 confusion matrix

```text
             Predicted
             C   V   A   S

Actual C     ...
Actual V     ...
Actual A     ...
Actual S     ...
```

## Phase 2 examples

Show:

```text
Input ciphertext
Expected plaintext
Model prediction
Character accuracy
```

This is particularly valuable because sequence models can appear to have reasonable aggregate metrics while still making important character-level errors.

---

# 26. Neural Network Visualization

The dashboard should eventually provide a conceptual network view.

For Phase 1:

```text
Input
  |
  v
Dense(64)
  |
  v
ReLU
  |
  v
Dense(32)
  |
  v
ReLU
  |
  v
Dense(4)
  |
  v
Softmax
```

For Phase 2, show a simplified conceptual view rather than every parameter:

```text
Characters
    |
    v
Embedding
    |
    v
Sequence Model
    |
    v
Output Vocabulary
```

For Transformers:

```text
Tokens
  |
Embedding
  |
Positional Information
  |
Multi-Head Attention
  |
Add & Norm
  |
Feed Forward
  |
Add & Norm
  |
Output
```

---

# 27. Testing Strategy

## Cipher tests

For every cipher:

```text
plaintext
 -> encrypt
 -> decrypt
 -> plaintext
```

must round-trip correctly.

Test:

- empty input where supported
- short input
- long input
- repeated characters
- spaces/punctuation according to the chosen specification
- invalid keys/parameters

## Dataset tests

Verify:

- labels are valid
- ciphertext is generated correctly
- plaintext/ciphertext pairs correspond
- splits are valid
- no obvious leakage

## Neural-network tests

Test:

- forward pass dimensions
- loss calculation
- gradient dimensions
- parameter updates
- numerical gradient checks where useful

## Integration tests

Test:

```text
ciphertext
 -> classifier
 -> cipher type
 -> decryptor
 -> output
```

---

# 28. Learning Checklist

The project should be considered educationally successful only if the following concepts are understood, not merely used.

## Neural-network fundamentals

- [ ] neuron
- [ ] weight
- [ ] bias
- [ ] weighted sum
- [ ] activation function
- [ ] ReLU
- [ ] sigmoid
- [ ] softmax
- [ ] forward pass
- [ ] loss
- [ ] gradient
- [ ] derivative
- [ ] chain rule
- [ ] backpropagation
- [ ] gradient descent
- [ ] learning rate
- [ ] optimizer
- [ ] batch
- [ ] epoch
- [ ] parameter
- [ ] training
- [ ] validation
- [ ] testing
- [ ] overfitting
- [ ] underfitting
- [ ] regularization

## Sequence modeling

- [ ] tokenization
- [ ] character-level representation
- [ ] embeddings
- [ ] sequence
- [ ] hidden state
- [ ] RNN
- [ ] LSTM
- [ ] gates
- [ ] sequence-to-sequence learning
- [ ] teacher forcing where applicable

## Transformers

- [ ] attention
- [ ] query
- [ ] key
- [ ] value
- [ ] self-attention
- [ ] scaled dot-product attention
- [ ] multi-head attention
- [ ] positional information
- [ ] feed-forward network
- [ ] residual connection
- [ ] layer normalization
- [ ] Transformer encoder/decoder concepts

## Cryptography

- [ ] plaintext
- [ ] ciphertext
- [ ] key
- [ ] encryption
- [ ] decryption
- [ ] substitution
- [ ] transposition
- [ ] Caesar
- [ ] Vigenère
- [ ] Affine
- [ ] monoalphabetic substitution
- [ ] frequency analysis
- [ ] entropy
- [ ] index of coincidence
- [ ] cryptanalysis

---

# 29. Future Extensions

Once the core project works, possible extensions include:

## More cipher types

- Rail Fence
- Columnar Transposition
- Playfair
- Hill cipher
- other classical educational ciphers

## Better classification

- character-level CNN
- RNN
- LSTM
- Transformer classifier

## Better decryption

- attention-based sequence-to-sequence model
- Transformer decoder
- key prediction as an auxiliary task
- confidence estimation
- candidate generation

## Explainability

Show:

- important input positions
- attention patterns
- token probabilities
- uncertainty
- alternative predictions

## Continual learning experiment

Eventually experiment with:

```text
Train on:
Caesar
Vigenère
Affine

        ↓

Introduce:
Substitution

        ↓

Adapt/retrain model
```

Then measure how the new training affects previously learned classes.

This should be treated as an experiment, not assumed to work automatically.

---

# 30. Future Advanced Architecture

A potential later architecture:

```text
                       CIPHERTEXT
                            |
                            v
                 +---------------------+
                 | Preprocessing       |
                 +----------+----------+
                            |
                            v
                 +---------------------+
                 | Phase 1 Transformer |
                 | Classifier           |
                 +----------+----------+
                            |
                            v
                      Cipher Type
                            |
                            v
                 +---------------------+
                 | Phase 2 Transformer |
                 | Decryptor            |
                 +----------+----------+
                            |
                            v
                       Plaintext
```

This is a future target, not the starting implementation.

---

# 31. What We Should NOT Do

Do not:

- immediately start with a huge Transformer
- download a giant pretrained language model and call the project finished
- hide all neural-network mathematics behind PyTorch
- train on modern production encryption
- claim that a model can universally break arbitrary encryption
- judge the project solely by accuracy
- add dozens of cipher types before the basic pipeline works
- mix dashboard code with model/training logic
- optimize before establishing a correct baseline

---

# 32. Recommended Development Sequence

The canonical order is:

```text
1. Understand the project architecture
        |
        v
2. Implement classical ciphers
        |
        v
3. Build dataset generator
        |
        v
4. Explore ciphertext statistics
        |
        v
5. Build a single neuron manually
        |
        v
6. Build dense layers with NumPy
        |
        v
7. Implement forward propagation
        |
        v
8. Implement loss
        |
        v
9. Implement backpropagation
        |
        v
10. Train Phase 1 classifier
        |
        v
11. Visualize Phase 1 training
        |
        v
12. Rebuild Phase 1 with PyTorch
        |
        v
13. Build Phase 2 baseline
        |
        v
14. Introduce sequence modeling
        |
        v
15. RNN
        |
        v
16. LSTM
        |
        v
17. Attention
        |
        v
18. Transformer
        |
        v
19. Build integrated dashboard
        |
        v
20. Evaluate and document experiments
```

---

# 33. Definition of the Finished Project

CipherMind is considered complete when a user can:

1. Open the dashboard.
2. See the training status and metrics of the models.
3. Enter a ciphertext.
4. Run the Phase 1 classifier.
5. See the predicted cipher and probability distribution.
6. Pass the result to Phase 2.
7. Receive a candidate plaintext.
8. Inspect useful metrics and model information.
9. Understand, at least conceptually, how the neural networks produced the result.

The project is also considered educationally complete when the developer can explain the major neural-network operations rather than simply knowing which API calls produced them.

---

# 34. Instructions for Future ChatGPT Sessions

This file is the canonical context for the CipherMind project.

At the beginning of future project conversations:

1. Read `PROJECT_SPEC.md`.
2. Preserve the two-phase architecture unless the user explicitly changes it.
3. Determine the current milestone before implementing new features.
4. Build incrementally.
5. Explain the relevant neural-network concept before or while implementing it.
6. Prefer simple working implementations over premature complexity.
7. Do not jump to Transformers before the earlier concepts are understood.
8. Keep cryptography implementations, ML models, training code, and dashboard code modular.
9. Keep Phase 1 classification and Phase 2 decryption conceptually separate.
10. When changing architecture, update this specification if the change becomes part of the canonical project design.
11. Record important experiments and their results.
12. Explain why a particular design choice is being made.
13. When debugging, identify the underlying concept rather than only applying a patch.
14. Treat this project as both software and an educational curriculum.

### Most important instruction

**Do not optimize for finishing quickly. Optimize for the developer understanding why the system works.**

The final project should demonstrate not only that a neural network can classify/decrypt the supported classical ciphers, but that the developer understands the mechanics that make the models learn.
