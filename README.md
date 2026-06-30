# Churn Analysis Project – Part 4

## Track Chosen
Track C – Model Prediction Explanation Pipeline

---

## Project Overview
This project builds an LLM-powered explanation layer on top of the churn prediction model from Part 3.

The workflow includes:

1. Load the trained model
2. Predict customer churn
3. Predict probability
4. Send prediction data to OpenRouter API
5. Get structured JSON explanation
6. Validate JSON output
7. Apply PII guardrails

---

## API Setup

The API key is stored in environment variables.

Example:

```python
import os
os.environ["LLM_API_KEY"] = "your_api_key_here"
```

No API key is hardcoded inside the repository.

---

## System Prompt

```text
You are a structured JSON explanation generator.

Return only valid JSON.

Schema:
{
  "prediction_label": "string",
  "confidence_level": "low|medium|high",
  "top_reason": "string",
  "second_reason": "string",
  "next_step": "string"
}

Do not add extra text.
```

---

## User Prompt Template

```text
Customer Features:
Age = {age}
Balance = {balance}
Tenure = {tenure}
Products = {products}

Predicted Class = {prediction}
Probability = {probability}

Explain the prediction in JSON.
```

---

## Why Temperature = 0

Temperature 0 is used because:

- Output is deterministic
- Better consistency
- Better JSON formatting
- Better validation success

Temperature 0.7 creates more random outputs.

---

## Temperature Comparison

| Input | Temp 0 | Temp 0.7 | Difference |
|---|---|---|---|
| Record 1 | Stable | Variable | More randomness |
| Record 2 | Stable | Different wording | Less deterministic |
| Record 3 | Stable | Creative response | Output changes |

---

## JSON Schema

```python
schema = {
    "type": "object",
    "properties": {
        "prediction_label": {"type": "string"},
        "confidence_level": {"type": "string"},
        "top_reason": {"type": "string"},
        "second_reason": {"type": "string"},
        "next_step": {"type": "string"}
    },
    "required": [
        "prediction_label",
        "confidence_level",
        "top_reason",
        "second_reason",
        "next_step"
    ]
}
```

---

## Guardrails (PII Check)

Before every LLM call:

```python
import re

def has_pii(text):
    email_pattern = r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'
    phone_pattern = r'\b\d{10}\b'
    return bool(re.search(email_pattern, text) or re.search(phone_pattern, text))
```

---

## Guardrail Test Results

| Input | Result |
|---|---|
| test@gmail.com | Blocked |
| customer age 35 balance 20000 | Passed |

---

## Three Input Demonstration

| Feature Input | Predicted Class | Probability | Explanation JSON | Validation Status |
|---|---|---|---|---|
| Age 42, Balance 50000, Tenure 3 | Class 0 | 0.81 | Valid JSON | Pass |
| Age 60, Balance 120000, Tenure 1 | Class 1 | 0.92 | Valid JSON | Pass |
| Age 25, Balance 10000, Tenure 5 | Class 0 | 0.73 | Valid JSON | Pass |

---

## Output Example

```json
{
  "prediction_label": "Class 0",
  "confidence_level": "high",
  "top_reason": "High balance",
  "second_reason": "Low tenure",
  "next_step": "Improve customer engagement"
}
```

---

## Validation

JSON validation is done using:

```python
validate(instance=response_json, schema=schema)
```

If validation fails, fallback values are returned.

---

## Files Included

- analysis.ipynb
- analysis_part4.ipynb
- best_model.pkl
- Cleaned_data.csv
- README.md

---

## Conclusion

This project successfully combines:

- Machine Learning prediction
- OpenRouter API integration
- Structured JSON explanation
- Schema validation
- PII guardrails
- Temperature comparison

The full pipeline works successfully end-to-end.
