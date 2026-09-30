# A model component that seems to respond to the word "or" rather than the number "2"

**Epistemic status**: A weekend personal experiment. I initially hypothesized "a component that responds to binary choice," but follow-up experiments led me to revise this to a simpler explanation (response to the word "or"). Checked on one model, a couple of features, and many sentences — not a systematic study.

*This post's experimental design, execution, and data reading were done by me. AI (Claude) assisted with organizing and drafting the writeup.*

## The starting point (hypothesis, original form)

*Note: this was my starting intuition, later substantially revised.*

I had a vague intuition that the number "2" is somehow specially tied to the concept of "boundary" or "binary choice." If that were true, I thought, a language model's internals might contain a dedicated component that responds to the *structure* of "binary choice" rather than to the digit "2" itself.

## Method

Using [Neuronpedia](https://www.neuronpedia.org/)'s Circuit Tracer (Gemma-2-2B, GemmaScope Transcoder), I searched for features related to "choice." I picked two top candidates:

- **Feature 498** (layer 10, gemmascope-transcoder-16k) — labeled "choices and decisions"
- **Feature 4482** (layer 2, gemmascope-transcoder-16k) — labeled "choices and options"

First, I compared activation strength across three sentences:

1. `You can choose coffee or tea.` (binary choice)
2. `I have two apples on the table.` (just the number 2, not a choice)
3. `You can choose from many flavors.` (a choice, but not using "or")

Based on this, to **disentangle "two vs. three options" from "presence of 'or'"**, I tested additional sentences on Feature 498:

4. `You can choose tea or coffee.` (binary, reversed order)
5. `You can choose coffee, tea, or juice.` (ternary choice)
6. `You can choose apple, orange, or banana.` (ternary, different word group)
7. `You can choose apple or orange.` (binary, fruit)
8. `You can choose apple, orange.` (no "or," plain enumeration)

Finally, to check **whether the feature responds to the literal word "or" or specifically to "or" presenting a choice**, I tested sentences with "or" used in non-choice senses:

9. `It's about five or six o'clock.` ("or" meaning "approximately," not a choice)
10. `Finish your homework, or else.` ("or" as a warning/threat, not a choice)

## Results

**Feature 498 ("choices and decisions", layer 10, gemma-2-2b/10-gemmascope-transcoder-16k)**

| Sentence | Structure | Activation |
|---|---|---|
| You can choose coffee or tea. | Binary, has "or" | 6.44 |
| You can choose tea or coffee. | Binary, has "or" (reversed) | 6.69 |
| You can choose coffee, tea, or juice. | Ternary, has "or" | 7.38 |
| You can choose apple or orange. | Binary, has "or" (fruit) | 5.81 |
| You can choose apple, orange, or banana. | Ternary, has "or" (fruit) | 6.19 |
| You can choose apple, orange. | No "or," plain enumeration | 2.05 |
| I have two apples on the table. | Neither a choice nor enumeration | 0.0000 |
| You can choose from many flavors. | A choice, but no "or" | 0.0000 |
| It's about five or six o'clock. | Has "or," but not a choice (approximation) | 0.0000 |
| Finish your homework, or else. | Has "or," but not a choice (warning) | 0.0000 |

*All values above were recorded in the same session, with model/layer/ID confirmed via the breadcrumb on screen each time (this is the confirmed, second-pass dataset).*

**Feature 4482 ("choices and options", layer 2, gemma-2-2b/2-gemmascope-transcoder-16k)**

| Sentence | Activation |
|---|---|
| You can choose coffee or tea. | 3.34 |
| I have two apples on the table. | 0.0000 |
| You can choose from many flavors. | 9.13 (strongest on "from") |

## Interpretation

The original hypothesis (that the number "2" itself is special) was not supported. The number "2" never triggered activation in either feature (`two apples` is always 0.0000).

**A follow-up round of experiments then overturned my second-pass interpretation too — that the feature responds to the "meaning" of binary choice.** The decisive contrast was this:

$$\text{apple, orange (no "or," 2.05)} \quad \text{vs.} \quad \text{apple or orange (has "or," 5.81)}$$

The same two words (apple, orange) — but adding "or" alone raises activation roughly 2.8×. Comparing binary choice (coffee or tea = 6.44, apple or orange = 5.81) against ternary choice (coffee, tea, or juice = 7.38, apple, orange, or banana = 6.19), both fall in a similar range — "two options vs. three" has no clear effect on activation strength.

> Feature 498 appears to respond not to "choosing between two things" specifically, but to **the word "or," or more precisely, to the sentence structure of "presenting multiple options via 'or.'"**

**However, this interpretation was itself refined by a further, decisive experiment.** Even when "or" is present, if it does not present a choice — `five or six o'clock` (approximation), `homework, or else` (a warning) — activation was exactly zero (0.0000).

$$\boxed{\text{Feature 498 is not responding to the surface token "or." It responds specifically to the structure of "presenting multiple options via 'or.'"}}$$

English "or" plays several distinct roles (presenting options, expressing approximation, issuing a warning). Feature 498 responds only to the option-presenting role — confirming this isn't naive string-matching on the token "or."

**Secondary observation**: sentences using drink words (coffee, tea, juice) consistently scored somewhat higher than matched sentences using fruit words (apple, orange, banana) — e.g., coffee or tea = 6.44 vs. apple or orange = 5.81. This effect is much smaller than the "or" effect, but suggests some lexical influence beyond structure alone.

Feature 4482 behaves differently from Feature 498: it responds most strongly to `many flavors` (a choice that doesn't use "or"), which suggests it may be tracking something closer to "choice" in a broader sense, rather than specifically the token "or." Why the two features diverge this way is not yet explained.

## Limitations

- Small sample: one model, a small number of features
- Still unexplained why Feature 4482 responds so strongly to "many flavors"
- Haven't checked how the model defines "presenting options" as a structure — e.g., whether other connectives, or constructions like "either...or," trigger the same response
- The maximum activation for Feature 498 across the model's full dataset is in the 58–60 range; the values observed here (2–7) are only about a tenth of that. This is an observation within a weak-activation regime
- **Methodological note**: when measuring on Neuronpedia, browser auto-translate must be disabled, and the breadcrumb (model name / layer number / feature ID) must be checked every time. The same feature ID (e.g., 4482) refers to a completely different feature if the layer differs, and on-screen auto-translation can cause misreading of token labels. The numbers in this post are re-measured values with these controlled for.

## Next steps

- Check why Feature 4482 responds so specifically to "many flavors," using other expressions for "many options"
- Test other option-presenting constructions, such as "either...or"
- I attempted to replicate this on another model, but Gemma-2-9B does not yet have inference enabled on Neuronpedia (i.e., no way to test custom sentences), so I couldn't check it this time. Will try on another model where inference is available.
