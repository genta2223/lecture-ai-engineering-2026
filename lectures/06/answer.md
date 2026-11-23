# 第6回 解答

## 演習

<!-- [tool call] の行は、表示された行をそのまま貼り付けてください。終わらなかった STEP は、空欄のままでかまいません。 -->

- STEP 1 のツールの名前：`glossary_search_glossary`
- STEP 2 の [tool call]：`[tool call]   glossary_search_glossary({'query': '宿題の3レベル'})`
- STEP 3 の [tool call]：`[tool call]   fetch_fetch({'url': 'https://modelcontextprotocol.io/docs/getting-started/intro'})`
- STEP 4 の [tool call]：`[tool call]   aws-docs_aws___search_documentation({'search_phrase': 'Amazon Bedrock Knowledge Bases', 'topics': ['general']})`
- 気づき・疑問：
  - `app/mcp_servers.json` に設定を追加するだけで、ローカルの Python プロセス（stdio）や `uvx` 経由のツール、外部のリモート HTTP サーバー（AWS ドキュメントなど）をエージェント側のコードを一切変えずに統合できる点に MCP の汎用性を実感した。
  - 一方で、接続サーバーを増やすとツール数とその説明文（スキーマ）がそのままプロンプトの入力トークンとして消費されるため（1個で694トークン → 7個で約10,000トークン超に増加）、必要なツールだけを厳選して接続する設計が重要だと分かった。

## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

- 取り組んだレベル：基礎
- やったこと：
  - `app/mcp_server.py` に `server.tool()(tools.get_lecture_summary)` を1行追加し、講義各回の要点を取得する関数を MCP サーバーのツールとして公開した。
  - `uv run python -m app.mcp_agent --list-tools` および `--trace "第2回では何を学びますか？"` を実行し、追加したツールがエージェントから自動認識・呼び出しされることと、`uv run pytest -q` で全21件のテストがパスすることを確認した。
- 結果：
  - 公開ツール一覧に `[glossary_get_lecture_summary]` が追加され、`--trace` 実行時に `[tool call] glossary_get_lecture_summary({'lecture': 2})` が呼び出されて第2回の要点（ファインチューニングとLoRA）が正確に返答された。
  - 自動テスト（`pytest`）も `21 passed` で正常に通過した。
- 気づき・疑問：
  - 既存の Python 関数に型ヒントと docstring が適切に書かれていれば、`server.tool()(...)` で登録するだけで MCP サーバーのツール仕様（名前・説明・引数スキーマ）としてそのまま公開でき、エージェント側の修正なしに新機能を追加できるのが非常に開発効率が高いと感じた。