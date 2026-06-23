# TakeMeter - plannning.md

---

## 1. Community Context
* **Target Community:** `r/LetsTalkMusic` (Specifically focusing on pop-culture and demographic-driven music debates).
* **Justification:** This forum is an active, text-heavy environment where users treat pop-culture topics with academic seriousness. It is an ideal fit for fine-tuning a classifier because the discourse naturally stratifies into distinct quality tiers: deep historical critique, unevidenced ideological hostility, and highly personal nostalgic throwaways.

## 2. Label Taxonomy
Our fine-tuned model categorizes discrete contributions into three mutually exclusive primitives:

### `analysis`
* **Definition:** The comment bypasses raw emotional framing to construct an objective sociological, historical, or industry-mechanic argument backed by specific, verifiable pop-culture precedents.
* **Example 1:** I think you're looking at it from the wrong angle.
It's not necessarily that people dismiss an artist's music because teenage girls like it. The bigger factor is that many artists with predominantly teenage female fanbases were deliberately designed and marketed to appeal to that demographic in the first place.
For decades, the music industry has created acts specifically aimed at young teenage girls, often prioritizing image, marketability, and emotional appeal over artistic innovation. As a result, critics sometimes associate those artists with a more commercial or "manufactured" approach to music.
There are obviously exceptions. Bands like The Beatles or The Rolling Stones attracted huge numbers of teenage girls, but they weren't originally assembled by record labels as products for that audience. Their popularity among teenage girls was a consequence of their success, not the reason they existed.
So the stereotype isn't really "teenage girls like it, therefore it's cheap." It's more that many acts that are perceived as manufactured happen to be targeted primarily at teenage girls, and people often confuse the audience with the marketing strategy behind the artist.
* **Example 2:** "That's because at one point, straight boys actually made up a material part of pop fandom. As they have moved into other media like YT and gaming, they've basically abandoned mainstream pop. Criticizing music they don't even engage with doesn't even occur to them."

### `hot_take`
* **Definition:** A bold, highly confident assertion, ideological decree, or terminal dismissal stated as absolute fact without supporting evidence, multi-variable reasoning, or nuance.
* **Example 1:** "Because patriarchy hates women. Don't worry about all the long-winded explanations."
* **Example 2:** "20 years teaching music and 25 years as a professional musician. The men of the industry will literally scoff at most things my teenage girl students like... they don't do that to the stuff young men like but it's not any better It's ok for young men to obsess over their favourite artist, but not young women.
ABSURD."

### `anecdote`
* **Definition:** A first-person subjective narrative or nostalgic self-disclosure that engages with the discourse strictly through personal vulnerability, lived experience, or immediate physical reaction rather than third-person data.
* **Example 1:** "I am male in my early 50s who gets ridiculed online because the Twilight books are my all-time favorite book series.
People try to mock, dismiss, disparage, discredit, make fun of and deny the excellent writing, story and world that author Stephenie Meyer created.
It was one of the biggest cultural events of my lifetime, but in 2026 if you mention those books on book subreddits you will get laughed at, attacked and downvoted for no reason even though it's one of all-time greatest selling book series and movie franchises in entertainment history.
I feel the same way about the questions you are asking and trying to discuss regarding music based on your post topic"
* **Example 2:** "OMG the number of times I watched the Wild Boys video drooling over JT. But their music was good. The stuff teenage girls line today in crap, of course."

## 3. Hard Edge Cases & Resolution Law

### 3.1 Deterministic Decision Rule
Strip away all first-person framing and subjective adjectives. If the remaining text provides concrete, verifiable third-person pop-culture precedents that logically support the claim on their own, classify as `analysis`. If the third-person framing is purely decorative and the core payload relies on subjective, unevidenced pejoratives or decrees, classify deterministically as `hot_take`.

### 3.2 Documented Empirical Edge Cases (Milestone 3 Audit)

