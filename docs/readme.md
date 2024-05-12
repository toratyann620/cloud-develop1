# ドキュメントの書式について

:::info これは情報欄です。

前提知識や重要な情報をここに記述します。

:::

:::tip これはヒント欄です

教科書のコラムや、追加の知識をここに記述します。

:::

:::warning これは注意欄です

注意が必要な操作等に表示します。

:::

:::danger これは危険欄です。

特に注意を必要とする項目について記述します。

:::

---

\*1 これは注釈です。

教科書の本文中の用語等で、欄外で改めて説明が必要な場合に用います。

---

## ソースコード

下のボックスはソースコードであることを表しています。

ボックスの右上をクリックするとソースコードをコピーすることができます。

```python
for i in range(1, 101):
    if i % 3 == 0 and i % 5 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```

ソースコードが長い場合、下のように折りたたんでいる場合があります。

クリックして展開してください。

<details>
<summary>ここをクリックしてコードを展開</summary>

```python
import boto3
import http.client
import urllib.parse
import uuid
import os
from decimal import Decimal

# 各サービスのオブジェクトを取得します
s3 = boto3.client('s3')
rekognition = boto3.client('rekognition')
sns = boto3.client('sns', region_name='ap-northeast-1')
dynamodb = boto3.resource('dynamodb')

def send_response(status_code, body):
    """
    レスポンスを作成して返します。

    Parameters:
        status_code (int): HTTPステータスコード
        body (str): レスポンスボディ

    Returns:
        dict: レスポンス辞書
    """
    response = {
        'statusCode': status_code,
        'headers': {
            'Access-Control-Allow-Credentials': 'true',
            'Access-Control-Allow-Origin': '*',
            'Content-Type': 'application/json'
        },
        'body': body
    }
    return response

def lambda_handler(event, context):
    """
    Lambda関数のエントリーポイントです。顔検出とメッセージ送信を行います。

    Parameters:
        event (dict): Lambdaイベントデータ
        context (obj): Lambda実行コンテキスト

    Returns:
        dict: レスポンス辞書
    """
    # 環境変数から設定を取得
    source_bucket = os.environ['DETECT_BUCKET']
    manager_phone_number = '+19999999999'
    table_name = os.environ['DYNAMO_TABLE']

    # クエリパラメータからソース画像のキーを取得
    source_key = urllib.parse.unquote_plus(event['queryStringParameters']['filename']).replace('+', ' ')
    params = {
        'Image': {
            'S3Object': {
                'Bucket': source_bucket,
                'Name': source_key
            }
        },
        'Attributes': ['ALL']
    }

    try:
        # 画像から顔を検出
        result = rekognition.detect_faces(**params)['FaceDetails']

        if result is not None and len(result) > 0:
            print(result)
            face_id = str(uuid.uuid4())

            # 検出された顔情報をDynamoDBに保存
            table = dynamodb.Table(table_name)
            table.put_item(
                Item={
                    'id': face_id,
                    'info': convert_to_decimal(result)
                }
            )

            if result[0] and result[0]['Confidence'] >= 80:
                item = result[0]
                if item['Emotions']:
                    emotions = item['Emotions'][0]

                    # 顔の感情がネガティブであり、被写体が15歳以上の場合
                    if emotions['Confidence'] > 80 and emotions['Type'].lower() in ['angry', 'sad', 'confused', 'disgusted'] and item['AgeRange']['Low'] > 15:
                        # Amazon SNSを利用してSMSを発行してマネージャーに知らせる
                        url = f"http://tinyurl.com/api-create.php?url=https://s3.ap-northeast-1.amazonaws.com/{source_bucket}/{source_key}"
                        conn = http.client.HTTPConnection("tinyurl.com")
                        conn.request("GET", url)
                        res = conn.getresponse()
                        shortened_url = res.read().decode('utf-8')

                        # snsでメッセージを発行
                        message = f"customer: {emotions['Type'].lower()}, gender: {item['Gender']['Value'].lower()}, age range: {item['AgeRange']['Low']}-{item['AgeRange']['High']}, picture: {shortened_url}"
                        params = {
                            'Message': message,
                            'MessageStructure': 'string',
                            'PhoneNumber': manager_phone_number
                        }

                        # ショートメッセージ発行
                        sns.publish(**params)

                        # 精神を落ち着かせようということでyogadvdを返す
                        return send_response(200, '"yogadvd.jpg"')
                    else:
                        if item['AgeRange']['High'] <= 15:
                            # 15歳未満の顧客向け
                            return send_response(200, '"gameconsole.jpg"')
                        else:
                            if item['Gender']['Confidence'] > 80:
                                if item['Gender']['Value'] == 'Female':
                                    # 女性の顧客向け
                                    return send_response(200, '"lipstick.jpg"')
                                else:
                                    if (item['Mustache']['Confidence'] > 80 and item['Mustache']['Value']) or (item['Beard']['Confidence'] > 80 and item['Beard']['Value']):
                                        # ヒゲまたは髭を持つ顧客向け
                                        return send_response(200, '"beardoil.jpg"')
                                    else:
                                        # その他の顧客向け
                                        return send_response(200, '"safetyrazor.jpg"')
                            else:
                                # 性別が不明な顧客向け
                                return send_response(200, '"tv.jpg"')
            else:
                return send_response(200, 'could not detect faces')
        else:
            return send_response(200, 'could not detect faces')

    except Exception as e:
        print(e)
        return send_response(500, 'an error occured')

def convert_to_decimal(data):
    """
    float型のデータをDecimal型に変換します。DynamoDBにデータを保存する際に必要です。

    Parameters:
        data (dict or list or float): 変換するデータ

    Returns:
        dict or list or Decimal: 変換後のデータ
    """
    if isinstance(data, list):
        return [convert_to_decimal(item) for item in data]
    elif isinstance(data, dict):
        return {key: convert_to_decimal(value) for key, value in data.items()}
    elif isinstance(data, float):
        return Decimal(str(data))
    else:
        return data
```

</details>

## その他

ドキュメントの右上にあるトグルボタンでダークモードとライトモードの切り替えができます。

ノートパソコンなど画面が小さめのデバイスで文字が小さくて見えにくい場合は、ブラウザのズーム設定を 125%や 150%にすると、全体のレイアウトが崩れることなく拡大することができます。
