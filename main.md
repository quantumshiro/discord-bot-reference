# bot作成方法　in python

ここではdiscordのbotを作成します
なのでdiscordを登録していない人は登録してください。

https://discord.com/

このurlから登録してください

discord.pyというライブラリを使用するため
以下のコードを実行してください

```
pip install discord.py
```

そして適当な場所でディレクトリを作成してください


sample:
---
```
cd ~
mkdir discord-bot
cd discord-bot
```

```
# インストールした discord.py を読み込む
import discord

# 自分のBotのアクセストークンに置き換えてください
TOKEN = 'xxxxxx-xxx-xx.xxx'

# 接続に必要なオブジェクトを生成
client = discord.Client()

# 起動時に動作する処理
@client.event
async def on_ready():
    # 起動したらターミナルにログイン通知が表示される
    print('ログインしました')

# メッセージ受信時に動作する処理
@client.event
async def on_message(message):
    # メッセージ送信者がBotだった場合は無視する
    if message.author.bot:
        return
    # 「/neko」と発言したら「にゃーん」が返る処理
    if message.content == '/neko':
        await message.channel.send('にゃーん')

# Botの起動とDiscordサーバーへの接続
client.run(TOKEN)
```
上記のコードを適当な名前で保存してください。

ここでは ```discord-bot.py```
という名前にします

ターミナルでさっきのコードを実行します

```
python discord-bot.py
```

discordの方を見てください
botが立ち上がっていることがわかるでしょうか?

botを作るときには基本的に、
- イベント
- イベントが起こったときの処理
これらの2つを用いて記述します。

こんなことしかできないの？と思ったかもしれません。

以下では他に使える機能を垂れ流しておきます

```
#メッセージ受信時
@client.event # イベントを受信するための構文（デコレータ）
async def on_message(message): # イベントに対応する関数と受け取る引数
    ... # 処理いろいろ
```

```
#Bot起動時に実行される
@client.event
async def on_ready():
    ...
```

```
"""リアクション追加時に実行されるイベントハンドラ"""
@client.event
async def on_reaction_add(reaction, user):
    ...
```

```
"""新規メンバー参加時に実行されるイベントハンドラ"""
@client.event
async def on_member_join(member):
    ...
```

```
"""メンバーのボイスチャンネル出入り時に実行されるイベントハンドラ"""
@client.event
async def on_voice_state_update(member, before, after):
    ...
```

このように色々な動作ができるになっています

最後に天気予報のbotを写経して終わりましょう

```main.py```
---
```
import discord
import asyncio
import requests
import json
from time import sleep

f = open("config.json", encoding='utf-8')
config = json.load(f)
f.close()

f = open("weathercode.json", encoding='utf-8')
weathercode = json.load(f)
f.close()

token = config["discord"]["token"]
yahoo_appid = config["yahoo"]["appid"]
weather_key = config["openweathermap"]["key"]

client = discord.Client()

def get_Weather(lot,lat):
    appid = weather_key
    output = "json"
    url = "http://api.openweathermap.org/data/2.5/weather?"
    url += "APPID=" + appid
    url += "&lat=" + lat
    url += "&lon=" + lot
    url += "&units=" + "metric" 
    r = requests.get(url)
    res = r.json()
    return res

def get_Coordinates(location_name):
    appid = yahoo_appid
    output = "json"
    query = location_name
    url = "https://map.yahooapis.jp/geocode/V1/geoCoder?"
    url += "appid=" + appid
    url += "&output=" + output
    url += "&query=" + query
    r = requests.get(url)
    res = r.json()
    return res["Feature"][0]["Geometry"]["Coordinates"]

def get_Weather_info(location_name):
    reslist = get_Coordinates(location_name).split(",")
    weather = get_Weather(reslist[0],reslist[1])
    em = discord.Embed(colour=0x3498db)
    em.add_field(name="天気", value=weathercode[str(weather["weather"][0]["id"])], inline=True)
    em.add_field(name="気温", value=str(int(weather["main"]["temp"]))+'°C', inline=True)
    em.add_field(name="湿度", value=str(weather["main"]["humidity"])+'％', inline=True)
    em.add_field(name="風速", value=str(weather["wind"]["speed"])+'m', inline=True)
    em.set_author(
        name=location_name+'の気象情報',
        icon_url='https://upload.wikimedia.org/wikipedia/commons/1/15/OpenWeatherMap_logo.png'
        )
    em.set_thumbnail(url='http://openweathermap.org/img/w/' + weather["weather"][0]["icon"].replace('n', 'd') +'.png')
    return em

@client.event
async def on_ready():
    print('Logged in as')
    print(client.user.name)
    print(client.user.id)
    print('------')

@client.event
async def on_message(message):
    messagelist = message.content.split()
    if len(messagelist) >= 2:
        if messagelist[0] == "天気":
            location_name = messagelist[1]
            em = get_Weather_info(location_name)
            return await client.send_message(message.channel,embed=em)

        if messagelist[1] == "天気":
            location_name = messagelist[0]
            em = get_Weather_info(location_name)
            return await client.send_message(message.channel,embed=em)


if __name__ == "__main__":
    client.run(token)
```

