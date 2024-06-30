MACでPythonの仮想環境を使用してFlaskアプリケーションをZappaを使ってLambdaにデプロイする手順を以下に示します。

### 手順1: 仮想環境の作成

まず、Pythonの仮想環境を作成します。

#### 1.1 必要なツールのインストール

仮想環境を作成するために `venv` モジュールを使用します。

```bash
python3 -m venv myenv
```

#### 1.2 仮想環境のアクティブ化

仮想環境をアクティブにします。

```bash
source myenv/bin/activate
```

### 手順2: 必要なパッケージのインストール

仮想環境内で必要なパッケージをインストールします。

```bash
pip install Flask PyMuPDF4LLM zappa
```

### 手順3: Flaskアプリの作成

Flaskアプリケーションを作成します。以下のコードを `app.py` として保存します。

```python
from flask import Flask, request, jsonify
import fitz  # PyMuPDF

app = Flask(__name__)

@app.route('/extract_text', methods=['POST'])
def extract_text():
    if 'file' not in request.files:
        return jsonify({"error": "No file provided"}), 400

    file = request.files['file']
    doc = fitz.open(stream=file.read(), filetype="pdf")
    text = ""
    for page in doc:
        text += page.get_text()

    return jsonify({"text": text})

if __name__ == '__main__':
    app.run()
```

### 手順4: Zappaのセットアップ

Zappaを使用してFlaskアプリケーションをAWS Lambdaにデプロイします。

#### 4.1 Zappaの初期化

Zappaの設定ファイルを作成します。

```bash
zappa init
```

対話形式のプロンプトに従い、設定を行います。例えば：

- ステージ名 (Stage name) : `dev`
- S3バケット (S3 bucket) : 任意のバケット名

#### 4.2 Zappa設定ファイルの確認

生成された `zappa_settings.json` ファイルを確認し、必要に応じて編集します。

```json
{
    "dev": {
        "app_function": "app.app",
        "aws_region": "us-east-1",
        "s3_bucket": "your-s3-bucket-name"
    }
}
```

### 手順5: Lambdaへデプロイ

#### 5.1 デプロイ

```bash
zappa deploy dev
```

#### 5.2 更新時のデプロイ

コードを更新した場合、以下のコマンドで再デプロイします。

```bash
zappa update dev
```

### 手順6: テスト

デプロイが成功すると、エンドポイントURLが表示されます。このURLを使用して、APIをテストできます。例えば、`curl` コマンドを使用してPDFファイルを送信し、テキストを抽出します。

```bash
curl -X POST https://your-api-endpoint/dev/extract_text -F 'file=@/path/to/your/file.pdf'
```

これで、仮想環境を使用してPyMuPDF4LLMを使用したFlask APIをZappaを通じてAWS Lambdaにデプロイする手順が完了です。
