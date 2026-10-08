# CipherMind — Complete System Architecture & Technology Stack

**Purpose:** Architecture companion to `PROJECT_SPEC.md`. The project spec remains the canonical source for project goals and milestones; this document explains how the system components fit together and how data moves through them. Update both documents if a major design decision changes.

## 1. Overview

CipherMind is an educational machine-learning lab for **classical/toy ciphers**. It has two distinct tasks:

1. **Phase 1 — Classifier:** Given ciphertext, estimate which supported cipher family most likely produced it.
2. **Phase 2 — Neural decryptor:** Given ciphertext and a cipher-type label, generate a candidate plaintext.

A Streamlit dashboard connects these components and visualizes inference, training progress, evaluation metrics, and selected model internals. We will implement a small network with NumPy first to understand the mathematics, then move to PyTorch for more advanced models.

This is not a system for breaking modern cryptography such as AES, RSA, ChaCha20, or production encryption. Model performance depends on the training distribution, preprocessing, cipher parameters, message lengths, and evaluation setup.

**Design principles**
- Separate cipher algorithms, data generation, features, models, training, evaluation, and UI.
- Keep classification and decryption as distinct tasks with explicit interfaces.
- Understand a mechanism before relying on a framework to hide it.
- Generate reproducible synthetic data with known ground truth.
- Evaluate on held-out data and document failures, not just successes.
- Treat model scores as estimates, not guaranteed or automatically calibrated confidence.
- Build a small working system first; add RNN/LSTM/attention/Transformer models progressively.

## 2. High-level architecture

```text
                               CIPHERMIND
                                   |
                 +-----------------+------------------+
                 |                                    |
                 v                                    v
         TRAINING WORKFLOW                    INFERENCE WORKFLOW
                 |                                    |
        Plaintext/templates                         User input
                 |                                    |
                 v                                    v
        Classical cipher engine                Input validation
                 |                                    |
                 v                                    v
        Labeled synthetic data                Shared preprocessing
                 |                                    |
           +-----+-----+                              v
           |           |                       Phase 1 classifier
           v           v                              |
       Phase 1      Phase 2                           v
       dataset      dataset                    Cipher prediction
           |           |                              |
           v           v                       Uncertainty check
       Classifier   Decryptor                          |
       training     training                           v
           |           |                        Phase 2 decryptor
           v           v                              |
       Checkpoint   Checkpoint                         v
           |           |                       Candidate plaintext
           +-----+-----+                              |
                 |                                    v
                 +----------> Dashboard <------ Result + diagnostics
```

The workflows are logically separate, even if they run on one computer and in one Python project. Training updates parameters and produces artifacts; inference loads those artifacts and makes predictions.

### Logical layers

1. **Presentation:** Streamlit pages, controls, plots, and result displays.
2. **Application/services:** Coordinates preprocessing, model loading, inference, training requests, and result formatting.
3. **Domain:** Classical cipher implementations and shared text-handling rules.
4. **Data:** Synthetic example generation, encoding, splitting, batching, and loading.
5. **ML:** Phase 1 and Phase 2 models, training loops, prediction, and evaluation.
6. **Artifacts/experiments:** Model parameters, metadata, configs, metric history, experiment records.
7. **Quality:** Unit/integration tests, reproducibility controls, input validation, error handling.

These are conceptual boundaries, not separate servers. The first version can be a local Python application.

## 3. Planned technology stack

| Technology | Role | When and why |
|---|---|---|
| **Python** | Main language | Used throughout; excellent ML ecosystem and suitable for learning by implementation. |
| **NumPy** | Numerical arrays and operations | First used to implement weighted sums, activations, loss, gradients, backpropagation, and parameter updates explicitly. |
| **PyTorch** | Deep-learning framework | Introduced after the NumPy classifier is understood; used for the practical classifier and Phase 2 sequence models. |
| **pandas** | Dataset inspection | Useful for inspecting generated records, label balance, and experiment summaries. |
| **scikit-learn** | Evaluation utilities | Confusion matrices, precision, recall, F1, and related classification metrics. Selected metrics can also be implemented manually for learning. |
| **Matplotlib** | Static plots | Loss/accuracy curves, confusion matrices, and educational plots. |
| **Streamlit** | Interactive dashboard | Input forms, training/evaluation views, metrics, and model inspection without a separate frontend framework. |
| **Plotly (optional)** | Interactive charts | Add later only if it materially improves the dashboard; not required initially. |
| **pytest** | Automated tests | Tests cipher round-trips, data correctness, numerical components, model interfaces, and integration flows. |
| **Git** | Version control | Tracks source and configuration changes. Avoid committing large generated datasets/model files unless intentionally small. |
| **Python `venv`** | Dependency isolation | Keeps project packages separate from system Python. |
| **YAML/JSON configuration** | Experiment settings | Stores seeds, model sizes, learning rates, batch sizes, paths, and other adjustable settings. YAML requires a parser dependency. |

