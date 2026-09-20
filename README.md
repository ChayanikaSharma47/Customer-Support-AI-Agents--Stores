# Customer-Support-AI-Agents--Store - Tesco


## 1. Problem Framing

The goal is to build a lightweight customer-support AI agent for Tesco that can:

1. Classify an incoming customer message into a small set of support intents.
2. Draft a response grounded in Tesco's historical responses.
3. Decide whether the message should be handled automatically or escalated.
4. Provide measurable evidence of system performance.

### What good means

A useful system should:

* identify the customer's issue correctly;
* retrieve a historically appropriate Tesco response;
* avoid inventing policies, refunds, procedures, or facts;
* identify cases that should not be handled automatically;
* provide evidence through a labelled evaluation set rather than relying only on model outputs.

### What was not built

This prototype does not attempt to:

* connect to Tesco's internal systems;
* issue refunds or make account changes;
* provide real-time order information;
* guarantee production-level customer-support decisions;
* replace human support agents.

---

## 2. Data and Evaluation

The source dataset contains **25,111 Tesco customer-support conversation pairs**.

A manually labelled **168-example golden evaluation set** was created, satisfying the required 150–250 example range.

The labelled data contains 11 intent categories, including:

* store issue
* product quality
* online/website
* product availability
* order delivery
* payment/card
* Clubcard/voucher
* price/promotion
* general FAQ
* other support
* out-of-scope

The same evaluation set was used to assess intent classification, escalation classification, and reply retrieval.

For reply retrieval evaluation, exact customer-message matches were excluded. This was necessary because **131 of the 168 evaluation messages had exact matches in the original dataset**. Including them would make retrieval performance misleadingly high.

---

## 3. Intent Classification

The main intent classifier uses **TF-IDF + Logistic Regression**.

Two baselines were also evaluated:

| Method                       |   Accuracy |
| ---------------------------- | ---------: |
| Majority-class baseline      | **23.53%** |
| Keyword/rule baseline        | **26.47%** |
| TF-IDF + Logistic Regression | **35.29%** |

The trained classifier therefore performed better than both simple baselines on the 34-example intent test split.

However, the test set is small and several intents have very few examples. Therefore, the 35.29% headline accuracy should not be interpreted as equal performance across all intents.

---

## 4. Escalation Classification

A separate **TF-IDF + Logistic Regression** model was trained to predict whether a customer message should be escalated.

| Method                       |   Accuracy |
| ---------------------------- | ---------: |
| Majority-class baseline      | **55.88%** |
| TF-IDF + Logistic Regression | **67.65%** |

The model's classification report showed:

* `no` recall: **0.95**
* `yes` recall: **0.33**
* overall accuracy: **0.6765**

The low recall for the `yes` class is important because some messages labelled for escalation are missed.

---

## 5. Reply Drafting

The prototype uses **historical-response retrieval** rather than relying entirely on generative text.

TF-IDF is used to find a historically similar customer message, and its associated Tesco response becomes the candidate reply.

For evaluation, exact customer-message matches were excluded.

The resulting retrieval similarity had:

* Mean: **0.565**
* Median: **0.473**

This is a retrieval-similarity diagnostic, not a correctness score.

### Reply similarity

TF-IDF cosine similarity between the retrieved response and the historical Tesco response had:

* Mean: **0.224**
* Median: **0.039**
* Maximum: **0.925**

The low median shows that many retrieved responses had little lexical overlap with the corresponding historical response.

---

## 6. LLM-as-Judge Validation

A 20-example sample was manually labelled by a human and independently evaluated using LLaVA as an LLM judge.

Results:

* Human/LLM agreement: **55%**
* Cohen's κ: **0.043**

This indicates weak agreement.

Therefore, the LLM judge was **not treated as a reliable standalone measure of reply quality**. Human-labelled examples were retained as the more important quality reference.

---

## 7. Top 5 Failure Modes

### 1. Wrong historical match for availability issues

A customer reported that items disappeared from an online basket. The retrieved response was about delivering a TV from another store.

