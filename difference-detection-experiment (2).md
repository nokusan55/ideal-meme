# モデルは「内容の差異」を、言葉に頼らず検知できるか ── 2つの特徴を調べた、現時点での記録

**Epistemic status**: 個人の週末実験、続編。「モデルは、状況が"驚くべきこと"かどうかを、言葉を使わず判断できるか」という、大きな問いの、ごく一部を、2つの特徴に絞って調べた。現時点では否定的な結果が多く、調査は継続中。

*この記事は、実験の設計・実行・データの読み取りを、投稿者本人が行い、文章の構成や整理に、AI（Claude）の助けを借りて作成しました。*

## きっかけ

人は、予測していたことと、実際に起きたことの"ズレ"を感じ取る。これが、意識や注意の、根源的な働きの一つではないか、という考えがある。もし、これが本当なら、言語モデルの内部にも、「文の内容が、常識や期待と矛盾しているかどうか」を、言葉を使わずに検知する部品があるのではないか、と考えた。

## 実験1：Feature 3500（"surprise"）

Neuronpedia（Gemma-2-2B、12-gemmascope-transcoder-16k、ID 3500、ラベル："words related to surprise, bewilderment, and astonishment"）を使い、"驚き"に関する語彙への反応を調べた。

### 結果

| 文章 | 条件 | 活性化 |
|---|---|---|
| I was surprised that the sun rose in the west this morning. | 驚きの語あり、矛盾した内容 | 25.00 |
| I was surprised that the sun rose in the east this morning. | 驚きの語あり、矛盾なし内容 | 25.00 |
| I was astonished / shocked that... | 同義語 | 25.13 / 26.38 |
| I was not surprised that... | 否定形 | 9.38 |
| It was a surprising morning. | 形容詞形 | 13.13 |
| Wow, the sun rose in the west this morning! | 驚きの語なし | 0.0000 |
| Everyone knows the sun rises in the east every day without exception. Today, it rose in the west.（長い文脈で矛盾を説明、驚きの語なし） | 驚きの語なし | 0.0000 |
| I was surprised that [flower bloomed overnight / metal expanded in the cold / dog could swim across the river / she finished the marathon in two hours / old computer still turned on] | 5つの異なる領域 | すべて25.00 |
| This is impossible. / Something is very wrong.（別の異常性語彙、驚きの語なし） | 驚きの語なし | 両方0.0000 |

### 結論（Feature 3500について）

$$\boxed{\text{Feature 3500は、文の内容が本当に驚くべきことかどうかを一切判断しない。"surprise"系の語彙が文中にあるかどうかだけで、ほぼ一定の値（約25）を返す、語彙検出器である}}$$

内容の領域（生物、物理、動物、人間、機械）を変えても反応は変わらず、東西を入れ替えても、長い文脈で矛盾を説明しても、驚きの語がなければ反応はゼロだった。同義語には同程度反応し、否定や形容詞化には部分的にしか対応しない。

## 実験2：Feature 1023（"words and phrases with strong emotional connotations"）

Circuit Tracerで `The sun rose in the west/east this morning.` を比較した際、west/east双方からほぼ同じ寄与（+0.55）を受けているにもかかわらず、個別ページでの活性化はwest=4.06、east=2.80と差があるように見えた特徴（Gemma-2-2B、6-gemmascope-transcoder-16k、ID 1023）を、追加で調べた。

### 結果

| 文章 | 活性化 |
|---|---|
| The sun rose in the west this morning. | 4.06 |
| The sun rose in the east this morning. | 2.80 |
| The moon appeared in the west this morning. | 1.14 |
| The moon appeared in the east this morning. | 1.70 |

### 結論（Feature 1023について）

$$\boxed{\text{sunの文ではwest>eastだったが、moonの文ではeast>westと逆転した。方角(west/east)そのものに、一貫して反応しているわけではなく、単語の組み合わせによる、説明のつかない変動である可能性が高い}}$$

これは、Feature 1023が「方角を検知する部品」であることを支持しない。ただし、このノード自体の活性化密度は4.6%と比較的高く、"nail"や"pudding"など無関係な文脈でも強く発火しており、そもそも方角とは無関係な、別の役割を持つ可能性もある。

### 追加調査：このノードの正体は「慣用句・比喩的なイディオムの検出器」だった

