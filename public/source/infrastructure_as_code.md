# Infrastructure as Code

## 01. 仮想サーバ(仮想マシン)のコード化

### 仮想サーバの構成管理

#### ・コード化ツールの種類

| ツール名 | 対象のProvider |
| -------- | -------------- |
| Ansible  | 要勉強         |
| Puppet   | 要勉強         |
| Chef     | 要勉強         |

#### ・Ansible

Ansibleでは，ymlの文法を用いて関数処理を実行できる．

| ファイル名   | 役割                                   |
| ------------ | -------------------------------------- |
| playbook.yml | ソフトウェアのインストールタスクの手順 |
| inventory/*  | 反映先のサーバの情報                   |
| group_vars/* | 複数のサーバへの設定                   |
| host_vars/*  | 単一のサーバへの設定                   |

<br>

## 02. コンテナのコード化

### コンテナの構成管理

#### ・コード化ツールの種類

| 名前              | 対象のProvider |
| ----------------- | -------------- |
| Dockerfile        | Docker         |
| Ansible Container | 要勉強         |

<br>


## 03. クラウドインフラストラクチャのコード化

### クラウドインフラストラクチャオーケストレーション

#### ・コード化ツールの種類

| 名前                   | 対象のクラウド |
| ---------------------- | -------------- |
| Terraform              | いろいろ       |
| AWS CloudFormation     | AWS            |
| Azure Resource Manager | Azure          |
