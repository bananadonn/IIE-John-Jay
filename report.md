# Model Comparison Report — Week 4

**Name:** Donald Reith 
**Date:** 2026-04-02  

---

## Test Setup

**Input dataset:** 5 cybersecurity alert text samples covering:
- 2 clearly concerning/high-severity records (unauthorized login, brute-force SSH)
- 1 ambiguous/edge case record (phishing email)
- 2 routine/benign records (scheduled firewall update, normal system utilization)

**Models tested:**
1. distilbert-base-uncased-finetuned-sst-2-english (sentiment analysis)
2. facebook/bart-large-mnli (zero-shot classification)
3. dslim/bert-large-NER (named entity recognition)
4. Groq Llama 3.1 8B Instant (LLM classification)

**Evaluation criteria:** Label accuracy, confidence score, reasoning quality, ease of integration in n8n

---

## Results Summary

| Record | Sentiment | Zero-Shot | NER Entities | Groq |
|--------|-----------|-----------|--------------|------|
| Unauthorized login from IP 198.51.100.4 at 3:47 AM | NEGATIVE (0.9994) | possible anomaly (0.90) | None | HIGH |
| Routine firewall rule update on fw-01 | NEGATIVE (0.9986) | routine activity (1.00) | None | INFORMATIONAL |
| Phishing email targeting finance@company.com | NEGATIVE (0.9968) | possible anomaly (0.80) | None | HIGH |
| Multiple failed SSH attempts — 47 in 5 minutes | NEGATIVE (0.9993) | possible anomaly (0.90) | "SS" (MISC, 0.994) | CRITICAL |
| System resource utilization normal | NEGATIVE (0.9880) | routine activity (1.00) | None | INFORMATIONAL |

---

## Analysis

**Where models agreed:**  
All models correctly identified records 2 and 5 as benign in some form. Zero-Shot labeled both as "routine activity" with perfect 1.0 confidence, and Groq classified both as INFORMATIONAL with reasoning that matched expected analyst judgment. Even Sentiment showed slightly lower confidence scores on these two records (0.9880 and 0.9986) compared to the clearly threatening ones, suggesting a weak signal in the right direction even if the label itself was wrong.

All models also agreed that records 1, 3, and 4 were in some way concerning — Sentiment scored them highest, Zero-Shot flagged them as "possible anomaly", and Groq assigned HIGH or CRITICAL severity.

**Where models disagreed:**  
The most significant disagreement was between Sentiment and every other model on the benign records. Sentiment labeled all 5 records as NEGATIVE — including the routine firewall update and the normal utilization report — directly contradicting Zero-Shot and Groq, which correctly identified those as non-threatening. This makes Sentiment unreliable as a standalone classifier for this domain.

A subtler disagreement was between Zero-Shot and Groq on threat severity. Zero-Shot treated all three threat records the same way ("possible anomaly" with scores between 0.80–0.90), unable to distinguish between a phishing email and a 47-attempt brute-force SSH attack. Groq correctly escalated the SSH attack to CRITICAL while keeping the others at HIGH, showing much stronger contextual reasoning.

**Most accurate model overall:**  
Groq Llama 3.1 8B was the most accurate. It correctly classified all 5 records with appropriate severity levels and provided coherent one-sentence reasoning for each decision that aligned with real-world security analyst judgment.

**Fastest/most practical:**  
Zero-Shot classification was the most practical HuggingFace option — it returned consistent results, required no prompt engineering, and correctly separated benign from threatening records. However, Groq was both fast and accurate, making it the best overall choice for production use given the quality of its output.

---

## Recommended Models for My Capstone Component

**Component:** [Your capstone component name]

**Primary model:** Groq Llama 3.1 8B Instant — produces nuanced severity classifications (CRITICAL/HIGH/MEDIUM/LOW/INFORMATIONAL) with human-readable reasoning, making it the most directly useful model for security alert triage.

**Secondary model:** facebook/bart-large-mnli (Zero-Shot) — provides a fast, lightweight confidence-scored classification that can serve as a pre-filter before passing records to Groq, reducing API calls on clearly routine records.

**Rejected models and why:**
- **distilbert-sst-2 (Sentiment):** Trained on consumer review data, not security text. Labeled every record — including benign ones — as NEGATIVE, making it useless for distinguishing threat severity in a cybersecurity context.
- **dslim/bert-large-NER:** Extracted almost no useful entities from security alert text. It recognized only "SS" from "SSH" across all 5 records. The model was trained on news and general text, so it looks for people, places, and organizations rather than IP addresses, protocols, or technical indicators of compromise.

---

## Failure Cases and Limitations

The most significant failure was Sentiment classifying the routine firewall update (record 2) as NEGATIVE with 0.9986 confidence — nearly identical to its score on the brute-force SSH attack. This is a fundamental mismatch between the model's training domain (movie and product reviews) and security operations text. Security language is inherently negative in tone — words like "failed", "unauthorized", "detected", and "anomaly" appear in both critical alerts and routine logs. Any model trained on general sentiment cannot distinguish between these without domain-specific fine-tuning.

A secondary failure was Groq returning multi-line responses for some records, which caused formatting issues in Airtable's CSV export. In production, output should be sanitized to remove line breaks before storing in a database.

---

## Next Steps

Given more time, the next steps would be to test a domain-specific security classification model from Hugging Face — such as one fine-tuned on CVE descriptions or MITRE ATT&CK data — to see if it outperforms the general-purpose models tested here. It would also be worth testing Groq with a more structured prompt that forces a single-word severity output followed by reasoning on a new line, which would resolve the formatting issues and make the output easier to parse programmatically. Finally, testing on a larger dataset of 50+ records would reveal whether Zero-Shot's confidence scores are consistent enough to use as a reliable pre-filter threshold.
