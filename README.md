# infra-shared

複数プロジェクトで共有する AWS インフラリソースを Terraform で管理するリポジトリ。

## 管理リソース

- **Route53 ホストゾーン** — カスタムドメインの DNS 管理

## ディレクトリ構成

```
infra-shared/
├── backend.tf              # S3 backend 定義
├── providers.tf            # AWS provider 設定
├── main.tf                 # リソース定義
├── variables.tf            # 変数定義
├── locals.tf               # ローカル値
├── outputs.tf              # 出力値
└── env/
    └── prod/
        ├── inputs.tfvars.example     # 変数の例
        └── s3.tfbackend.example      # backend 設定の例
```

## セットアップ

### 1. 設定ファイルの準備

`.example` ファイルをコピーして実際の値を記入する。

```bash
cp env/prod/s3.tfbackend.example env/prod/s3.tfbackend
cp env/prod/inputs.tfvars.example env/prod/inputs.tfvars
```

### 2. Terraform 初期化

```bash
terraform init -backend-config=env/prod/s3.tfbackend
```

### 3. 既存リソースの Import（初回のみ）

```bash
terraform import -var-file=env/prod/inputs.tfvars aws_route53_zone.main <ZONE_ID>
```

### 4. 適用

```bash
terraform plan -var-file=env/prod/inputs.tfvars
terraform apply -var-file=env/prod/inputs.tfvars
```

## Outputs

| 名前 | 説明 |
|------|------|
| `route53_zone_id` | Route53 ホストゾーン ID |
| `route53_zone_name_servers` | NS レコード |
| `domain_name` | ドメイン名 |

## 他プロジェクトからの参照

`terraform_remote_state` で outputs を参照できる。

```hcl
data "terraform_remote_state" "shared" {
  backend = "s3"
  config = {
    bucket  = "<tfstate バケット名>"
    key     = "shared/terraform.tfstate"
    region  = "ap-northeast-1"
    profile = "<AWS プロファイル名>"
  }
}

# 使用例
resource "aws_route53_record" "example" {
  zone_id = data.terraform_remote_state.shared.outputs.route53_zone_id
  name    = "app.${data.terraform_remote_state.shared.outputs.domain_name}"
  type    = "A"
  # ...
}
```
