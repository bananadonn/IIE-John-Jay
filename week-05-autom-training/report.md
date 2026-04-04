# Week 5 Report: AutoML Training & Fine-Tuned Model Evaluation

**Name:** Donald Reith
**Date:** 2026-04-03
**Capstone Project:** Automated Threat Intelligence Hub
**My Component:** Feed Collector

---

## Part A: Teachable Machine Training

### Training Setup

- **Task:** Phishing vs Legitimate email screenshot classification
- **Training images per class:** 30
- **Test images per class:** 5
- **Total training time:** ~10 seconds

### Test Results

| # | Actual Class | Predicted Class | Confidence | Correct? |
|---|--------------|-----------------|------------|----------|
| 1 | Phishing | Phishing | 63% | Yes |
| 2 | Phishing | Phishing | 81% | Yes |
| 3 | Phishing | Legitimate | 59% | No |
| 4 | Phishing | Legitimate | 100% | No |
| 5 | Phishing | Legitimate | 91% | No |
| 6 | Legitimate | Legitimate | 99% | Yes |
| 7 | Legitimate | Legitimate | 100% | Yes |
| 8 | Legitimate | Legitimate | 76% | Yes |
| 9 | Legitimate | Legitimate | 93% | Yes |
| 10 | Legitimate | Legitimate | 100% | Yes |

### Confusion Matrix

| | Predicted: Phishing | Predicted: Legitimate |
|---|---|---|
| **Actual: Phishing** | TP = 2 | FN = 3 |
| **Actual: Legitimate** | FP = 0 | TN = 5 |

### Calculated Metrics

- **Accuracy:** 70% — (2+5) / (2+5+0+3) = 7/10
- **Precision:** 100% — 2 / (2+0)
- **Recall:** 40% — 2 / (2+3)
- **F1 Score:** 57.1% — 2 × (1.0 × 0.4) / (1.0 + 0.4)

### Interpretation

I was honestly surprised by how well the model identified legitimate images — all five were correct with high confidence which I wasn't expecting given how little time the training took. The phishing results were more in line with what I expected. The model missed 3 out of 5 phishing images and in some cases was extremely confident it was legitimate which is the wrong direction for a security tool. I've trained models before in Google Colab where I had to manually tune weights, loss functions, and run through epochs which took a long time, so getting a working model in about 10 seconds with Teachable Machine was a lot easier by comparison. To improve the phishing recall I would add more diverse training images covering different email clients, spoofed brands, and layouts so the model doesn't just learn one specific phishing style.

---

## Part B: Generic vs Fine-Tuned Model Comparison

### Models Tested

1. **Generic:** distilbert-base-uncased-finetuned-sst-2-english — binary sentiment analysis (POSITIVE/NEGATIVE), trained on product and movie reviews
2. **Fine-Tuned A:** mrm8488/bert-tiny-finetuned-sms-spam-detection — classifies text as SPAM or HAM, fine-tuned on SMS messages to detect suspicious or unsolicited content
3. **Fine-Tuned B:** cardiffnlp/twitter-roberta-base-sentiment-latest — 3-class sentiment (negative/neutral/positive), fine-tuned on short social media text which is similar in length to CVE summaries and RSS headlines

### Results

| Input | Generic Label (Score) | Fine-Tuned A Label (Score) | Fine-Tuned B Label (Score) | Best Model |
|-------|-----------------------|----------------------------|----------------------------|------------|
| Unauthorized login from IP | NEGATIVE (0.9961) | Legitimate (0.7045) | neutral (0.8441) | Generic |
| Routine firewall rule update | NEGATIVE (0.9986) | spam (0.5376) | neutral (0.8743) | Fine-Tuned B |
| Phishing email with spoofed domain | NEGATIVE (0.9959) | Legitimate (0.7415) | negative (0.5479) | Fine-Tuned B |
| Multiple failed SSH attempts | NEGATIVE (0.9994) | spam (0.5719) | negative (0.8257) | Fine-Tuned B |
| System resource utilization normal | NEGATIVE (0.9880) | Legitimate (0.8388) | neutral (0.6699) | Fine-Tuned B |

### Analysis

**Generic model strengths:** The generic model was consistent and confident across all five records. If you only looked at the scores you might think it was doing well.

**Generic model weaknesses:** It called everything NEGATIVE including the routine firewall update and the normal system utilization report. That was the same problem we saw in Week 4 — it was trained on reviews and product text so words like "failed" and "unauthorized" always read as negative to it regardless of what is actually happening. It cannot tell the difference between a real threat and a scheduled maintenance log.

**Fine-tuned model advantage:** Fine-Tuned B was clearly the best of the three. It output negative for the phishing email and the SSH brute force attack and neutral for the two routine records which is exactly the kind of triage logic the feed collector needs. Fine-Tuned A was harder to trust — it flagged the SSH attack correctly but then called an unauthorized login legitimate which does not make sense from a security perspective.

**Biggest surprise:** The most surprising result was Fine-Tuned A labeling the routine firewall update as spam while calling the unauthorized login legitimate. I expected it to be the other way around if anything. It seems like it was picking up on word patterns rather than understanding context which is a good reminder that applying a model outside of its training domain can produce results that seem backwards.

### Recommended Model for My Capstone Component

**Component:** Feed Collector

The feed collector is essentially a cloud researcher that is constantly checking different cybersecurity information sources for threats even while you are asleep, doing the tedious work of going through CVE databases and RSS feeds so analysts do not have to. For that use case I am recommending Fine-Tuned B.

**Primary model:** cardiffnlp/twitter-roberta-base-sentiment-latest — the three class output fits naturally into how the feed collector works. Negative entries get escalated to the threat queue, neutral entries get logged and deprioritized, and positive entries like resolved vulnerabilities get archived. It is the only model that actually differentiated between the threatening and routine records in a way that would be useful in production.

**Confidence threshold:** I would set the threshold at 0.75 for automatic escalation. Anything between 0.50 and 0.74 goes to human review and anything below 0.50 gets logged as routine. The two correctly flagged threats came in at 0.548 and 0.826 so 0.75 feels like a reasonable cutoff that catches high confidence threats without overwhelming analysts.

**Priority metric:** Recall — the feed collector cannot afford to miss real threats. A false positive just means an analyst looks at something that turns out to be fine which is annoying but recoverable. A false negative means a real CVE or attack indicator never gets surfaced which could mean a vulnerability goes unpatched or an active threat goes undetected. Missing threats is the worse outcome so recall has to be the priority.

---

## Limitations & Next Steps

The biggest limitation of this week's evaluation is that 5 test records is just not enough to be confident in any of these numbers. The metrics would look very different with 50 or 100 records. Choosing which fine-tuned models to use was also genuinely the hardest part of this lab — there are a lot of options on Hugging Face and as someone who is naturally indecisive it took time to narrow it down to two that felt relevant to the feed collector. I am happy with the choices I made but I think with more time I would want to test a model that was actually fine-tuned on cybersecurity text like CVE descriptions or MITRE ATT&CK data rather than adapting sentiment and spam models. A purpose built classifier trained on threat intelligence data would likely outperform everything tested this week for this specific use case. I would also go back to the Teachable Machine classifier and retrain it with more diverse phishing images to bring that recall up from 40%.
