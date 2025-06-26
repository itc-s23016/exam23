# AWSネットワーク＆WordPress構築手順書

## ネットワーク設計

### VPC 設計
- **VPC 名称**: `s23016_vpc`
- **CIDR ブロック**: `192.168.10.0/24`

### サブネット設計

#### Public Subnet
| 名前 | CIDR ブロック |
|------|----------------|
| s23016_public_1 | 192.168.10.0/27 |
| s23016_public_2 | 192.168.10.32/27 |

#### Private Subnet
| 名前 | CIDR ブロック |
|------|----------------|
| s23016_private_1 | 192.168.10.64/27 |
| s23016_private_2 | 192.168.10.96/27 |

### ルートテーブル設計

#### Public Subnet用
- `0.0.0.0/0` → Internet Gateway
- `192.168.10.0/24` → Local

#### Private Subnet用
- `192.168.10.0/24` → Local

## サーバー構成とIP

| サーバー種別 | 名前 | サブネット | IPアドレス |
|--------------|------|------------|------------|
| 踏み台サーバ | s23016_humidai | s23016_public_2 | 192.168.10.40 |
| Webサーバ    | s23016_web     | s23016_public_2 | 192.168.10.50 |
| DBサーバ     | s23016_DB      | s23016_private_1 | 192.168.10.70 |

## セキュリティグループ設計

### 踏み台サーバ SG
- SSH：`0.0.0.0/0`

### Webサーバ SG
- SSH：`踏み台サーバ`
- HTTP：`0.0.0.0/0`

### DBサーバ SG
- SSH：`踏み台サーバ`
- MySQL：`Webサーバ`
- ICMP：`192.168.10.32/27`

---

## 実装手順

### 1. VPC とサブネットの作成
- VPC作成（CIDR: `192.168.10.0/24`）
- 上記CIDRで 4 つのサブネットを作成

### 2. インターネットゲートウェイ作成とVPCにアタッチ

### 3. ルートテーブルの作成とサブネットへの関連付け
- Public → IGWへのルート
- Private → Localのみ（NAT後に変更）

### 4. EC2インスタンス作成（踏み台・Web・DB）
- Amazon Linux 2023
- SSH鍵ペア作成 `.pem`
- IPとサブネット、セキュリティグループ設定

### 5. Apache インストール（Webサーバ）
```bash
sudo dnf -y install httpd
sudo systemctl start httpd
sudo systemctl enable httpd

