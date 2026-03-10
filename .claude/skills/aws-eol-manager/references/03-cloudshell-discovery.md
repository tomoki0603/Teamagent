# 03 — CloudShellリソース洗い出しガイド

## 概要

AWS CloudShellを使用して、EOL対象リソースを各アカウント・リージョンから洗い出す手順をまとめる。
CloudShellはAWSマネジメントコンソールからブラウザ上で直接利用でき、追加セットアップ不要。

---

## 前提条件

- AWSマネジメントコンソールへのログインアクセス
- 対象サービスへの `Describe` / `List` 権限（ReadOnlyAccess相当）
- 複数アカウントの場合: Organizations管理アカウントへのアクセスまたはスイッチロール用IAMロール

---

## サービス別コマンド

### RDS（MySQL / PostgreSQL）

```bash
# RDSインスタンス一覧（エンジン・バージョン付き）
aws rds describe-db-instances \
  --query "DBInstances[].{ID:DBInstanceIdentifier,Engine:Engine,Version:EngineVersion,Status:DBInstanceStatus,Class:DBInstanceClass}" \
  --output table

# 特定エンジンバージョンのフィルタ（例: MySQL 5.7）
aws rds describe-db-instances \
  --query "DBInstances[?Engine=='mysql'&&starts_with(EngineVersion,'5.7')].{ID:DBInstanceIdentifier,Version:EngineVersion,Class:DBInstanceClass}" \
  --output table

# Extended Support料金算出用のvCPU情報
aws rds describe-db-instances \
  --query "DBInstances[].{ID:DBInstanceIdentifier,Engine:Engine,Version:EngineVersion,MultiAZ:MultiAZ,VCPUs:ProcessorFeatures}" \
  --output table
```

### Aurora

```bash
# Auroraクラスター一覧（エンジン・バージョン付き）
aws rds describe-db-clusters \
  --query "DBClusters[].{ID:DBClusterIdentifier,Engine:Engine,Version:EngineVersion,Status:Status,Members:DBClusterMembers[].DBInstanceIdentifier}" \
  --output table

# 特定エンジンバージョンのフィルタ（例: Aurora PostgreSQL 13.x）
aws rds describe-db-clusters \
  --query "DBClusters[?Engine=='aurora-postgresql'&&starts_with(EngineVersion,'13.')].{ID:DBClusterIdentifier,Version:EngineVersion}" \
  --output table
```

### ElastiCache（Redis / Valkey）

```bash
# ElastiCacheクラスター一覧
aws elasticache describe-cache-clusters \
  --query "CacheClusters[].{ID:CacheClusterId,Engine:Engine,Version:EngineVersion,NodeType:CacheNodeType,Status:CacheClusterStatus}" \
  --output table

# レプリケーショングループ一覧
aws elasticache describe-replication-groups \
  --query "ReplicationGroups[].{ID:ReplicationGroupId,Status:Status,Description:Description,NodeGroups:NodeGroups[].{Slots:Slots,Members:NodeGroupMembers[].CacheClusterId}}" \
  --output table
```

### OpenSearch

```bash
# OpenSearchドメイン一覧
aws opensearch list-domain-names --output table

# 全ドメインの詳細（バージョン含む）
for domain in $(aws opensearch list-domain-names --query "DomainNames[].DomainName" --output text); do
  aws opensearch describe-domain --domain-name $domain \
    --query "DomainStatus.{Name:DomainName,Version:EngineVersion,InstanceType:ClusterConfig.InstanceType,Count:ClusterConfig.InstanceCount}" \
    --output table
done
```

### Lambda

```bash
# 全Lambda関数のランタイム一覧
aws lambda list-functions \
  --query "Functions[].{Name:FunctionName,Runtime:Runtime,LastModified:LastModified}" \
  --output table

# 特定ランタイムのフィルタ（例: Python 3.9）
aws lambda list-functions \
  --query "Functions[?Runtime=='python3.9'].{Name:FunctionName,Runtime:Runtime,LastModified:LastModified}" \
  --output table

# ランタイム別の関数数カウント
aws lambda list-functions \
  --query "Functions[].Runtime" \
  --output text | tr '\t' '\n' | sort | uniq -c | sort -rn
```

### Windows Server（EC2上）

