# 許可されているリソースの種類 (Allowed Resource Types Policy)

このカスタム Azure Policy は、環境内でデプロイを許可するリソースの種類をホワイトリスト形式で指定します。指定外のリソースは `effect` に従って `Deny` または `Audit` されます。

## ポリシー定義（概要）

- displayName: 許可されているリソースの種類
- policyType: Custom
- mode: All

定義ファイル: `policyDefinition.json`

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

## デプロイ方法（az rest）

ポリシー定義が ARM 形式（`properties` 等を含む完全な定義）である場合、`az policy definition create` では登録できないことがあります。そのため、本リポジトリのデプロイ方法は `az rest` を使った直接 PUT の手順のみを推奨します。サブスクリプション ID は匿名化したプレースホルダに置き換えています。

```bash
policyName="allowed-resource-types"
policyFile="policyDefinition.json"
apiVersion="2023-04-01"
subId=$(az account show --query id -o tsv)

body=$(jq -c . "$policyFile")

uri="https://management.azure.com/subscriptions/<SUBSCRIPTION_ID>/providers/Microsoft.Authorization/policyDefinitions/${policyName}?api-version=${apiVersion}"
az rest --method put --uri "$uri" --body "$body"
```

上記はユーザー環境で実行する際に `<SUBSCRIPTION_ID>` を実際のサブスクリプション ID に置き換えてください。`az account show --query id -o tsv` を使うことでスクリプト内で ID を取得して利用することもできます。

## 検証・テスト方法

- 許可外のリソースをテスト環境にデプロイして `Deny` / `Audit` が発生することを確認してください。
- Azure CLI での確認コマンド例:
  - `az policy state summarize --subscription <SUBSCRIPTION_ID>`
  - `az policy assignment list --scope /subscriptions/<SUBSCRIPTION_ID>`
- Azure Portal の Policy 評価結果からも確認できます。
