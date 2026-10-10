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

## Experiment 3: Two features that deal with arithmetic correctness

Since "surprise" and "idiom detection" both turned out unrelated to truth/falsity of content, I looked for features more directly tied to logical correctness.

**Feature 9758** ("arithmetic," layer 8, 8-gemmascope-transcoder-16k): comparing `2 + 2 = 4` and `2 + 2 = 5`, both scored exactly **19.25**.

**Feature 204** ("calculations," layer 14, 14-gemmascope-transcoder-16k): the same two sentences both scored **0.0000**.

$$\boxed{\text{Neither feature judges whether the calculation is correct. Feature 9758 fires whenever an equation is present at all; Feature 204 doesn't fire under this condition — different patterns, but both indifferent to correctness.}}$$

**I also tried comparing full Circuit Tracer graphs** for `2+2=4` vs. `2+2=5`. But as with the west/east comparison earlier, the graphs looked similar enough that I couldn't tell whether there was a meaningful difference just by eye — this would require checking individual nodes one by one, which I didn't complete this round.

## A side experiment: does layer depth mean "smarter"? (tentative)

I did a quick check of how the same concept, "choice," looks different depending on layer depth. Details and the latest measurements are in the other post (binary-choice-feature-experiment), under "Additional experiment."

**Feature 4482** ("choices and options," layer 2, shallow) on `You can choose coffee or tea.`: "coffee" scored 1.01, "or" scored 3.34.

**Feature 5328** ("phrases related to preferred methods / choice," layer 19, deep) on the same sentence: **0.0000**. But on `Coffee is my drink of choice.` "of" scored 69.00, and on `The weapon of` "of" scored 68.00 — while `The method of` gave no response. My earlier explanation ("specialized for the construction 'of choice'") was too simple; at present it is only an unconfirmed hypothesis that the feature fires on word combinations where "choice" is likely to come next (and why `method of` gives 0 is unexplained).

So I can't yet say either "deeper layers respond more intelligently" or "layers swap in different specialists." I examined only a few layers and features, so both remain hypotheses.

## A methodological takeaway

Across 5 independent features (the "or"-structure detector, the "surprise" vocabulary detector, the idiom detector, and two arithmetic-related features), all five led to the same conclusion: response to words/form, not to truth, falsity, or validity of content.

$$\boxed{\text{All five independent candidates reached the same conclusion: response to words or form, not to the truth of the content.}}$$

This is not proof that no "difference detection" exists in the model — rather, it suggests that **examining individual features (neurons) one at a time may itself be running into a limit**. If something resembling "judging truth" exists in the model, it's likely not localized to a single node, but distributed across a "computational process" involving multiple nodes working together — something that, in principle, may never show up by searching single nodes one at a time.

## Overall summary (as of now)

Across 5 independent features examined (surprise vocabulary, idiom detection, two arithmetic-related features, and the west/east node), I have not found evidence that the model detects content-level difference, contradiction, or correctness independent of specific words or surface form. What I found instead were detectors tied to vocabulary or to the mere presence of a particular form (e.g., "an equation is present"), none of which cared about the content's truth or validity.

This is not proof that "no difference-detecting component exists" in the model — rather, it's a limited result suggesting that **searching one feature (neuron) at a time may itself be hitting a limit**. Five is still a small number, but it's notable that every independent search pointed the same way.

## Directions for future checks (not yet attempted — recorded only)

- **(Most important next step)** Learn a more advanced method for analyzing circuits — the coordinated behavior of multiple nodes — rather than single features. This likely exceeds my current skill level and is recorded as the next big challenge.
- Check whether other directional/positional pairs (north/south, left/right) similarly turn out to have an unrelated "true" role
- Response to the more abstract concept of symmetry/asymmetry
- Further testing the "idiom detector" hypothesis for Feature 1023 against plain, non-idiomatic sentences

## Limitations

- A very small investigation — only two features examined
- The relationship between a node's "input contribution" shown in the full graph view and its activation on the standalone feature-test page is not yet fully understood; the two don't always agree
- "Difference detection" is a large concept, and this is only an early, exploratory pass around its edges
