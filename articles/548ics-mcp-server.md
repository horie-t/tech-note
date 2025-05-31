---
title: "Terraform Registryのmoduleの最新バージョンを検索するMCPサーバを作る"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["mcp", "terraform", "claudecode"]
published: true
---

[Claude Code](https://docs.anthropic.com/ja/docs/claude-code/overview)等のコーディングエージェントを使用してTerraform等のIaCのコードを生成させた場合、module等のバージョンが古い場合があります。これは、コーディングエージェントが使用しているLLMの学習した知識は、ある時点以前のものに限定されているからです。これをナレッジカットオフといいます。

ですが、Model Context Protocol(MCP)を使用するとMCPサーバを介して最新の情報を取得して、その知識を元にコードを生成する事ができます。つまり、Terraform Registryで公開されている最新のmoduleのバージョン番号を返すMCPサーバを構築すれば、最新のバージョンを指定したTerraformのコードを生成させる事ができます。(ただし、LLMの知識にあるバージョンと最新版の間で非互換が発生していると、エラーになってしまいますが…)

本記事では、Terraform Registryのmoduleの最新バージョン番号を検索する簡単なMCPサーバを構築して、Claude Codeで使ってみます。なお、MCPについての解説は[こちら](https://zenn.dev/mkj/articles/0ed4d02ef3439c)を参照してください。

## MCPサーバのプロジェクトの作成

本記事では、[公式のPython SDK](https://github.com/modelcontextprotocol/python-sdk)を使ってMCPサーバを構築します。

まず[uv](https://docs.astral.sh/uv/)を使ってPythonのプロジェクトを作成します。

```bash
uv init --package mcp-server-tf-module-versions
cd mcp-server-tf-module-versions
```

次に依存関係を設定します。

```bash
uv add "mcp[cli]"
```

Terraform Registryの[APIの仕様書](https://developer.hashicorp.com/terraform/registry/api-docs#registry-api)を参照するとmodule最新バージョンの情報は以下の書式エンドポイントにアクセスします。

| Method | Path                                  |
| ------ | ------------------------------------- |
| GET    | <base_url>/:namespace/:name/:provider |

例えば、AWS VPCのモジュールの場合は以下のリクエストで取得できます。

```bash
curl https://registry.terraform.io/v1/modules/terraform-aws-modules/vpc/aws
```

モジュールの最新バージョン番号を取得できるように、プロジェクトの__init___.pyファイルを以下のように編集します。

```python:src/mcp_server_tf_module_versions/__init__.py
import httpx
import os

from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Terraform Registry")

@mcp.tool()
async def fetch_latest_module_version(namespace: str, name: str, provider: str):
    """Fetch latest module version number"""
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://registry.terraform.io/v1/modules/{namespace}/{name}/{provider}")
        latest_module = response.json()
        return latest_module["version"]

def main() -> None:
    mcp.run()
```

## Claude CodeからMCPサーバの呼び出し

MCPサーバが完成したので、Claude Codeから呼び出してみます。

まず、以下のMCPサーバの設定ファイルを作成します。

```json:mcp-servers.json 
{
  "mcpServers": {
    "terraformRegistry": {
      "command": "uv",
      "args": [
	"run",
        "--directory",
        "/<Pythonのプロジェクトへのパス>/mcp-server-tf-module-versions",
        "mcp-server-tf-module-versions"
      ]
    }
  }
}
```

以下のように設定ファイルを指定してClaude Codeを起動します。

```bash
claude --mcp-config mcp-servers.json
```

起動したら以下のようにプロンプトを与えてコード生成させると、MCPサーバを呼び出して最新バージョンのmoduleでtfファイルが生成される様子が見れます。

```
> terraformディレクトリにEKSのクラスタを作成するコードを書いてください。EKSはt4a.mediumのスポットインスタンスを
  使って、リージョンはus-west-2 に作成してください。Terraform AWS modulesの最新版を使用してください。

● I'll help you create Terraform code for an EKS cluster using spot instances. Let me start by creating a todo list
   to track this task.

● Update Todos
  ⎿  ☐ Create terraform directory structure
     ☐ Get latest version of AWS VPC module
     ☐ Get latest version of AWS EKS module
     ☐ Create main.tf with provider configuration
     ☐ Create VPC configuration
     ☐ Create EKS cluster configuration with t4a.medium spot instances
     ☐ Create variables.tf and outputs.tf files

● Now let me start by creating the terraform directory and getting the latest module versions.

● Bash(mkdir -p terraform)…
  ⎿  (No content)

● terraformRegistry:fetch_latest_module_version (MCP)(namespace: "terraform-aws-modules", name: "vpc", provider: …
                                                     "aws")
  ⎿  5.21.0

● terraformRegistry:fetch_latest_module_version (MCP)(namespace: "terraform-aws-modules", name: "eks", provider: …
                                                     "aws")
  ⎿  20.36.0

(中略)

● I've created a complete Terraform configuration for an EKS cluster with the following specifications:

  Key Features:
  - EKS cluster in us-west-2 region using latest AWS modules (EKS v20.36.0, VPC v5.21.0)
  - t4a.medium spot instances for cost optimization
  - Auto-scaling node group (1-10 nodes, 2 desired)
  - Custom VPC with proper subnet configuration
  - Essential cluster addons (CoreDNS, VPC CNI, etc.)

  Files created:
  - terraform/main.tf - Main configuration with VPC and EKS modules
  - terraform/variables.tf - Configurable variables
  - terraform/outputs.tf - Cluster endpoints and resource IDs
  - terraform/README.md - Usage documentation

  To deploy: cd terraform && terraform init && terraform plan && terraform apply
```

