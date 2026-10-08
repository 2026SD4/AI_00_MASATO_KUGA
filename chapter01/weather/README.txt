ご提示いただいた要件および制約に基づき、google-adk==2.2.0 を使用した天気情報提供エージェントのコード一式を作成しました。

型ヒントの適用、Python 3.11+ 互換、日本語コメントの追加、および環境変数によるAPIキーの管理を行っています。

1. .env (環境変数テンプレート)
   APIキーなどの機密情報を管理するためのテンプレートファイルです。

# Gemini APIのアクセスキー（Google AI Studio等から取得したキーを設定してください）

GEMINI_API_KEY=your_gemini_api_key_here

2. agent.py (エージェントの定義)
   エージェント本体、および天気を取得するダミーのツール関数を定義します。

import os
from typing import Dict
from dotenv import load_dotenv
from google_adk import Agent, Tool  # google-adk 2.2.0 の仕様を想定

# .envファイルから環境変数を読み込みます

load_dotenv()

# APIキーの存在を確認（直書きを防ぐ）

if not os.environ.get("GEMINI_API_KEY"):
raise ValueError("環境変数 'GEMINI_API_KEY' が設定されていません。")

def get_weather(city: str) -> str:
"""指定された都市の天気情報を取得するツール。

Args:
    city (str): 天気情報を取得したい都市の名前（例: 「東京」, 「大阪」）。

Returns:
    str: 指定された都市の天気と気温のシミュレーションデータ。
"""
# 簡易的なダミーの天気データ
dummy_weather_db: Dict[str, str] = {
    "東京": "晴れ、気温 22℃、湿度 45%",
    "大阪": "曇り、気温 20℃、湿度 60%",
    "名古屋": "雨、気温 18℃、湿度 85%",
    "札幌": "雪、気温 2℃、湿度 90%",
    "福岡": "快晴、気温 25℃、湿度 40%",
}

# 指定された都市がDBにない場合はデフォルトの回答を返す
weather_info: str = dummy_weather_db.get(
    city, "晴れ、気温 20℃ (詳細データなし)"
)
return f"{city}の現在の天気は {weather_info} です。"
# Instruction（指示文）の定義

INSTRUCTION: str = (
"ユーザーが指定した都市の天気を教えてください。\n"
"都市名が指定されていない場合は、どの都市の天気を知りたいか確認してください。"
)

# weather_agentのインスタンス化

weather_agent: Agent = Agent(
name="weather_agent",
model="gemini-3.5-flash",  # 指定されたモデル名
instruction=INSTRUCTION,
tools=[get_weather],  # ツール（関数）の登録
)

3. __init__.py (パッケージ初期化)
   パッケージとしてインポートしやすくするための初期化ファイルです。

from .agent import get_weather, weather_agent

# 外部からインポートを許可するオブジェクトを定義

__all__ = ["weather_agent", "get_weather"]

補足説明
APIキーのロード: dotenv ライブラリを使用して .env ファイルから GEMINI_API_KEY を自動的にロードし、直接コードに書き込まないようにしています。
ツール定義: get_weather 関数は、引数と戻り値に型ヒント（str）を付与し、Docstring（関数説明）を詳細に記述しています。google-adk はこのDocstringを読み取ってモデルにツールの役割を伝えるため、正確な記述が重要となります。
エージェント定義: Agent クラスにエージェント名、指定モデル（gemini-3.5-flash）、指示、およびツールをバインドしています。


作成したADKエージェントを実際に実行するための手順を説明します。

以下のステップに従って、環境構築、APIキーの設定、および実行用スクリプトの作成を行ってください。

ステップ 1: 必要なライブラリのインストール
作成したプログラムを動作させるために、必要なPythonパッケージをインストールします。コマンドライン（ターミナル）で以下のコマンドを実行してください。

pip install google-adk==2.2.0 python-dotenv

ステップ 2: APIキーの設定
同一フォルダ内にある .env ファイルを開きます。
your_gemini_api_key_here の部分をご自身の Gemini API キー（Google AI Studio などで取得したもの）に書き換えて保存します。
GEMINI_API_KEY=AIzaSyD...（実際のキー）

ステップ 3: 実行用スクリプト (main.py) の作成
エージェントを呼び出してユーザーと対話させるための実行用スクリプト main.py を同じディレクトリ（フォルダ）内に作成します。

# main.py
import os
from agent import weather_agent


def main() -> None:
    print("--- 天気予報エージェント 起動 ---")

    # パターン1: 都市名が指定されていない場合（確認が入るかテスト）
    prompt_1 = "天気を教えて"
    print(f"\nユーザー: {prompt_1}")

    # google-adk 2.2.0 では通常 .run() または .chat() メソッドで実行します
    response_1 = weather_agent.run(prompt_1)
    print(f"エージェント: {response_1}")

    # パターン2: 都市名が指定されている場合（ツールが呼ばれるかテスト）
    prompt_2 = "東京の天気を教えてください"
    print(f"\nユーザー: {prompt_2}")

    response_2 = weather_agent.run(prompt_2)
    print(f"エージェント: {response_2}")


if __name__ == "__main__":
    # 環境変数が正しくロードされているか簡易確認
    if not os.environ.get("GEMINI_API_KEY"):
        print("エラー: .env ファイルに GEMINI_API_KEY が設定されていません。")
    else:
        main()

ステップ 4: 実行する
ターミナルで以下のコマンドを実行します。

python main.py

期待される実行結果
正しく実行されると、モデル（gemini-3.5-flash）がInstruction（指示）を理解し、以下のような対話が出力されます。

--- 天気予報エージェント 起動 ---

ユーザー: 天気を教えて
エージェント: どちらの都市の天気を知りたいですか？ 都市名（例：東京、大阪など）を教えてください。

ユーザー: 東京の天気を教えてください
エージェント: 東京の現在の天気は 晴れ、気温 22℃、湿度 45% です。どうぞ快適な一日をお過ごしください。

注意点
ADKの実行メソッドについて: google-adk の具体的なインターフェース仕様により、エージェントを呼び出すメソッドが run() ではなく chat() や generate() になっている場合があります。もしエラーが出る場合は、weather_agent.run(prompt) の部分を weather_agent.chat(prompt) に書き換えてお試しください。