**Tool progression**
- **Early stage:** Python + NumPy + pytest for ciphers, datasets, features, and a tiny network from scratch.
- **Classifier stage:** Add pandas, scikit-learn, and Matplotlib; train/evaluate Phase 1, then reproduce it in PyTorch.
- **Sequence stage:** Use PyTorch for a constrained baseline, RNN, LSTM, attention, and a small Transformer.
- **Dashboard stage:** Add Streamlit after core APIs and metrics are stable; add Plotly only if useful.

No database, cloud service, API server, GPU, or pretrained language model is required for the initial local version.

## 4. Component responsibilities

### Dashboard — `app/`
Collects user input, calls application services, and displays outputs. It must not implement cipher algorithms, backpropagation, or core training logic itself.

Planned views: overview; Phase 1 training and evaluation; Phase 2 training and example predictions; architecture/parameter explorer; inference; educational explanations.

### Application/service layer
Coordinates UI with domain and model code. Conceptual operations:

```python
classifier_service.predict(ciphertext)
decryptor_service.predict(ciphertext, cipher_type)
training_service.train_phase1(config)
training_service.train_phase2(config)
evaluation_service.evaluate_phase1(model, dataset)
evaluation_service.evaluate_phase2(model, dataset)
```

These are interface sketches, not a fixed API. Services are a suitable place for model loading, input validation, exceptions, and result formatting.

### Classical cipher library — `src/ciphermind/ciphers/`
Contains deterministic implementations for Caesar, Vigenère, Affine, and monoalphabetic substitution. Define consistent key and text-handling rules: spaces, punctuation, case, and unsupported characters. Keep this library independent of NumPy, PyTorch, and Streamlit.

The algorithms both generate synthetic examples and provide reference behavior for tests. A conventional decryptor given the secret key is not the same thing as the neural decryptor, which learns from examples.

### Data pipeline — `src/ciphermind/data/`
Generates or loads permitted plaintext; selects cipher and valid key/parameters; encrypts with the cipher library; records plaintext, ciphertext, cipher label, and key; splits data; encodes/decodes characters and labels; creates batches; records seeds and configuration.

Example record:

```json
{
  "plaintext": "HELLOWORLD",
  "cipher": "caesar",
  "key": 3,
  "ciphertext": "KHOORZRUOG"
}
```

The key is useful for reproducibility and evaluation, but is not automatically a Phase 2 input. The initial task is `(ciphertext, cipher_type) -> plaintext`; do not silently assume the model knows the secret key.

### Feature engineering — `src/ciphermind/features/`
Computes Phase 1 statistical features: length, character frequencies, normalized frequencies, n-grams, entropy, index of coincidence, and repeated-pattern statistics. Feature order and preprocessing must be identical during training and inference.

### NumPy neural network — `src/ciphermind/nn_from_scratch/`
Implements dense layers, weights/biases, initialization, activations, forward pass, classification loss, backward pass, gradients, parameter updates, and numerical gradient checks where practical. Optimize for clarity rather than speed.

### Phase 1 classifier — `src/ciphermind/phase1/`
Task:

```text
f(ciphertext representation) -> probabilities/scores over cipher classes
```

Initial classes: `0=Caesar`, `1=Vigenere`, `2=Affine`, `3=Substitution`.

Initial dense architecture:

```text
Feature vector -> Dense(64) -> ReLU -> Dense(32) -> ReLU
              -> Dense(4) -> class scores
```

Starting choices from `PROJECT_SPEC.md`: hidden sizes 64 and 32, cross-entropy, Adam initially, learning rate around `1e-3`, batch size around `64`, and an experimental range of 20–100 epochs. These are starting points, not guaranteed optimal settings.

In PyTorch, train using raw logits with the appropriate cross-entropy loss; apply softmax to display probabilities, not unnecessarily before the loss. The classifier owns its model, training, prediction, evaluation, and compatible checkpoint logic.

