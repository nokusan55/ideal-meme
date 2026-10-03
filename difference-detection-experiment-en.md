# Can a model detect "difference in content" without relying on words? — Current notes on two features

**Epistemic status**: A follow-up weekend experiment. I looked at a small piece of a much bigger question — "can a model tell whether a situation is 'surprising' without using words for it?" — by examining two specific features. Results so far are mostly negative. Investigation ongoing.

*This post's experimental design, execution, and data reading were done by me. AI (Claude) assisted with organizing and drafting the writeup.*

## Motivation

Humans seem to notice the "gap" between what they expected and what actually happened — this may be one of the more basic functions underlying consciousness or attention. If true, I wondered whether a language model might also have some internal component that detects "does this sentence contradict common sense or expectation," independent of any specific words.

## Experiment 1: Feature 3500 ("surprise")

Using Neuronpedia (Gemma-2-2B, 12-gemmascope-transcoder-16k, ID 3500, labeled "words related to surprise, bewilderment, and astonishment"), I tested its response to vocabulary around surprise.

### Results

| Sentence | Condition | Activation |
|---|---|---|
| I was surprised that the sun rose in the west this morning. | Has "surprise," contradicts reality | 25.00 |
| I was surprised that the sun rose in the east this morning. | Has "surprise," consistent with reality | 25.00 |
| I was astonished / shocked that... | Synonyms | 25.13 / 26.38 |
| I was not surprised that... | Negated | 9.38 |
| It was a surprising morning. | Adjective form | 13.13 |
| Wow, the sun rose in the west this morning! | No "surprise"-family word | 0.0000 |
| Everyone knows the sun rises in the east every day without exception. Today, it rose in the west. (long context spelling out the contradiction, no "surprise"-family word) | No "surprise"-family word | 0.0000 |
| I was surprised that [a flower bloomed overnight / metal expanded in the cold / a dog could swim across a river / she finished a marathon in two hours / an old computer still turned on] | 5 different domains | All 25.00 |
| This is impossible. / Something is very wrong. (other anomaly vocabulary, no "surprise"-family word) | No "surprise"-family word | Both 0.0000 |

### Conclusion (Feature 3500)

$$\boxed{\text{Feature 3500 does not judge whether the content of a sentence is actually surprising. It returns a near-constant value (~25) whenever a "surprise"-family word is present, regardless of content — a lexical detector.}}$$

Varying the domain (biology, physics, animals, humans, machines) made no difference; swapping east/west, or spelling out the contradiction in a long context, made no difference either — without a "surprise"-family word, activation was always zero. Synonyms triggered similarly strong activation; negation and adjective forms were only partially tracked.

## Experiment 2: Feature 1023 ("words and phrases with strong emotional connotations")

While comparing Circuit Tracer graphs for `The sun rose in the west/east this morning.`, both "west" and "east" showed nearly identical input contribution (+0.55) to this node (Gemma-2-2B, 6-gemmascope-transcoder-16k, ID 1023), yet the standalone feature page showed differing activation (west=4.06, east=2.80). I investigated this discrepancy further.

### Results

| Sentence | Activation |
|---|---|
| The sun rose in the west this morning. | 4.06 |
| The sun rose in the east this morning. | 2.80 |
| The moon appeared in the west this morning. | 1.14 |
| The moon appeared in the east this morning. | 1.70 |

### Conclusion (Feature 1023)

$$\boxed{\text{For "sun," west > east; for "moon," east > west — the direction reverses. This does not support the idea that the feature consistently tracks the direction word itself; the variation may instead reflect unexplained word-combination effects.}}$$

This doesn't support Feature 1023 being a "direction detector." Notably, this node's activation density is relatively high (4.6%) and it fires strongly in unrelated contexts (e.g., "nail," "pudding"), so it may simply be unrelated to direction altogether.

### Follow-up: this node turns out to be an idiom / figurative-expression detector

Looking through roughly 30 of its top activating examples revealed a clear pattern: "fought tooth and nail," "the proof is in the pudding," "writing on the wall," "pushes the envelope," "ignorance is bliss," "meeting of the minds," "fanning the flames," "like a lead balloon," "going for broke," "give up the ghost," "ticking time bomb," "the sky was falling," "separate the wheat from the chaff," "safety in numbers," "melting pot," and more — **nearly all of the top examples were fixed figurative idioms, not literal language.**

$$\boxed{\text{Feature 1023 has little to do with direction; it's a detector for English idioms and figurative expressions.}}$$

This also explains the earlier inconsistent west/east results (west > east for "sun," east > west for "moon"): phrases like "appeared in the west" or "rose in the west" happen to have a cadence close to a fixed expression, triggering a faint response — not because the feature was tracking direction at all.

What started as an attempt to investigate west/east ended up surfacing a node with a completely different, unexpected role (idiom detection) — a fairly typical experience in interpretability research.

## Overall summary (as of now)

Across the two features examined, I have not yet found evidence that the model detects "difference" or "contradiction" in content independent of specific words. What I found instead was a detector keyed to specific vocabulary (the "surprise" word family), and a feature with a completely different, unrelated role (idiom/figurative-expression detection).

This is not proof that "no difference-detecting component exists" — only that it wasn't found in these two particular features. The second finding in particular is a concrete example of a common experience in interpretability work: stumbling onto a feature with an unexpected role, different from the one you were originally looking for.

## Directions for future checks (not yet attempted — recorded only)

The following are ideas I haven't tested yet. Each would need the same treatment (hypothesis → contrast test → attempt to disprove) before being trusted.

- Check whether other directional/positional pairs (north/south, left/right) similarly turn out to have an unrelated "true" role
- Response to the more abstract concept of symmetry/asymmetry
- Searching for features that respond to numerical inconsistency (e.g., the "which is bigger, 1031 or 1034" pattern seen in an unrelated context earlier)
- Further testing the "idiom detector" hypothesis for Feature 1023 against plain, non-idiomatic sentences

## Limitations

- A very small investigation — only two features examined
- The relationship between a node's "input contribution" shown in the full graph view and its activation on the standalone feature-test page is not yet fully understood; the two don't always agree
- "Difference detection" is a large concept, and this is only an early, exploratory pass around its edges