```bash
# Windowsプラットフォームの稼働中EC2一覧
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" "Name=platform-details,Values=Windows*" \
  --query "Reservations[].Instances[].{ID:InstanceId,Type:InstanceType,AMI:ImageId,Platform:PlatformDetails,Name:Tags[?Key=='Name']|[0].Value}" \
  --output table

# Windows AMIのOS名・バージョン確認
for ami in $(aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" "Name=platform-details,Values=Windows*" \
  --query "Reservations[].Instances[].ImageId" \
  --output text | sort -u); do
  aws ec2 describe-images --image-ids $ami \
    --query "Images[].{AMI:ImageId,Name:Name}" \
    --output table 2>/dev/null
done

# SSM Inventoryで詳細OS情報を取得
aws ssm describe-instance-information \
  --filters "Key=PlatformTypes,Values=Windows" \
  --query "InstanceInformationList[].{ID:InstanceId,Name:ComputerName,OS:PlatformName,Version:PlatformVersion,AgentVersion:AgentVersion}" \
  --output table

# Trusted AdvisorでWindows EOS検出（Business/Enterpriseプラン必要）
aws support describe-trusted-advisor-checks \
  --language en \
  --query "checks[?name=='Amazon EC2 instances running Microsoft Windows Server end of support'].{ID:id,Name:name}" \
  --output table
```

> AMI名に `Windows_Server-2012` や `Windows_Server-2016` が含まれるものがEOL対象。
> SSM Agentがインストール済みのインスタンスのみ `describe-instance-information` で取得可能。

---

### EC2 / Amazon Linux 2

```bash
# 稼働中EC2インスタンスのAMI情報
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[].Instances[].{ID:InstanceId,Type:InstanceType,AMI:ImageId,Name:Tags[?Key=='Name']|[0].Value}" \
  --output table

# 使用中AMIの詳細確認（AL2判定）
for ami in $(aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query "Reservations[].Instances[].ImageId" \
  --output text | sort -u); do
  aws ec2 describe-images --image-ids $ami \
    --query "Images[].{AMI:ImageId,Name:Name,Description:Description}" \
    --output table 2>/dev/null
done
```

---

## 複数アカウント対応

### Organizations全アカウント一覧

```bash
aws organizations list-accounts \
  --query "Accounts[?Status=='ACTIVE'].{ID:Id,Name:Name,Email:Email}" \
  --output table
```

### 他アカウントへのスイッチロール

```bash
CREDS=$(aws sts assume-role \
  --role-arn "arn:aws:iam::TARGET_ACCOUNT_ID:role/ReadOnlyRole" \
  --role-session-name "eol-audit" \
  --query "Credentials" --output json)

export AWS_ACCESS_KEY_ID=$(echo $CREDS | python3 -c "import sys,json;print(json.load(sys.stdin)['AccessKeyId'])")
export AWS_SECRET_ACCESS_KEY=$(echo $CREDS | python3 -c "import sys,json;print(json.load(sys.stdin)['SecretAccessKey'])")
export AWS_SESSION_TOKEN=$(echo $CREDS | python3 -c "import sys,json;print(json.load(sys.stdin)['SessionToken'])")
```

> `TARGET_ACCOUNT_ID` と `ReadOnlyRole` を環境に合わせて変更する。
> 実行後は上記の各コマンドがターゲットアカウントに対して実行される。

### 全リージョンでの一括実行

```bash
for region in $(aws ec2 describe-regions --query "Regions[].RegionName" --output text); do
  echo "=== $region ==="
  aws rds describe-db-instances --region $region \
    --query "DBInstances[].{ID:DBInstanceIdentifier,Engine:Engine,Version:EngineVersion}" \
    --output table 2>/dev/null
done
```

> `rds describe-db-instances` を他のコマンドに差し替えれば各サービスで使用可能。
> 全リージョンのスキャンには数分かかる。

---

## 注意事項

- CloudShellのセッションは**20分間無操作で切断**される
- 大量のリソースがある場合は `--max-items` や `--page-size` でページネーションを制御
- 出力結果をファイルに保存する場合: `> eol-audit-results.txt` をコマンド末尾に追加
- CloudShellからファイルをダウンロードする場合: Actions → Download file
