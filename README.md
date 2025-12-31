# ScrapeGraphAI を使用した LLM ベースの Webスクレイピング

[![Promo](https://github.com/bright-jp/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/)

このガイドでは、ScrapeGraphAI と大規模言語モデル（LLM）を使用して Webスクレイピングを簡素化し、データ抽出を自動化する方法を説明します。

- [なぜ ScrapeGraphAI を使用するのか？](#why-use-scrapegraphai)
- [前提条件](#prerequisites)
- [環境のセットアップ](#setting-up-your-environment)
- [ScrapeGraphAI でデータをスクレイピングする](#scraping-data-with-scrapegraphai)
  - [スクレイパーコードの作成](#writing-the-scraper-code)
  - [ScrapeGraphAI でプロキシを使用する](#using-proxies-with-scrapegraphai)
- [データのクリーニングと前処理](#cleaning-and-preparing-data)

## Why use ScrapeGraphAI?

従来の Webスクレイピングでは、各 Webサイトのレイアウトに固有の複雑で時間のかかるコードを書く必要があり、サイト変更があると壊れてしまうことがよくあります。

[ScrapeGraphAI](https://scrapegraphai.com/) は大規模言語モデル（LLM）を活用して、人間のようにデータを解釈・抽出できるため、レイアウトではなくデータそのものに集中できます。LLM を統合することで、ScrapeGraphAI はデータ抽出を改善し、コンテンツ集約を自動化し、リアルタイム分析を可能にします。

## Prerequisites

以下の前提条件が必要です。

* [Python 3.x](https://www.python.org/downloads/)。
* [GPT-4](https://openai.com/index/gpt-4/) にアクセスするための [OpenAI アカウント](https://platform.openai.com/signup)。

## Setting Up Your Environment

仮想環境を作成します。

```bash
python -m venv venv
```

次に、仮想環境を有効化します。macOS と Linux の場合：

```bash
source venv/bin/activate
```

Windows の場合は、このコマンドを使用できます：

```powershell
venv\Scripts\activate
```

ScrapeGraphAI と依存関係をインストールします。

```bash
pip install scrapegraphai
playwright install
```

`playwright install` コマンドは、Chromium、Firefox、WebKit に必要なブラウザをセットアップします。

環境変数を安全に管理するために、`python-dotenv` をインストールします。

```bash
pip install python-dotenv
```

API キーなどの機密情報は保護することが重要です。そのため、環境変数は `.env` ファイルに保存し、コードファイルとは分離してください。

プロジェクトディレクトリに `.env` という名前の新しいファイルを作成し、OpenAI key を指定する次の行を追加します。

```python
OPENAI_API_KEY="your-openai-api-key"
```

このファイルは Git などのバージョン管理システムにコミットしないでください。これを防ぐために、`.gitignore` ファイルに `.env` を追加します。

## Scraping Data with ScrapeGraphAI

まず、Webスクレイピング手法の練習用に特化したデモサイトである [Books to Scrape](http://books.toscrape.com/) から商品データをスクレイピングします。この Webサイトはオンライン書店を模しており、価格、評価、在庫状況を含むさまざまなジャンルの書籍を提供しています。

![Books to Scrape website](https://github.com/bright-jp/web-scraping-with-scrapegraphai/blob/main/images/Books-to-Scrape-website-1024x772.png)

従来の HTML スクレイピングでは、データを抽出するために要素を手動で調査します。ScrapeGraphAI では、プロンプトで必要なデータを指定するだけで、LLM が抽出してくれます。

ScrapeGraphAI は、さまざまなスクレイピング要件に対応する複数の graph を提供しています。

* **[SmartScraperGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#smartscrapermultigraph)**: プロンプトと URL、またはローカルファイルを使用する単一ページのスクレイパーです。
* **[SearchGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#searchgraph)**: 検索エンジンの結果からデータを抽出する複数ページのスクレイパーです。
* **[SpeechGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#speechgraph)**: text-to-speech を追加して SmartScraperGraph を拡張し、音声ファイルを生成します。
* **[ScriptCreatorGraph](https://scrapegraph-ai.readthedocs.io/en/latest/scrapers/types.html#scriptcreatorgraph-scriptcreatormultigraph)**: 指定した URL をスクレイピングする Python スクリプトを出力します。

また、ノードを組み合わせてニーズに合うカスタム graph を作成することもできます。

正確なスクレイピングを行うために、明確なプロンプト、モデル選択、ジオロケーション制限コンテンツ向けのプロキシ、効率のための headless モードなど、スクレイパーを適切に設定してください。適切なセットアップは、抽出データの精度に影響します。

### Writing the Scraper Code

`app.py` という名前の新しいファイルを作成し、次のコードを挿入します。

```python
from dotenv import load_dotenv
import os
from scrapegraphai.graphs import SmartScraperGraph

# Load environment variables from .env file
load_dotenv()

# Access the OpenAI API key
OPENAI_API_KEY = os.getenv('OPENAI_API_KEY')

# Configuration for ScrapeGraphAI
graph_config = {
    "llm": {
        "api_key": OPENAI_API_KEY,
        "model": "openai/gpt-4o-mini",
 }
}

# Define the prompt and source
prompt = "Extract the title, price and availability of all books on this page."
source = "http://books.toscrape.com/"

# Create the scraper graph
smart_scraper_graph = SmartScraperGraph(
 prompt=prompt,
 source=source,
 config=graph_config
)

# Run the scraper
result = smart_scraper_graph.run()

# Output the results
print(result)
```

このコードは、環境変数管理用の `os` や `dotenv`、およびスクレイピング用の ScrapeGraphAI の `SmartScraperGraph` クラスなど、必須モジュールをインポートします。`dotenv` を介して環境変数を読み込み、機密データ（例：API キー）を安全に保ちます。次に、モデルと API key を指定してスクレイピング用の LLM を設定します。この設定に加え、サイト URL とスクレイピングプロンプトで `SmartScraperGraph` を定義し、`run()` メソッドで実行して指定データを収集します。

コードを実行するには、ターミナルで `python app.py` コマンドを使用します。出力は次のようになります。

```
{
    "books": [
        {
            "title": "A Light in the Attic",
            "price": "£51.77",
            "availability": "In stock"
        },
        {
            "title": "Tipping the Velvet",
            "price": "£53.74",
            "availability": "In stock"
        }, ...
       ]
}
```

> **Note**:
> 
> 潜在的なエラーを回避するために、[`grpcio`](https://pypi.org/project/grpcio/) パッケージがインストールされていることを確認してください。

ScrapeGraphAI は Webスクレイピングにおけるデータ抽出部分を簡単にしますが、CAPTCHA や IP ブロックなどの一般的な課題は依然として存在します。

ブラウジング行動を模倣するために、コード内に時間差の遅延を実装できます。また、ローテーティングプロキシを利用して検知を回避することも可能です。さらに、Bright Data の CAPTCHA solver や Anti Captcha のような CAPTCHA 解決サービスをスクレイパーに統合し、CAPTCHA を自動的に解くこともできます。

> **Important**:
> 
> 必ず Webサイトの利用規約に準拠していることを確認してください。個人利用目的のスクレイピングは許容されることが多い一方で、データの再配布は法的な影響を伴う可能性があります。

## Using Proxies with ScrapeGraphAI

ScrapeGraphAI では、IP ブロックを回避し、ジオロケーション制限のあるページにアクセスするためにプロキシサービスを設定できます。そのためには、`graph_config` に次を追加します。

```python
graph_config = {
    "llm": {
        "api_key": OPENAI_API_KEY,
        "model": "openai/gpt-4o-mini",
 },
    "loader_kwargs": {
        "proxy": {
            "server": "broker",
            "criteria": {
                "anonymous": True,
                "secure": True,
                "countryset": {"US"},
                "timeout": 10.0,
                "max_tries": 3
 },
 },
 }
}
```

この設定は、条件に一致する無料のプロキシサービスを使用するように ScrapeGraphAI に指示します。

Bright Data のようなプロバイダーのカスタムプロキシサーバーを使用するには、`graph_config` を次のように変更し、server URL、username、password を挿入します。

```python
graph_config = {
    "llm": {
        "api_key": OPENAI_API_KEY,
        "model": "openai/gpt-4o-mini",
 },
    "loader_kwargs": {
        "proxy": {
            "server": "http://your_proxy_server:port",
            "username": "your_username",
            "password": "your_password",
 },
 }
}
```

カスタムプロキシサーバーの利用には、特に大規模な Webスクレイピングにおいて複数の利点があります。プロキシの場所を制御できるため、ジオロケーション制限のあるコンテンツにアクセス可能になります。さらに、カスタムプロキシは無料プロキシよりも信頼性とセキュリティが高く、IP ブロックやレート制限のリスクを低減します。

## Cleaning and Preparing Data

スクレイピング後、特に AI モデルで使用する場合は、データのクリーニングと前処理が重要です。クリーンなデータにより、モデルは正確で一貫性のある情報から学習でき、性能と信頼性が向上します。データクリーニングには通常、欠損値の処理、データ型の修正、テキストの正規化、重複の削除などが含まれます。

以下は、[pandas](https://pypi.org/project/pandas/) を使用してスクレイピングしたデータをクリーニングする例です。

```python
import pandas as pd

# Convert the result to a DataFrame
df = pd.DataFrame(result["books"])

# Remove currency symbols and convert prices to float
df['price'] = df['price'].str.replace('£', '').astype(float)

# Standardize availability text
df['availability'] = df['availability'].str.strip().str.lower()

# Handle missing values if any
df.dropna(inplace=True)

# Preview the cleaned data
print(df.head())
```

このコードは、書籍価格から通貨記号を削除し、在庫状況を小文字に変換して標準化し、欠損値があれば処理することでデータをクリーニングします。

このコードを実行する前に、データ操作のための `pandas` ライブラリをインストールします。

```bash
pip install pandas
```

ターミナルを開いて `python app.py` を実行します。出力は次のようになります。

```
                                   title  price availability
0                   A Light in the Attic  51.77     in stock
1                     Tipping the Velvet  53.74     in stock
2                             Soumission  50.10     in stock
3                          Sharp Objects  47.82     in stock
4  Sapiens: A Brief History of Humankind  54.23     in stock
```

これはスクレイピングデータのクリーニング例にすぎず、プロセスはデータと LLM のユースケースに応じて異なります。クリーニングにより、言語モデルは構造化された意味のある入力を受け取れます。最も人気のある [AI use cases](https://brightdata.jp/ai) について学んでください。

このチュートリアルの全コードは [this GitHub repo](https://github.com/mikeyny/scrapegraphai-demo) で確認できます。

## Conclusion

ScrapeGraphAI は LLM を使用して適応型の Webスクレイピングを実現し、Webサイトの変更に合わせて調整しながら、データをインテリジェントに抽出します。ただし、スクレイピングをスケールさせると、IP ブロック、CAPTCHA、法的コンプライアンスなどの課題が伴います。

Bright Data は、これらの課題に対処するソリューションとして、[Web Scraper APIs](https://brightdata.jp/products/web-scraper)、[proxy services](https://brightdata.jp/proxy-types)、および [Serverless Scraping](https://brightdata.jp/products/web-scraper/functions) を提供しています。また、100 以上の人気 Webサイトからの [ready-to-use datasets](https://brightdata.jp/products/datasets) も提供しています。

今すぐ無料トライアルを開始しましょう！