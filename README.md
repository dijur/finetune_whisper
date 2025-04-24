# Fine-Tuning Whisper for Low-Resource Languages Using Synthetic Speech Data

This is a comprehensive toolkit designed to improve Automatic Speech Recognition (ASR) for low-resource languages through synthetic data generation and model fine-tuning. It addresses the challenge of limited audio data availability by leveraging text-to-speech technologies and providing robust evaluation metrics.

---

## Data

We use the **[Tajik Corpus](https://huggingface.co/datasets/muhtasham/tajik-corpus)** hosted on HuggingFace as the main text dataset. From this corpus, text samples are selected and converted into synthetic speech using the `facebook/mms-tts-tgk` TTS model.

**Notebook**: `notebooks/1_synthetic_data_generation.ipynb` handles the data preparation.

### Synthetic Data Generation

Synthetic audio is generated using a custom class `SyntheticDataGenerator` that:
- Loads a TTS model (`facebook/mms-tts-tgk`)
- Samples random text entries from the corpus
- Converts text into audio
- Applies augmentations
- Stores audio files and logs metadata

#### Output Format

- **Audio files**: Stored as `synthetic_000001.wav`, `synthetic_000002.wav`, etc.
- **Metadata**: JSON lines file (`metadata.jsonl`) including:
  - `audio_path`
  - `text`
  - `duration`

### Audio Augmentation Techniques

Enabled by configuration:

```json
"augmentation": {
  "enabled": true,
  "noise_factor": 0.003,
  "speed_range": [0.9, 1.1],
  "pitch_shift_range": [-2, 2]
}
```

Includes:
- Gaussian noise addition
- Speed perturbation
- Pitch shifting

These help diversify synthetic audio and simulate more realistic conditions.

---

## Fine-Tuning Whisper

Once synthetic data is prepared, we fine-tune OpenAI’s Whisper model using the notebook:

**Notebook**: `notebooks/2_fine_tune_whisper.ipynb`

This notebook is adapted from the HuggingFace tutorial:  
👉 [Fine-tune Whisper for Multilingual ASR](https://huggingface.co/blog/fine-tune-whisper)

All experiments were conducted using **Google Colab** with GPU support.

---

## Evaluation

To assess model performance:

- We manually recorded **100 real Tajik sentences**
- Evaluation will be conducted comparing:
  - Baseline Whisper models: `small`, `medium`, `large`
  - Our **fine-tuned Whisper model**

**Metrics** (TBD): Word Error Rate (WER), Character Error Rate (CER), etc.

---

## Future Improvements

- [ ] **Expand synthetic dataset** with other TTS models (e.g., OpenAI TTS, Coqui)
- [ ] **Optimize training hyperparameters** (batch size, learning rate, etc.)
- [ ] **Gather more real evaluation data**, including diverse speakers
- [ ] 

---

## Acknowledgements

- HuggingFace TTS & Whisper resources
- [Tajik Corpus](https://huggingface.co/datasets/muhtasham/tajik-corpus)
- MMS-TTS by Facebook AI

