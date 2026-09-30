# モデルの、ある部品は「2」ではなく「or」という単語そのものに反応しているらしい

**Epistemic status**: 個人の週末実験。当初「二者択一に反応する部品」という仮説を立てたが、追加実験の結果、より単純な説明（"or"という単語への反応）に、書き直すことになった。1つのモデル、複数の特徴、多数の文章で確認したが、まだ体系的な検証ではない。

*この記事は、実験の設計・実行・データの読み取りを、投稿者本人が行い、文章の構成や整理に、AI（Claude）の助けを借りて作成しました。*

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

さらに、**「'or'という単語の見た目だけに反応しているのか、それとも'選択を提示する'という文脈に反応しているのか」**を切り分けるため、選択を意味しない"or"を含む文章も試した：

9. `It's about five or six o'clock.` （「だいたい〜くらい」という意味の"or"、選択ではない）
10. `Finish your homework, or else.` （警告・脅しの"or"、選択ではない）

さらに、**"or"の前に、選択を強調する"either"を加えると、活性化が変わるか**を確認した：

11. `You can choose either coffee or tea.` （二者択一、"either...or"の強調構文）

## 結果

**Feature 498（"choices and decisions"、layer 10、gemma-2-2b/10-gemmascope-transcoder-16k）**

| 文章 | 構造 | 活性化 |
|---|---|---|
| You can choose coffee or tea. | 二者択一、"or"あり | 6.44 |
| You can choose tea or coffee. | 二者択一、"or"あり（逆順） | 6.69 |
| You can choose coffee, tea, or juice. | 三者択一、"or"あり | 7.38 |
| You can choose apple or orange. | 二者択一、"or"あり（果物） | 5.81 |
| You can choose apple, orange, or banana. | 三者択一、"or"あり（果物） | 6.19 |
| You can choose apple, orange. | "or"なし、単なる列挙 | 2.05 |
| I have two apples on the table. | 選択でも列挙でもない | 0.0000 |
| You can choose from many flavors. | 選択だが"or"を使わない | 0.0000 |
| It's about five or six o'clock. | "or"あり、しかし選択ではない（あいまいさの表現） | 0.0000 |
| Finish your homework, or else. | "or"あり、しかし選択ではない（警告・脅し） | 0.0000 |
| **You can choose either coffee or tea.** | **二者択一、"either...or"の強調構文** | **8.25** |
| **You can choose either coffee or tea.** | **二者択一、"either...or"の強調構文** | **8.25** |

*これらの数値は、すべて同一セッション内で、モデル・層・IDを画面のパンくずリストで確認しながら記録した、確定版である（2回目の検証）。*

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

同じ2つの単語（apple, orange）でも、"or"が入るだけで、活性化が、約2.8倍に跳ね上がる。二者択一（coffee or tea = 6.44、apple or orange = 5.81）と三者択一（coffee, tea, or juice = 7.38、apple, orange, or banana = 6.19）を比べると、いずれも同程度の範囲に収まっており、「選択肢が2つか3つか」は、活性化の強さに、明確な影響を与えていない。

> Feature 498は、「2つから選ぶ」という意味ではなく、**"or"という単語、あるいは"orで終わる列挙"という文の構造そのもの**に反応している可能性が高い。

**しかし、この解釈も、さらなる実験で、より正確な形に絞り込まれた。** "or"を含んでいても、選択を提示しない文脈（`five or six o'clock` のような、あいまいさの表現、`homework, or else` のような警告）では、活性化は完全にゼロ（0.0000）だった。

$$\boxed{\text{Feature 498は、"or"という単語の見た目に反応しているのではなく、"orを使って、複数の選択肢を並べて提示する"という、特定の構造に反応している}}$$

英語の"or"には、複数の異なる役割がある（選択肢の提示、あいまいさの表現、警告など）。Feature 498は、この中で、選択肢を提示する役割の"or"にだけ反応しており、単なる字面のパターンマッチングではないことが、この追加実験で確認できた。

**副次的な観察**：飲み物の単語（coffee, tea, juice）を使った文章は、果物の単語（apple, orange, banana）を使った、対応する文章より、一貫して、やや高い活性化を示した（例：coffee or tea = 6.44 vs apple or orange = 5.81）。これは、"or"の有無ほど大きな効果ではないが、語彙自体にも、何らかの影響がある可能性を示す。

Feature 4482は、Feature 498と異なり、`many flavors`（"or"を使わない、多数からの選択）に、最も強く反応しており、こちらは、"or"という単語そのものより、選択という意味に近い何かを、捉えている可能性がある。この2つの特徴の違いは、まだ十分に説明できていない。

**追加確認："either...or"で、活性化はさらに上がった。** `You can choose coffee or tea.`（6.44）に対して、`You can choose either coffee or tea.`は**8.25**となり、約1.3倍、上昇した。"either"自体にも、薄く反応が見られた。

$$\boxed{\text{Feature 498は、"or"という単語の締めくくりだけでなく、"選択肢を提示する"という意味・強調の度合いにも反応している}}$$

"either...or"は、英語で選択肢を強調する、決まった言い回しである。この構文で活性化が上がったことは、この特徴が単なる字面のパターンではなく、「今から選択肢を並べる」という予告・強調の強さに、ある程度反応している可能性を示す。

## 限界

- 1つのモデル、少数の特徴だけで確認した、小さなサンプル
- Feature 4482が、なぜ"many flavors"に強く反応するのか、まだ説明できていない
- 選択肢の提示という"構造"を、モデルがどう定義しているか（例えば、"or"以外の接続詞、"either...or"のような構文）は、まだ確認していない
- この特徴（498番）自体の、モデル全体における最大活性化は58〜60台であり、今回観測した値（2〜7台）は、その1割強にとどまる。弱い反応の範囲内での観察である
- **測定上の注意点**：Neuronpediaでの計測時は、ブラウザの自動翻訳を無効にし、URLのパンくずリスト（モデル名・層番号・特徴ID）を毎回確認する必要がある。同じ特徴ID（例：4482）でも層が異なれば全く別の特徴になり、また画面の日本語自動翻訳は単語ラベルの読み取りを誤らせる。本記事の数値は、これらを統制した上での再測定値である

## 次にやること

- Feature 4482が、なぜ"many flavors"に特化して強く反応するのか、他の"選択肢の多さ"を示す表現で確認する
- 他のモデルでの再現を試みたが、Gemma-2-9BはNeuronpedia上で、まだ推論機能（自分で書いた文章をその場で試す機能）が有効になっておらず、今回は検証できなかった。推論が有効な、別のモデルで、改めて試す
