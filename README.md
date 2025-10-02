# 許可されているリソースの種類 (Allowed Resource Types Policy)

このカスタム Azure Policy は、環境内でデプロイを許可するリソースの種類をホワイトリスト形式で指定します。指定外のリソースは `effect` に従って `Deny` または `Audit` されます。

## ポリシー定義（概要）

- displayName: 許可されているリソースの種類
- policyType: Custom
- mode: All
- version: 2.0.0
- 定義ファイル: `policyDefinition.json`

このポリシーは `type` フィールドを評価し、`listOfAllowedResourceTypes` パラメータに含まれないリソースタイプを検出した場合に指定した `effect` を適用します。

## パラメータ

- `listOfAllowedResourceTypes` (Array)
  - 説明: 許可するリソースタイプ（ホワイトリスト）。例: `Microsoft.Compute/virtualMachines`
- `effect` (String)
  - 説明: ポリシーの効果。`Audit`, `Deny`, `Disabled` のいずれか。デフォルトは `Deny`。

### params.json の例

```json
{
  "listOfAllowedResourceTypes": {
    "value": [
      "Microsoft.Compute/virtualMachines",
      "Microsoft.Storage/storageAccounts",
      "Microsoft.Network/virtualNetworks"
    ]
  },
  "effect": {
    "value": "Deny"
  }
}
```

## デプロイ方法（Azure CLI）

以下はポリシー定義を作成し、サブスクリプションに割り当てる最小の例です。`<SUBSCRIPTION_ID>` とファイルパスは環境に合わせて置き換えてください。

```bash
# ポリシー定義を作成
az policy definition create \
  --name allowed-resource-types \
  --display-name "許可されているリソースの種類" \
  --description "指定したリソース種類のみを許可する" \
  --rules policyDefinition.json \
  --params params.json \
  --mode All \
  --subscription <SUBSCRIPTION_ID>

# サブスクリプションに割り当て（例）
az policy assignment create \
  --name enforce-allowed-resources \
  --scope /subscriptions/<SUBSCRIPTION_ID> \
  --policy allowed-resource-types \
  --params params.json
```

注意: 上記コマンドを実行するには適切な権限（例: Policy Contributor）が必要です。

## 検証・テスト方法

- 許可外のリソースをテスト環境にデプロイして `Deny` / `Audit` が発生することを確認してください。
- Azure CLI での確認コマンド例:
  - `az policy state summarize --subscription <SUBSCRIPTION_ID>`
  - `az policy assignment list --scope /subscriptions/<SUBSCRIPTION_ID>`
- Azure Portal の Policy 評価結果からも確認できます。