| CSV Row | Raw Forum Text Snippet | Linguistic Conflict | Resolved Label | Deterministic Justification |
| :--- | :--- | :--- | :---: | :--- |
| **Row 23** | "OMG the number of times I watched the Wild Boys video drooling over JT. But their music was good. The stuff teenage girls line today in crap, of course." | Opens with nostalgic self-disclosure (`anecdote`), ends with an absolute, unevidenced dismissal (`hot_take`). | `anecdote` | Core payload is lived personal vulnerability; the closing pejorative is secondary, decorative throwaway text. |
| **Row 39** | "I think part of the issue here is there are at least 3 elements at play here: 1: the marketability element discussed above 2: young people often have divisive and new opinions on pop music 3: people fucking hate teenage girls" | Uses academic numbered list formatting (`analysis`), but the thesis is an aggressive subjective decree (`hot_take`). | `hot_take` | The numbered formatting is decorative; the core argument relies entirely on an unevidenced generalization. |
| **Row 143** | "Yes. This is huge. And idk. Might be people not secure in their masculinity sometimes? The Cure, Beach Boys, Empire of the Sun, Tame Impala, The Monkees 5 groups/artists off the top of my head that are utterly incredible, but people act the way you mentioned about them, and they don't get the respect they deserve from the masses." | Opens with unevidenced psychological speculation (`hot_take`), pivots to citing historical band precedents (`analysis`). | `analysis` | Concrete pop-culture precedents independently support the claim once the speculative opening is stripped away. |

## 4. Data Collection Plan
* **Sourcing:** Manually scraping at least 200 discrete comments from the locked `r/LetsTalkMusic` target discussion thread into a single, unsplit CSV (`text`, `label`, `notes`).
* **Imbalance Mitigation:** We will audit the label counts upon reaching 200 rows. If any single class exceeds a **70% majority ceiling**, we will pause and selectively scrape secondary analytical threads within `r/LetsTalkMusic` to pad the minority buckets until reaching a stable $\ge 20\%$ distribution per label.

## 5. Evaluation Metrics
To evaluate `distilbert-base-uncased` against the zero-shot baseline, we reject overall accuracy as a standalone metric. We will track:
1. **Macro F1-Score:** Because our dataset will likely have minor class imbalances, Macro F1 calculates the harmonic mean of precision and recall for each label independently and averages them, preventing a majority class from masking minority failure modes.
2. **Directional Confusion Matrix:** We will map true vs. predicted labels to identify specific semantic leakages (e.g., tracking if the model systematically confuses short `analysis` posts for `hot_take` posts).

## 6. Definition of Success
To consider this fine-tuned classifier genuinely useful for deployment in an automated forum moderation interface, it must hit two strict numerical thresholds:
1. **Absolute Performance:** Hit a **Macro F1-Score of >= 0.75** across all three target primitives on the locked test set.
2. **Delta Performance:** Demonstrate an absolute overall accuracy improvement of **>= 15%** over the zero-shot prompt baseline (`llama-3.3-70b-versatile`).

## 7. AI Tool Plan
## 7. AI Tool Plan

### 7.1 Label Stress-Testing (Adversarial Generation)
* **Tool:** `ChatGPT (GPT-4o)` / `Claude 3.5 Sonnet`
* **Input:** My Section 2 Label Taxonomy (`analysis`, `hot_take`, `anecdote`) and my Section 3 Deterministic Decision Rule.
* **Prompt Directive:** *"Act as an adversarial linguist. Read my 3 definitions and my tie-breaking rule. Generate 5 to 10 synthetic, highly complex forum posts that sit directly on the razor's edge between `analysis` and `hot_take`, or `analysis` and `anecdote`. Your goal is to write posts that break my rubric."*
* **Verification Plan:** I will take the synthetic boundary posts and attempt to manually classify them. If the AI succeeds in generating a post that I cannot deterministically assign to exactly one label using my Section 3 rule, it proves my definitions contain linguistic flaws. I will revise Section 2 to tighten the boundary before annotating real data.

### 7.2 Annotation Assistance & Tracking
* **Tool:** `None (Manual  Workflow)`
* **Workflow & Rationale:** I explicitly opted *out* of using an LLM to pre-label my dataset.

### 7.3 Failure Analysis
* **Tool:** `Claude 3.5 Sonnet`
* **Input:** The raw array of test-set examples where my fine-tuned DistilBERT model made an incorrect prediction (extracted from `evaluation_results.json`).
* **Note on Patterns & Verification:** I will prompt the LLM to identify macro linguistic patterns in the errors (e.g., *"The model systematically misclassifies short analysis posts as hot takes"*). I will verify these hypotheses myself by manually cross-referencing the flagged errors against the raw text before authoring the Evaluation Report in my README.