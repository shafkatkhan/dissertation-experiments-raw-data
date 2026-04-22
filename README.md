# Dissertation Experiments Raw Data

Raw experimental data for the evaluation of AI-powered audio transcription and structured data extraction across multiple LLM providers (OpenAI, Mistral, and Google Gemini), conducted as part of an undergraduate dissertation at King's College London.

## Overview

This repository contains the complete, unfiltered results of two experiments designed to evaluate the transcription accuracy, extraction accuracy, and processing latency of three LLM providers. The data includes all processing attempts, including repeated runs and attempts where the provider failed to return a valid response.

| Experiment | Provider | Recordings |
|---|---|---:|
| 1 | Mistral | 5,192 |
| 1 | OpenAI | 3,262 |
| 1 | Google Gemini | 2,699 |
| 2 | Mistral | 389 |
| 2 | OpenAI | 270 |
| 2 | Google Gemini | 399 |

## Experiments

### Experiment 1: Controlled Speech Recordings

Uses the [Speech Accent Archive](https://www.kaggle.com/datasets/rtatman/speech-accent-archive), a dataset of 2,138 audio recordings in which speakers of different native languages read the same standardised English passage ("Please call Stella..."). This controlled setup enables direct comparison of transcription accuracy and structured data extraction across accents and providers. Each recording was transcribed and then passed through an extraction stage to produce a structured JSON object.

### Experiment 2: Realistic Medical Consultations

Uses another publicly available dataset of 272 simulated [doctor–patient consultations](https://www.kaggle.com/datasets/azmayensabil/doctor-patient-conversation-large) (OSCE recordings). These longer recordings (7–18 minutes) test transcription quality under conditions that more closely resemble real-world usage. Only transcription accuracy (WER) and processing latency are measured; no extraction is performed.

## Repository Structure

```
dissertation-experiments-raw-data/
├── experiment-1/
│   ├── experiment-1-results.csv
│   └── experiment-1-extractions.csv
├── experiment-2/
│   └── experiment-2-results.csv
└── README.md
```

## File Descriptions

### `experiment-1/experiment-1-results.csv`

Contains one row per processing attempt for Experiment 1. Data is sorted by model configuration, then by filename.

| Column | Type | Description |
|---|---|---|
| `id` | int | Unique identifier for the processing attempt. |
| `filename` | varchar | Name of the audio file processed (e.g. `afrikaans3.mp3`). |
| `speaker_native_language` | varchar | Native language of the speaker in the recording (e.g. `afrikaans`, `arabic`). |
| `audio_duration_seconds` | decimal | Duration of the audio file in seconds. |
| `transcript_text` | text | The full transcript produced by the LLM transcription model. |
| `wer` | decimal | Word Error Rate, calculated by comparing the AI-generated transcript against the ground truth using a Levenshtein Distance algorithm. Expressed as a percentage. |
| `transcription_time_ms` | int | Time taken by the provider's transcription API to return a response, in milliseconds. |
| `extracted_json` | text | The raw JSON object returned by the extraction model, containing structured data extracted from the transcript. |
| `extraction_accuracy` | decimal | Percentage of the 11 expected data points correctly extracted from the transcript (see extraction schema below). |
| `extraction_time_ms` | int | Time taken by the provider's extraction API to return a response, in milliseconds. |
| `total_time_ms` | int | Total end-to-end processing time (transcription + extraction), in milliseconds. |
| `status` | varchar | Processing outcome: `success` or `failed`. |
| `error_message` | text | Error message returned by the provider if the attempt failed; `NULL` for successful attempts. |
| `llm_provider` | varchar | The LLM provider used: `openai`, `mistral`, or `gemini`. |
| `llm_transcription_model` | varchar | The specific model used for transcription. |
| `llm_extraction_model` | varchar | The specific model used for data extraction. |
| `processed_at` | timestamp | Timestamp of when the processing attempt was executed. |

### `experiment-1/experiment-1-extractions.csv`

Contains the extracted data from each processing attempt of Experiment 1. Each row corresponds to a `result_id` found in `experiment-1-results.csv`, and records the individual field-level extraction outcomes for that attempt.

| Column | Type | Description |
|---|---|---|
| `result_id` | int | Foreign key referencing the `id` column in `experiment-1-results.csv`. |
| `person` | varchar | The extracted person's name (expected: `Stella`). |
| `items` | text | JSON array of extracted items from the passage. |
| `has_snow_peas` | boolean | Whether `snow peas` was correctly identified in the extracted items. |
| `has_blue_cheese` | boolean | Whether `blue cheese` was correctly identified in the extracted items. |
| `has_snack` | boolean | Whether `snack` was correctly identified in the extracted items. |
| `has_plastic_snake` | boolean | Whether `plastic snake` was correctly identified in the extracted items. |
| `has_toy_frog` | boolean | Whether `toy frog` was correctly identified in the extracted items. |
| `brother` | varchar | The extracted brother's name (expected: `Bob`). |
| `bags_count` | int | The extracted number of bags (expected: `3`). |
| `bags_color` | varchar | The extracted bag colour (expected: `red`). |
| `meeting_day` | varchar | The extracted meeting day (expected: `Wednesday`). |
| `meeting_location` | varchar | The extracted meeting location (expected: `train station`). |

### `experiment-2/experiment-2-results.csv`

Contains one row per processing attempt for Experiment 2. Data is sorted by model configuration, then by filename. No extraction is performed in this experiment.

| Column | Type | Description |
|---|---|---|
| `id` | int | Unique identifier for the processing attempt. |
| `filename` | varchar | Name of the audio file processed (e.g. `CAR0001.mp3`). |
| `audio_duration_seconds` | decimal | Duration of the audio file in seconds. |
| `transcript_text` | longtext | The full transcript produced by the LLM transcription model. |
| `wer` | decimal | Word Error Rate, expressed as a percentage. |
| `transcription_time_ms` | int | Time taken by the provider's transcription API to return a response, in milliseconds. |
| `status` | varchar | Processing outcome: `success` or `failed`. |
| `error_message` | text | Error message returned by the provider if the attempt failed; `NULL` for successful attempts. |
| `llm_provider` | varchar | The LLM provider used: `openai`, `mistral`, or `gemini`. |
| `llm_transcription_model` | varchar | The specific model used for transcription. |
| `processed_at` | timestamp | Timestamp of when the processing attempt was executed. |

## Extraction Schema (Experiment 1)

The extraction model was instructed to return a JSON object conforming to the following schema, derived from the standardised passage content:

| Field | Type | Expected Value |
|---|---|---|
| `person` | string | `Stella` |
| `items` | array | `["snow peas", "blue cheese", "snack", "plastic snake", "toy frog"]` |
| `brother` | string | `Bob` |
| `bags_count` | integer | `3` |
| `bags_color` | string | `red` |
| `meeting_day` | string | `Wednesday` |
| `meeting_location` | string | `train station` |

Extraction accuracy is calculated as the percentage of 11 individual data points correctly matched (6 scalar fields + 5 array items). String comparisons are case-insensitive; integer comparisons use type coercion; array items are matched individually as complete phrases.

## Notes

- All experiments were conducted in March–April 2026.
- Data represents the complete, unfiltered output of both experiments, including repeated runs and failed attempts.
- Failed attempts (where `status = 'failed'`) were excluded from aggregate metric calculations in the dissertation analysis.

## Licence

This data is provided for academic and research purposes.
