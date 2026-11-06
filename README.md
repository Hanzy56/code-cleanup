# refactor-structure / audit-test-quality

コード構造の改善とテスト品質の整理を、別々に、利用者の承認に基づいて行うための2つのSkill。どちらも既定では監査と提案で停止する。

```
skills/
  refactor-structure/
    SKILL.md
    references/
      decision-log-template.md
      examples.md
  audit-test-quality/
    SKILL.md
    references/
      test-audit-template.md
      examples.md
LICENSE
README.md
```

## 使い分け

| | refactor-structure | audit-test-quality |
| --- | --- | --- |
| 目的 | 外部から見た振る舞いを維持しながら、変更の影響範囲を小さくする | 回帰検出能力を維持・向上させながら、テストの保守コスト、誤検知、重複、実行負荷を減らす |
| 主な問い | これらのコードは同じ理由で一緒に変更されるべきか | このテストはどんな回帰を検出できるか。削除後に残るテストで足りるか |
| 変更してよいもの | 承認された本体コード。テストは承認された機械的な追従だけ | 承認されたテストコードとテスト専用の補助コード |
| 変更しないもの | テストの検証内容。テストの削除・統合は行わない | プロダクションコード |
| 目的にしないこと | 行数、ファイル数、共通化率、特殊処理数の削減 | テスト数の削減。削除件数や削減率の目標 |

各Skillは単独で利用でき、必須の安全規則はそれぞれの `SKILL.md` に書いてある。

## 流れ

1. **監査**：対象、規約、未コミット変更、検証手段、権限と予算を確認し、合意した範囲を1巡して提案を出す。対象リポジトリの追跡ファイルは変更しない。
2. **承認**：利用者が候補IDと変更内容を指定する。指定がなければ実装に進まない。
3. **実装**：承認された変更単位ごとに行う。前提や影響範囲が変わったら止めて再承認を求める。
4. **検証**：変更前後を比較し、差分を確認する。結果と未実行の項目、残るリスクを報告して終了する。

「改善余地がなくなるまで続ける」という終了条件はない。別の問題を見つけても記録するだけで、範囲は広げない。

## 呼び出し例

```text
/refactor-structure src/billing
/audit-test-quality tests/billing
```

呼び出しの形式は実行環境によって異なる場合がある。

## 承認の例

Skillは提案の末尾で、候補ごとの変更内容と検証コマンドを示して停止する。

```text
R-01とR-03を提案どおり実施してください。R-02は見送ります。commitはしないでください。
```

```text
T-02を書き換え、T-05とT-06を統合してください。T-09の削除は見送ります。
```

次の返答は、対象を特定できないので承認として扱わない。Skillは実施する候補を尋ね返す。

```text
いいね
続けて
```

## 構造改善とテスト品質整理を同じ変更に混ぜない理由

構造変更の検証は、「テストを変えずに、変更前後で同じ結果になる」ことに支えられている。テストを同時に変えると、次の区別ができなくなる。

- テストが通ったのは、振る舞いが維持されたからか。テストが弱くなったからか。
- テストが失敗したのは、振る舞いが変わったからか。テストが内部構造に依存していたからか。

分離すれば、レビューでも検証でも、差分の意味を1つの問いとして確認できる。

## テストを先に整える場合

テストが内部構造に依存していて、振る舞いを変えない構造変更でも失敗する場合は、順序を分ける。

1. **テストの変更**（`audit-test-quality`）：本体を固定したまま、振る舞いを検証する形に書き換える。本体が同じままで通ることが、書き換えの確認になる。
2. **構造の変更**（`refactor-structure`）：テストを変更せずに、構造を変える。

既存テストが十分に信頼できるなら、この準備は必要ない。必ずこの順序で行う決まりではなく、必要なときの選択肢である。

## Skillで保証できないこと

**Skillの文章やfrontmatterは、実行環境の権限制御やサンドボックスの代わりにはならない。**