### Phase 2 decryptor — `src/ciphermind/phase2/`
Task:

```text
(ciphertext, cipher_type) -> candidate plaintext sequence
```

This is sequence prediction, not classification. Progress from a constrained baseline to RNN, LSTM, attention, then a small Transformer.

Character vocabulary may include `A-Z`, space, supported punctuation, and special tokens such as `<PAD>`, `<BOS>`, `<EOS>`, and `<UNK>`. Save the vocabulary and tokenization rules with the model.

Training may use **teacher forcing** (the correct previous output token is supplied while training next-token predictions). At inference, generation generally uses the model's own prior outputs, so training and inference behavior differ.

Metrics include character-level accuracy, exact-sequence accuracy, validation/test loss, and target-versus-prediction examples. The model can only learn what its data and architecture enable; cipher type alone does not provide the secret key.

### Shared training utilities — `src/ciphermind/training/`
Provides metric recording, checkpoint saving/loading, logs, reproducibility helpers, and later optional callbacks. Start with explicit training loops and refactor repeated code after both phases work.

### Configs, artifacts, experiments
`configs/` stores settings; `models/` stores model parameters and metadata; `experiments/` stores results and notes. Record seed, data settings/version, feature/vocabulary definition, architecture, optimizer, learning rate, batch size, epochs, metrics, and checkpoint identity.

Model weights and metadata must be compatible. Do not load checkpoints with mismatched preprocessing, class mappings, or vocabulary. Load trusted model files and follow framework safe-loading guidance.

### Tests — `tests/`
Test cipher round-trips and invalid keys; dataset correctness and split leakage; feature shape/values; forward/backward dimensions; loss and parameter updates; numerical gradients; save/load compatibility; inference contracts; and dashboard error cases.

## 5. Training workflow

```text
Plaintext templates/corpus
          |
          v
Select plaintext + cipher + valid key
          |
          v
Classical cipher implementation
          |
          v
Labeled records (plaintext, ciphertext, cipher, key)
          |
          v
Reproducible generation and data split
          |
          +------------------------------+
          |                              |
          v                              v
   Phase 1 preparation            Phase 2 preparation
   features + label               input + target sequences
          |                              |
          v                              v
   Classifier training             Decryptor training
          |                              |
          v                              v
   Validation metrics              Validation metrics
          |                              |
          v                              v
   Selected checkpoint             Selected checkpoint
          +---------------+--------------+
                          |
                          v
                 Dashboard artifacts
```

### Dataset and leakage strategy
`PROJECT_SPEC.md` suggests eventually starting around 10,000–50,000 examples, balanced across classes where practical. Begin smaller while debugging, then scale after correctness is established. Vary message lengths, text patterns, valid keys, and supported character handling. Record random seeds and generation settings.

A naive random row split can overstate generalization if examples are near-duplicates. Consider grouping related source sentences/templates before splitting, holding out some keys, or using separate message pools. Report both:
- **In-distribution evaluation:** test examples from the same broad generation rules.
- **Generalization evaluation:** deliberately held-out templates, keys, lengths, or conditions.

For each batch: encode inputs/targets; run forward pass; calculate loss; calculate gradients (manual NumPy or PyTorch autograd); update parameters; record metrics. At each epoch end, evaluate validation data without updating parameters. Keep the test set for final evaluation. Save selected checkpoints and training history.

## 6. Inference workflow

```text
User enters ciphertext
          |
          v
Validate input and supported length
          |
          v
Apply exact expected preprocessing
          |
          v
Phase 1 classifier -> class scores/probabilities
          |
          v
Uncertainty/input-quality decision
          |
          +---- uncertain/unsupported -> show warning
          |
          v
Phase 2 decryptor receives ciphertext + selected type
          |
          v
Generate candidate plaintext
          |
          v
Check output format and generation limit
          |
          v
Display result, scores, and limitations
```

Example: input `KHOORZRUOG`. A classifier might return illustrative scores such as Caesar `0.92`, Vigenère `0.04`, Affine `0.03`, Substitution `0.01`. These are placeholders, not benchmark results. The app could pass the top prediction to Phase 2. For a known Caesar example encrypted with shift 3, the correct plaintext is `HELLOWORLD`; the neural model must be evaluated to determine whether it learned to reproduce it.

A high classifier score does not prove the class is correct. The four-class model knows only those classes. An `Unknown / uncertain` behavior needs a defined rule and evaluation; adding a label alone does not make a model recognize every unknown cipher. Short messages may be inherently ambiguous.

