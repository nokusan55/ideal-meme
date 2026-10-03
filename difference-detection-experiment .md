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

## 全体のまとめ（現時点）

2つの特徴を調べた限りでは、「モデルが、言葉に頼らず、内容の差異・矛盾そのものを検知している」ことを示す証拠は、まだ見つかっていない。見つかったのは、特定の語彙（"surprise"系の単語）にのみ反応する検出器と、方角とは無関係な、まったく別の役割（慣用句・比喩表現の検出）を持つ特徴だけだった。

これは、「差異検知の部品が存在しない」ことの証明ではなく、「この2つの特徴には、それが見当たらなかった」という、限定的な結果である。特に2つ目の発見は、「探していたものとは違う、思いがけない役割を持つ特徴に出会う」という、解釈可能性研究によくある経験を、具体的に示す例になった。

## 今後、検証したい方向性（未着手、記録のみ）

以下は、まだ検証していない、思いついた段階のアイデアである。1つずつ、同じ手順（仮説→対照実験→反証の試み）で、検証していく必要がある。

- 北・南、左・右など、他の方角・方向の組でも、同様に「実は別の役割だった」という結果になるか確認する
- 対称・非対称という、より抽象的な対の概念での反応
- 数値の大小比較（以前、別の文脈で見た「1031 or 1034」のような、数の矛盾）に反応する特徴の探索
- Feature 1023の「慣用句検出器」という仮説自体を、イディオムでない、ごく普通の文章と比較して、さらに検証する

## 限界

- 2つの特徴だけを調べた、ごく小さな調査
- グラフ上のノード間の寄与度（INPUT FEATURES）の数値と、個別ページでのテスト結果が、必ずしも一致しない場合があり、両者の関係はまだ十分に理解できていない
- 「差異検知」という、検証したい概念自体が大きく、現時点ではその周辺を手探りしている段階である