**Hypothesis:** TF-IDF can match overlapping words without understanding the underlying issue.

### 2. Poor retrieval for product-availability problems

A customer reported Halloween food disappearing after login. The retrieved response was a generic Halloween promotional message.

**Hypothesis:** shared topic words such as “Halloween” and “food” can dominate the retrieval signal.

### 3. Positive/neutral messages retrieved as complaints

A customer said:

> “Also this is a good section. Definitely my fave.”

The retrieved response was a store-complaint response.

**Hypothesis:** the retrieval system does not understand sentiment or conversational intent.

### 4. Short messages provide weak retrieval signals

A customer wrote:

> “6 pack!!! Someone in the bakery can't count.”

The retrieved response concerned Clubcard cards.

**Hypothesis:** very short messages contain insufficient lexical information for reliable TF-IDF retrieval.

### 5. Payment issue matched to unrelated promotional content

A customer reported being charged twice because of a broken card machine. The retrieved response was about a Tesco planner.

**Hypothesis:** lexical retrieval can fail when the correct response requires understanding the specific context of the payment problem.

---

## 8. What Is Misleading About My Headline Number?

The **35.29% intent accuracy** is higher than both baselines, but accuracy hides the uneven distribution of intents. Several intents have very few test examples, so this number does not represent performance equally across all customer issues.

The **67.65% escalation accuracy** also requires context. The escalation class had only **0.33 recall**, meaning that some examples requiring escalation were missed.

The **0.224 reply similarity** is not a correctness or helpfulness score. A response can use different wording while being correct, or share words with a response while addressing the wrong issue.

Finally, the LLM judge achieved only **55% human agreement with κ = 0.043**, so its output should not be treated as reliable standalone evidence of reply quality.

---

## 9. Next Week Plan

1. Replace basic TF-IDF retrieval with semantic retrieval.
2. Clean historical replies by removing usernames, URLs, thread markers, and other conversation noise.
3. Add more labelled examples for under-represented intents.
4. Evaluate intent performance separately for each class.
5. Improve escalation recall.
6. Expand human evaluation of reply quality.
7. Introduce confidence thresholds for automatic handling.
8. Send low-confidence or poor retrieval cases to human support.

---

## 10. Decision Log

| #  | Decision                                          | Reason                                                                 |
| -- | ------------------------------------------------- | ---------------------------------------------------------------------- |
| 1  | Use Tesco as the target brand                     | Provides a large set of customer-support conversations.                |
| 2  | Use the customer message as the primary input     | Represents the incoming support request.                               |
| 3  | Define a limited intent set                       | Makes classification practical and measurable.                         |
| 4  | Use TF-IDF + Logistic Regression                  | Simple, transparent and fast baseline model.                           |
| 5  | Add a keyword/rule baseline                       | Provides a transparent comparison.                                     |
| 6  | Add a majority-class baseline                     | Provides a trivial reference point.                                    |
| 7  | Use historical Tesco replies                      | Keeps response drafting grounded in historical brand behaviour.        |
| 8  | Exclude exact matches during retrieval evaluation | Prevents artificially inflated retrieval results.                      |
| 9  | Use cosine similarity as a reply diagnostic       | Provides a reproducible automated similarity measure.                  |
| 10 | Validate the LLM judge against humans             | Tests whether automated judging is trustworthy.                        |
| 11 | Do not rely on the LLM judge alone                | Human agreement was only 55%, with κ = 0.043.                          |
| 12 | Consider low-confidence cases for escalation      | Retrieval failures show that automatic responses can be inappropriate. |

## 11. Conclusion

The prototype demonstrates a measurable improvement over simple baselines for intent classification and escalation detection. However, the reply-retrieval experiments show important weaknesses, particularly when messages are short, contextual, positive/neutral, or lexically ambiguous.

The evaluation therefore supports a **prototype rather than a production-ready autonomous support agent**. The strongest next improvements are better semantic retrieval, cleaner historical responses, more balanced evaluation data, and stronger human-centred evaluation of reply quality.