Conceptual classifier result:

```python
{
    "predicted_cipher": "caesar",
    "probabilities": {
        "caesar": 0.92,
        "vigenere": 0.04,
        "affine": 0.03,
        "substitution": 0.01
    },
    "warning": None
}
```

Conceptual decryptor result:

```python
{
    "candidate_plaintext": "HELLOWORLD",
    "token_scores": None,
    "warning": None
}
```

All values are illustrative. Only expose token scores when available, and do not call them calibrated probabilities of correctness without calibration and evidence. If a user message has no known plaintext, the app cannot calculate its actual decryption accuracy.

## 7. Neural-network mechanics in context

For a dense layer:

```text
z = W x + b
a = activation(z)
```

- `x`: input vector.
- `W`: learned weights controlling influence of inputs.
- `b`: bias vector shifting the pre-activation values.
- `z`: weighted sums before activation.
- `a`: activated outputs.

**Forward propagation** computes the prediction from current parameters. **Loss** compares that prediction with the target. **Backpropagation** applies the chain rule to calculate how the loss changes with each trainable parameter. A **gradient** describes local change; it is not itself an update. An **optimizer** uses gradients to update parameters, while the **learning rate** controls update scale.

For Phase 1, cross-entropy measures the predicted class distribution against the true label. For Phase 2, token-level cross-entropy can measure next-character predictions across a target sequence. A **batch** is a subset used for an update; an **epoch** is one pass through training data. If training metrics improve while validation performance stalls or worsens, investigate overfitting.

## 8. Dashboard pages and data sources

1. **Overview:** purpose, supported ciphers, current model/checkpoint, latest evaluation and training status.
2. **Phase 1 training:** epoch, train/validation loss and accuracy, learning rate, batch size, architecture and checkpoint metadata. Source: training history/config.
3. **Phase 1 evaluation:** confusion matrix, per-class precision/recall/F1, class scores and failure cases. Source: predictions compared with ground-truth labels.
4. **Phase 2:** sequence loss, character accuracy, exact-sequence accuracy and target/prediction examples. Source: training history and held-out evaluation.
5. **Network explorer:** simplified architecture and selected weights, biases, activations, or gradients. Do not attempt to display every parameter of a large model.
6. **Inference:** ciphertext input, classify action, probabilities, uncertainty warning, decrypt action, candidate plaintext.
7. **Learning view:** explanations of the actual operation being demonstrated.

Charts must show real metrics, not simulated progress. Training may be long-running; avoid launching duplicate jobs when Streamlit reruns. Start with short local jobs and add explicit background-job state only when needed.

## 9. Proposed repository layout

```text
ciphermind/
├── README.md
├── PROJECT_SPEC.md
├── CIPHERMIND_SYSTEM_ARCHITECTURE.md
├── requirements.txt
├── .gitignore
├── configs/
│   ├── phase1.yaml
│   └── phase2.yaml
├── data/{raw,processed,generated}/
├── notebooks/
├── src/ciphermind/
│   ├── ciphers/
│   ├── data/
│   ├── features/
│   ├── nn_from_scratch/
│   ├── phase1/
│   ├── phase2/
│   ├── training/
│   └── utils/
├── models/{phase1,phase2}/
├── experiments/{phase1,phase2}/
├── tests/
└── app/
    ├── app.py
    ├── components/
    └── assets/
```

We should create only files needed for the current milestone, not dozens of empty files in advance.

**Dependency boundaries**
- `ciphers/` has no PyTorch or Streamlit dependency.
- `data/` uses the cipher library and data/numerical libraries.
- `features/` defines Phase 1 feature representation.
- `nn_from_scratch/` implements educational mechanics.
- `phase1/` owns classifier-specific behavior.
- `phase2/` owns sequence models and generation.
- `training/` owns shared logging/checkpoint utilities.
- `app/` calls services and renders outputs.

## 10. Runtime and deployment

### Initial local setup
1. Create a Python virtual environment.
2. Install dependencies from `requirements.txt`.
3. Generate a small dataset.
4. Run tests.
5. Train the current model from a script or notebook.
6. Save model artifacts and history.
7. Launch Streamlit.
8. Load saved artifacts for inference.

Start with CPU-friendly models. GPU acceleration is optional. There is no need for a separate backend, database, or cloud service initially. Pin or constrain dependency versions once the environment works, and keep `requirements.txt` aligned with actual imports.

