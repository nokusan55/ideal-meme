# モデルの、ある部品は「2」ではなく「or」という単語そのものに反応しているらしい

**Epistemic status**: 個人の週末実験。当初「二者択一に反応する部品」という仮説を立てたが、追加実験の結果、より単純な説明（"or"という単語への反応）に、書き直すことになった。1つのモデル、複数の特徴、多数の文章で確認したが、まだ体系的な検証ではない。

## きっかけ（仮説、当初の形）

*※これは、最初の出発点であり、後で大きく修正されることになる。*

日常の中で、「境界」や「二者択一」という概念に、数の"2"が、なんとなく特別な形で関わっている気がする、という直感を持っていた。もしこれが本当なら、言語モデルの内部にも、「2」という数字そのものよりも、「二者択一」という"構造"に反応する、専用の部品(特徴)があるのではないか、と考えた。

## 方法

[Neuronpedia](https://www.neuronpedia.org/) のCircuit Tracer（Gemma-2-2B、GemmaScope Transcoder）を使い、"choice"に関連する特徴を検索した。上位候補として、次の2つを選んだ。

- **Feature 498**（layer 10、gemmascope-transcoder-16k）── ラベル: "choices and decisions"
- **Feature 4482**（layer 2、gemmascope-transcoder-16k）── ラベル: "choices and options"

まず、次の3つの文章で、活性化の強さを比較した：

1. `You can choose coffee or tea.` （二者択一）
2. `I have two apples on the table.` （ただの数字の2、選択ではない）
3. `You can choose from many flavors.` （選択だが、"or"を使わない、多数からの選択）

この結果を受けて、**「2つか3つか」と「'or'の有無」を、切り分けるため**、追加の文章を、Feature 498に対して試した：

4. `Tea or coffee.` （二者択一、順番を逆に）
5. `Coffee, tea, or juice.` （三者択一）
6. `Apple, orange, or banana.` （三者択一、別の単語グループ）
7. `You can choose apple or orange.` （二者択一、果物）
8. `You can choose apple, orange.` （"or"を含まない、単なる列挙）

## 結果

**Feature 498（"choices and decisions"）**

| 文章 | 構造 | 活性化 |
|---|---|---|
| coffee or tea | 二者択一、"or"あり | 6.44 |
| tea or coffee | 二者択一、"or"あり（逆順） | 6.69 |
| apple or orange | 二者択一、"or"あり（果物） | 5.81 |
| coffee, tea, or juice | 三者択一、"or"あり | 7.06 |
| apple, orange, or banana | 三者択一、"or"あり（果物） | 5.06 |
| apple, orange | "or"なし、単なる列挙 | 2.05 |
| two apples | 選択でも列挙でもない | 0.0000 |
| many flavors | 選択だが"or"を使わない | 0.0000 |

**Feature 4482（"choices and options"）**

| 文章 | 活性化 |
|---|---|
| coffee or tea | 3.34 |
| two apples | 0.0000 |
| many flavors | 9.13（"from"に最も強く反応） |

## 解釈

当初の仮説（「2」という数字自体が特別）は、支持されなかった。「2」という数字は、どの文章でも、活性化を引き起こしていない（`two apples` は常に0.0000）。

**さらに、「二者択一」という"意味"への反応、という解釈も、追加実験で崩れた。** 決定的だったのは、次の対比である。

$$\boxed{\text{apple, orange（"or"なし、2.05） vs apple or orange（"or"あり、5.81）}}$$

同じ2つの単語（apple, orange）でも、"or"が入るだけで、活性化が、約2.8倍に跳ね上がる。さらに、三者択一（coffee, tea, or juice = 7.06）は、二者択一（coffee or tea = 6.44）より、むしろ高い活性化を示した。「選択肢が2つか3つか」は、活性化の強さに、明確な影響を与えていない。

> Feature 498は、「2つから選ぶ」という意味ではなく、**"or"という単語、あるいは"orで終わる列挙"という文の構造そのもの**に反応している可能性が高い。

**副次的な観察**：飲み物の単語（coffee, tea, juice）を使った文章は、果物の単語（apple, orange, banana）を使った、対応する文章より、一貫して、やや高い活性化を示した（例：coffee or tea = 6.44 vs apple or orange = 5.81）。これは、"or"の有無ほど大きな効果ではないが、語彙自体にも、何らかの影響がある可能性を示す。

Feature 4482は、Feature 498と異なり、`many flavors`（"or"を使わない、多数からの選択）に、最も強く反応しており、こちらは、"or"という単語そのものより、選択という意味に近い何かを、捉えている可能性がある。この2つの特徴の違いは、まだ十分に説明できていない。

## 限界

- 1つのモデル、少数の特徴だけで確認した、小さなサンプル
- 「'or'への反応」という新しい解釈も、"or"を含む、他の文脈（例："I like apples or oranges, whichever."のような、選択を意味しない"or"）で、試せていない
- Feature 4482が、なぜ"many flavors"に強く反応するのか、まだ説明できていない

## 次にやること

- "or"を含むが、選択を意味しない文章（例："I like apples or oranges, whichever." のような、選択ではない"or"）で、Feature 498を試し、"or"という単語そのものへの反応か、文脈込みの反応かを切り分ける
- Feature 4482が、なぜ"many flavors"に特化して強く反応するのか、他の"選択肢の多さ"を示す表現で確認する
- 他のモデル（Gemma-2-9Bなど）でも、同じパターン（"or"の有無が支配的）が再現するか確認する
