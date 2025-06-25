# Fine-Tuning Whisper for Tajik Using Synthetic Speech Data

This toolkit enables automatic speech recognition (ASR) for low-resource languages by generating synthetic speech data and fine-tuning Whisper. The approach is validated on Tajik, showing that synthetic data and augmentation can dramatically improve ASR performance, even surpassing much larger baseline models.

---

## Data

We use the **Tajik Corpus** as the main text source. Sentences are selected for vocabulary and syntactic diversity, excluding those with non-standard orthography or excessive length.

Synthetic speech is generated using the `facebook/mms-tts-tgk` TTS model at 16 kHz. The process is automated with a custom `SyntheticDataGenerator` class, which:

* Randomly samples sentences from the corpus
* Converts text to audio (WAV)
* Applies augmentations (see below)
* Logs metadata (audio path, text, duration) in a JSONL file

**Quality control**: Random samples of generated audio are perceptually checked and compared to real Tajik recordings.

---

## Output Format

* **Audio files**: `synthetic_000001.wav`, `synthetic_000002.wav`, etc.
* **Metadata**: `metadata.jsonl` with `audio_path`, `text`, `duration`

---

## Audio Augmentation

Augmentation is crucial for bridging the gap between synthetic and real speech. The following are applied probabilistically:

```json
"augmentation": {
  "enabled": true,
  "noise_factor": 0.003,
  "speed_range": [0.9, 1.1],
  "pitch_shift_range": [-2, 2]
}
```

Techniques:

* **Gaussian noise** (Ko et al., 2015)
* **Speed perturbation** (Ko et al., 2015; Park et al., 2019)
* **Pitch shifting** (Cui et al., 2015)

These simulate real-world variability and increase model robustness.

---

## Fine-Tuning Whisper

Fine-tuning is performed on the **Whisper-small** model using Hugging Face Transformers in Google Colab Pro. The pipeline includes:

* **Feature extraction**: Audio resampled to 16 kHz, converted to log-Mel spectrograms
* **Tokenization**: Using Whisper tokenizer
* **Training**: Cross-entropy loss, early stopping on validation loss

**Training setup:**

* Batch size: `16`
* Learning rate: `2e-5`
* Epochs: `10`
* Optimizer: `Adam`

All code, environment details, and hyperparameters are provided for full reproducibility.

**Notebook**: `notebooks/2_fine_tune_whisper.ipynb` (adapted from Hugging Face’s tutorial)

---

## Evaluation

100 real Tajik sentences were manually recorded (50 male, 50 female speakers), covering a range of vocabulary, sentence lengths, and environments (indoors, outdoors, with noise).

### Models compared:

* Whisper-small (baseline)
* Whisper-medium (baseline)
* Whisper-large (baseline)
* Fine-tuned Whisper-small (ours)

### Metrics:

* **Word Error Rate (WER)**
* **Character Error Rate (CER)**

### Results:

| Model          | WER (%) | CER (%) |
| -------------- | ------- | ------- |
| Fine-tuned     | 55.45   | 26.86   |
| Whisper-Small  | 101.46  | 44.52   |
| Whisper-Medium | 95.56   | 40.41   |
| Whisper-Large  | 94.23   | 43.23   |

### Gender Analysis:

* Fine-tuned model:

  * WER: 40.9% (female), 69.7% (male)
  * CER: 10.4% (female), 43.6% (male)
* All models performed better on female speakers, likely due to TTS voice characteristics.

### Qualitative Analysis:

* Fine-tuned model produces more intelligible transcriptions, but struggles with:

  * Word boundaries
  * Rare vocabulary
  * Morphology (especially for male speakers and complex sentences)
* Baseline models often fail to produce usable transcriptions, sometimes showing cross-lingual interference (e.g., Persian-like outputs)

---

## Data Preparation

Before running notebooks, manually download and place data:

### Training (`2_fine_tune_whisper.ipynb`):

* `Colab Outputs` folder
* `metadata.jsonl`
* `audio/`

### Evaluation (`4_evaluate_and_compare.ipynb`):

* `holdout_metadata.jsonl` & folder
* `holdout_audio/` folder

> **Note:** Notebooks do not download or mount Google Drive automatically.

---

## Limitations

* Evaluation set is small (100 sentences) and may not capture full diversity (dialect, age, environment)
* Reliance on a single TTS model may introduce bias (e.g., gender)
* Synthetic data, while effective, cannot fully substitute for real speech
* Error rates, though improved, remain high for real-world deployment

---

## Future Directions

* Expand synthetic dataset with more TTS voices (different genders, ages, accents) and models (e.g., OpenAI TTS, Coqui)
* Refine text selection to better cover rare words and morphological variants
* Incorporate even small amounts of real speech for hybrid training
* Expand evaluation set for more robust assessment
* Explore advanced augmentations (reverberation, real background noise, adversarial perturbations)
* Apply pipeline to other low-resource languages

---

## Acknowledgements

* Hugging Face TTS & Whisper resources
* Tajik Corpus
* MMS-TTS by Facebook AI