上位の活性化例を、約30件、まとめて確認したところ、はっきりしたパターンが見つかった。"fought tooth and nail"（必死に戦った）、"the proof is in the pudding"（論より証拠）、"writing on the wall"（不吉な前兆）、"pushes the envelope"（限界を押し広げる）、"ignorance is bliss"（知らぬが仏）、"meeting of the minds"（合意）、"fanning the flames"（火に油を注ぐ）、"like a lead balloon"（まったく受けない）、"going for broke"（一か八かやる）、"give up the ghost"（あきらめる）、"ticking time bomb"（危険な状態）、"the sky was falling"（大騒ぎする）、"separate the wheat from the chaff"（良いものと悪いものを選別する）、"safety in numbers"（数は力なり）、"melting pot"（人種のるつぼ）など、**上位例のほとんどすべてが、文字通りの意味とは異なる、決まった比喩的な言い回し（イディオム）だった**。

$$\boxed{\text{Feature 1023は、方角とはほぼ無関係であり、"英語の慣用句・比喩的なイディオム"を検出する特徴である}}$$

これにより、前述のwest/eastの不安定な反応（sunではwest>east、moonではeast>west）の正体も説明がつく。"appeared in the west"や"rose in the west"という言い回しが、たまたま決まり文句のリズムに近かったために、わずかに反応していたにすぎず、方角そのものを検知していたわけではなかった。

最初、west/eastの違いを調べるために、たまたま見つけたノードが、実は、まったく別の役割（慣用句検出器）を持っていた、という、解釈可能性研究らしい教訓が得られた。

## 実験3：Feature 9758（"arithmetic"）

数式の正誤判断に反応する特徴を探すため、Neuronpedia（Gemma-2-2B、8-gemmascope-transcoder-16k、ID 9758、ラベル："mathematical terms, equations, operations, and code" / "mathematical operations" / "arithmetic"）を調べた。

| 文章 | 活性化 |
|---|---|
| 2 + 2 = 4（正しい式） | 19.25 |
| 2 + 2 = 5（誤った式） | 19.25 |

正誤に関わらず、まったく同じ値だった。このノードは、計算の正しさではなく、「数式という"形"」そのものに反応している。

## 実験4：Feature 204（"calculations"）

同じく数式関連のラベルを持つ、別の特徴（Gemma-2-2B、14-gemmascope-transcoder-16k、ID 204、ラベル："technical measurements..." / "calculations"）も調べた。

| 文章 | 活性化 |
|---|---|
| 2 + 2 = 4（正しい式） | 0.0000 |
| 2 + 2 = 5（誤った式） | 0.0000 |

こちらは、正誤どころか、式があっても、まったく反応しなかった。

## Circuit Tracerによる、全体比較の試み

個々のノードではなく、`2 + 2 = 4` と `2 + 2 = 5` という、文章全体のCircuit Tracerグラフを、直接比較することも試みた。しかし、グラフの見た目だけでは、違いがあるかどうかを判断できず、west/eastの実験のときと同じ限界に突き当たった。これを確かめるには、個々のノードを1つずつクリックして、ラベルを比較する、地道な作業が必要になる。

## 実験3：算術の正誤を扱う2つの特徴

「驚き」「慣用句」が、どちらも、内容の真偽とは無関係だったことを受け、より直接的に、"論理的な正誤"を扱っていそうな特徴を探した。

**Feature 9758**（"arithmetic"、layer 8、8-gemmascope-transcoder-16k）で、`2 + 2 = 4` と `2 + 2 = 5` を比較したところ、**両方とも19.25と、まったく同じ値**だった。

**Feature 204**（"calculations"、layer 14、14-gemmascope-transcoder-16k）でも、同じ2文を試したところ、**両方とも0.0000**だった。

$$\boxed{\text{2つの特徴は、どちらも、計算の正誤を判断していない。Feature 9758は"数式がある"こと自体に反応し、Feature 204はこの条件では反応しないという、違うパターンを示したが、"真偽の判断"という点では、両方とも同じ結論だった}}$$

**Circuit Tracerでの、グラフ全体の比較も試みた**（`2+2=4` と `2+2=5` の、それぞれのグラフ）。しかし、グラフの見た目だけでは、"違いがあるかどうか"を判断できず、west/eastのときと同じ限界にぶつかった。個々のノードを1つずつ確認する必要があり、今回は、そこまでは至らなかった。