- Skillの規則はモデルへの指示であり、技術的な強制ではない。指示どおりに動作する保証はない。
- ファイル変更、コマンド実行、ネットワーク接続の制限は、実行環境の権限設定やサンドボックスで行う必要がある。
- `disable-model-invocation: true` の扱いは実行環境に依存する。
- `git status` と差分の確認は、利用者自身でも行うことを勧める。
- 提案の根拠には誤りがあり得る。特にテストの削除は、利用者が根拠を確認してから承認する。

このリポジトリには、ツール権限を広げる設定、インストーラー、外部送信処理、自動実行用のhookは含まれていない。

## 参照元と独自に変更した方針

参照元：[caldoai/skills の skills/untangle-codebase/SKILL.md](https://github.com/caldoai/skills/blob/main/skills/untangle-codebase/SKILL.md)

元Skillは参照資料として使った。文章をそのまま移したものではなく、考え方を基に日本語で書き起こした。例はすべて新たに作った架空のものである。

### 継承した考え方

- 「これらのコードは、同じ理由で一緒に変更されるべきか」を判断軸にする。
- 問題を、隠れた依存、不要な依存、存在意義の薄い構造に分ける。
- 個別の症状だけでなく、その背後に共通する原因を探す。
- ファイル・行番号・具体的な根拠を伴うDecision Logを残す。
- テストは、意図した振る舞いの破壊を検出するためのものと考える。

### 変更した考え方

| 元Skill | このSkill |
| --- | --- |
| コードベース全体を一度に改修する | 既定は監査と提案。承認された範囲だけを実装する |
| 修正すべき点がなくなるまで繰り返す | 合意した範囲を1巡して終了する |
| 残った特殊処理の少なさを主要な指標にする | 特殊処理の数は指標にしない。要件や外部制約を確認し、不明なら維持する |
| 実装を繰り返すテストや失敗し得ないテストを削除する | 実装をなぞるテストは独立した期待値への書き換えを先に検討する。削除には代替と根拠を求める |
| 構造改善の一部としてテストとコメントも整理できる | Skillを分離する。コメントの一括整理は行わない |
| リファクタリングで直ったバグは許容する | バグ修正を混ぜず、別の変更として提案する |

追加した方針：確認済みの事実・推論・未確認事項の区別、残存テスト集合の確認、隔離コピーでのミューテーション検証、実行とファイル変更の制限。

## 確認できた参照元とライセンス

2026-11-06 に GitHub から取得して確認した。

| 項目 | 結果 |
| --- | --- |
| 参照元の取得 | 成功（`main` ブランチ、8,881 bytes） |
| Blob SHA | `102ff57803e504876f6eb9b412e4eb0f6c576350` |
| 確認済みの版との差異 | なし。取得内容からローカルで計算した値、GitHub APIの値、依頼時に示された値がすべて一致した |
| 参照元の最新コミット（このファイルに関するもの） | `442c29918339cc8dbf09ae27aaf17b05745465a3` |
| 取得時の `main` のコミット | `f3cf47774975a7d9fca45730babf4ff7ad765ada` |
| ライセンス | MIT License（`LICENSE` ファイル、GitHub API、参照元のREADMEで確認） |
| `LICENSE` のBlob SHA | `0f255b7e8b17e994384d1b981fc1a2d4a27d7bde` |
| 著作権表示 | Copyright (c) 2026 Maze |

**確認の制限**

- `SKILL.md` にはファイル個別のライセンス表記がなかった。リポジトリの `LICENSE` がこのファイルに適用されると解釈した（推論）。
- リポジトリのファイル一覧に `NOTICE` はなかった。
- 各Skillを本READMEと切り離して配布する場合は、`LICENSE` と下記の表示も同梱する。
- この記載は法的助言ではない。商用配布などで判断が必要な場合は、専門家に確認する。

## ライセンス

このリポジトリの2つのSkillとドキュメントは [MIT License](LICENSE)（Copyright (c) 2026 Hanzy56）で提供する。

### 参照元の帰属表示

これらのSkillは、以下のソフトウェアを参考に作成した。

```text
MIT License

Copyright (c) 2026 Maze

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