Load model parameters, metadata, vocabulary, and preprocessing configuration as a compatible unit. Fail clearly if an artifact is missing or incompatible; do not retrain automatically during inference.

If the project is deployed later, review resource limits, model-file handling, dependencies, logging, input validation, and data retention. Deployment is future work.

## 11. Error handling

Handle: empty input; unsupported characters; input exceeding model length; missing/incompatible checkpoints; invalid keys; dataset failures; malformed feature vectors; non-finite loss; uncertain classification; output generation reaching its maximum length; and training/evaluation preprocessing mismatches.

Show actionable messages, log technical details for debugging, and never label a guessed plaintext as verified unless it has been compared with known ground truth.

## 12. Evaluation and acceptance criteria

### Cipher library
- Round-trip encryption/decryption tests pass for supported cases.
- Invalid parameters are handled consistently.
- Case, spaces, punctuation and unsupported characters follow documented rules.

### Data pipeline
- Every record has valid labels/parameters.
- Stored ciphertext matches plaintext and key under the reference implementation.
- Generation is reproducible from seed/config.
- Splits are valid and checked for obvious leakage.

### Phase 1
- Preprocessing and feature order match between training and inference.
- Output shape matches supported classes.
- Evaluate accuracy, precision, recall, F1, confusion matrix and per-class behavior.
- Use held-out test data only after model/config selection.
- Record failure cases.

### Phase 2
- Vocabulary and target encoding match between training and inference.
- Evaluate character-level and exact-sequence accuracy plus validation/test loss.
- Compare predictions with known plaintext on held-out examples.
- Bound output generation length.
- Document key and message-length assumptions.

### Integration
- Supported ciphertext flows through preprocessing, Phase 1, uncertainty handling, Phase 2 and dashboard display.
- Invalid input and missing artifacts produce clear errors.
- Dashboard metrics match saved records.
- No UI action claims success before the operation completes.

### Educational success
The developer should be able to explain weights, biases, activations, forward propagation, loss, gradients, backpropagation, optimizer updates, overfitting, sequence prediction, embeddings, attention, and Transformers as they are introduced.

## 13. Implementation roadmap

Follow `PROJECT_SPEC.md`:
1. Environment/repository setup.
2. Classical ciphers and unit tests.
3. Dataset generator.
4. Statistical features.
5. Single neuron manually.
6. Dense layers with NumPy.
7. Forward propagation.
8. Loss.
9. Backpropagation and optimizer updates.
10. Phase 1 classifier.
11. Training visualizations.
12. PyTorch Phase 1 and comparison.
13. Phase 2 baseline.
14. Sequence modeling.
15. RNN.
16. LSTM.
17. Attention.
18. Transformer.
19. Integrated dashboard.
20. Evaluate and document experiments.

At every milestone, keep a small working result, add tests, and record what was learned before increasing complexity.

## 14. One complete run, summarized

### Training
1. Configuration defines ciphers, data counts, seeds and model settings.
2. Generator creates examples using the reference cipher implementations.
3. Pipeline prepares separate Phase 1 and Phase 2 datasets.
4. Classifier trains on cipher labels.
5. Decryptor trains on ciphertext/type inputs and plaintext targets.
6. Both record metrics and validate checkpoints.
7. Selected artifacts and metadata are saved.
8. Dashboard reads history and artifacts.

### User inference
1. User pastes ciphertext.
2. Service validates and preprocesses it.
3. Classifier returns scores over known classes.
4. App applies uncertainty handling.
5. Decryptor generates a candidate plaintext.
6. Dashboard displays candidate and model outputs with caveats.
7. For a labeled test example, evaluation can compare the candidate with known plaintext.

## 15. Final architecture decision

CipherMind will begin as a **modular local Python application**:
- Python coordinates the project.
- NumPy exposes neural-network fundamentals.
- PyTorch powers later deep-learning models.
- pandas and scikit-learn support inspection/evaluation.
- Matplotlib visualizes training.
- Streamlit presents the dashboard.
- pytest checks correctness.
- Git and `venv` support reproducible development.

The classifier and decryptor remain separate models with separate objectives, datasets/preprocessing, metrics, and artifacts. They communicate through explicit interfaces. The dashboard calls those interfaces rather than containing ML logic.

The goal is not merely to produce a decryption candidate. It is to understand how the system learns, how we evaluate it, and where its limitations become visible.