```config.json```
---
```
{
    "yahoo": {
        "appid": "xxxxxxxxxxxxxxxxxxxxxxxxxx"
    },
    "openweathermap": {
        "key": "xxxxxxxxxxxxxxxxxxxxxxxxxx"
    },
    "discord": {
        "token": "xxxxxxxxxxxxxxxxxxxxxxxxxx"
    }
}
```

```weathercode.json```
---
```
{
    "200": "小雨と雷雨",
    "201": "雨と雷雨",
    "202": "大雨と雷雨",
    "210": "光雷雨",
    "211": "雷雨",
    "212": "重い雷雨",
    "221": "ぼろぼろの雷雨",
    "230": "小雨と雷雨",
    "231": "霧雨と雷雨",
    "232": "重い霧雨と雷雨",
    "300": "中くらいの霧雨",
    "301": "霧雨",
    "302": "重い強度霧雨",
    "310": "中くらいの霧雨の雨",
    "311": "霧雨の雨",
    "312": "重い強度霧雨の雨",
    "313": "にわか雨と霧雨",
    "314": "重いにわか雨と霧雨",
    "321": "にわか霧雨",
    "500": "小雨",
    "501": "適度な雨",
    "502": "重い強度の雨",
    "503": "非常に激しい雨",
    "504": "極端な雨",
    "511": "雨氷",
    "520": "中くらいのにわか雨",
    "521": "にわか雨",
    "522": "重い強度にわか雨",
    "531": "不規則なにわか雨",
    "600": "小雪",
    "601": "雪",
    "602": "大雪",
    "611": "みぞれ",
    "612": "にわかみぞれ",
    "615": "光雨と雪",
    "616": "雨や雪",
    "620": "光のにわか雪",
    "621": "にわか雪",
    "622": "重いにわか雪",
    "701": "ミスト",
    "711": "煙",
    "721": "ヘイズ",
    "731": "砂、ほこり旋回する",
    "741": "霧",
    "751": "砂",
    "761": "ほこり",
    "762": "火山灰",
    "771": "スコール",
    "781": "竜巻",
    "800": "晴天",
    "801": "薄い雲",
    "802": "雲",
    "803": "曇りがち",
    "804": "厚い雲",
    "900": "竜巻",
    "901": "熱帯低気圧",
    "902": "ハリケーン",
    "903": "寒い",
    "904": "暑い",
    "905": "強い風",
    "906": "雹",
    "951": "落ち着いた",
    "952": "弱い風",
    "953": "そよ風",
    "954": "風",
    "955": "爽やかな風",
    "956": "強風",
    "957": "暴風に近い強風",
    "958": "暴風",
    "959": "深刻な暴風",
    "960": "嵐",
    "961": "暴風雨",
    "962": "ハリケーン"
  }
```

main.pyのコードはdiscord.pyの他にもrequest
というライブラリも必要になるのでpipでinstallしておきましょう

apiが必要になるので獲得しておきましょう

OpenWeatherMap https://openweathermap.org/
Yahoo!API https://e.developer.yahoo.co.jp/register


以上、discordのbot制作でした。