## 補足実験：層の深さは「賢さ」の違いなのか（暫定）

同じ"choice"という概念が、層の深さで、どう違って見えるかを、簡単に確認した。詳細と最新の測定値は、別記事（binary-choice-feature-experiment）の「追加実験」に記録している。

**Feature 4482**（"choices and options"、layer 2、浅い層）では、`You can choose coffee or tea.` に対し "coffee" が1.01、"or" が3.34だった。

**Feature 5328**（"phrases related to preferred methods / choice"、layer 19、深い層）では、同じ文章で**0.0000**だった。一方、`Coffee is my drink of choice.` では "of" が69.00、`The weapon of` でも "of" が68.00と強く反応したが、`The method of` では反応がなかった。「"of choice"という構文に特化」という当初の説明は単純すぎ、現時点では「次に"choice"が来そうな語の組み合わせに反応するのではないか」という、未確認の仮説にとどまる（なぜ`method of`が0なのかは未解明）。

したがって、「層が深いほど賢く反応する」とも「層ごとに専門家が入れ替わる」とも、**まだ言えない**。調べたのは少数の層・特徴だけであり、どちらも仮説として扱う。

## 方法論上の、重要な気づき

ここまでで、**5つの、独立した特徴**（"or"の構造検出、"surprise"の語彙検出、慣用句検出、そして2つの算術特徴）を調べ、**すべてが、同じ結論**（内容の真偽や、状況の妥当性ではなく、言葉・形・語彙に反応している）に達した。

$$\boxed{\text{個々の特徴(ニューロン)を1つずつ探す、というやり方では、少なくとも5つの独立した候補において、"内容の真偽を判断する"機能は見つからなかった}}$$

これは、「そのような機能が存在しない」という証明ではなく、**「この探し方では見つからない」という、方法論上の限界**を示していると考えられる。もし、何らかの形で"真偽の判断"に相当するものがモデル内に存在するなら、それは単一のノードではなく、複数のノードが協調して働く"計算の過程"として存在している可能性が高い。これは、今回使った、個々の特徴を探索する手法の、射程を超えている。

## 全体のまとめ（現時点）

5つの独立した特徴（surprise語彙、慣用句検出、数式検出×2、west/east）を調べた限りでは、「モデルが、言葉や形に頼らず、内容の差異・矛盾・正誤そのものを検知している」ことを示す証拠は、見つからなかった。見つかったのは、特定の語彙や、数式という"形"にのみ反応する検出器ばかりで、いずれも、内容の真偽・正誤には無関係だった。

$$\boxed{\text{5つの独立した候補すべてで、同じ結論（言葉・形への反応であり、内容の真偽への反応ではない）に達した}}$$

これは、「差異検知の部品がモデルに存在しない」ことの証明ではない。むしろ、**「個々の特徴（ニューロン）を1つずつ探す」という、今回の方法そのものが、限界に来ている**可能性を示している。もし、モデルの中に「内容の真偽を判断する」ような働きがあるとすれば、それは単一のノードではなく、**複数のノードが協調して働く、"計算の過程"として存在している**のかもしれない。これは、1つのノードをいくら探しても、原理的に見つからない可能性がある。

## 今後、検証したい方向性（未着手、記録のみ）

- **（最も重要な、次の課題）** 単一の特徴ではなく、複数ノードの協調（回路）を分析する、より高度な手法を学ぶ。これは、現在の自分の技術レベルを超える可能性が高く、大きな、次の挑戦として位置づける
- 北・南、左・右など、他の方角・方向の組でも、同様に「実は別の役割だった」という結果になるか確認する
- 対称・非対称という、より抽象的な対の概念での反応
- Feature 1023の「慣用句検出器」という仮説自体を、イディオムでない、ごく普通の文章と比較して、さらに検証する

## 限界

- 2つの特徴だけを調べた、ごく小さな調査
- グラフ上のノード間の寄与度（INPUT FEATURES）の数値と、個別ページでのテスト結果が、必ずしも一致しない場合があり、両者の関係はまだ十分に理解できていない
- 「差異検知」という、検証したい概念自体が大きく、現時点ではその周辺を手探りしている段階である
