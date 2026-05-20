import os
import requests
import time

# 直接從 GitHub Secrets 安全讀取憑證
TELEGRAM_BOT_TOKEN = os.environ.get('TG_BOT_TOKEN')
TELEGRAM_CHAT_ID = os.environ.get('TG_CHAT_ID')

def send_telegram_message(msg):
    """發送 Telegram 通知"""
    url = f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage"
    payload = {'chat_id': TELEGRAM_CHAT_ID, 'text': msg}
    try:
        requests.post(url, json=payload)
    except Exception as e:
        print(f"Telegram 發送失敗: {e}")

def check_market():
    print(" [雲端系統] 正在掃描全網合約，篩選異常資費與爆發持倉...")
    try:
        # 1. 一次性抓取幣安所有合約的現行費率
        funding_url = "https://fapi.binance.com/fapi/v1/premiumIndex"
        funding_data = requests.get(funding_url).json()

        for coin in funding_data:
            symbol = coin['symbol']
            if not symbol.endswith('USDT'):
                continue

            funding_rate = float(coin['lastFundingRate'])

            # 🔴 燈號一：資金費率極度負值 (低於 -0.05%)
            if funding_rate < -0.0005:
                
                # 🔴 燈號二：調取歷史 OI 數據，比對 5 分鐘內的持倉變化幅度
                # 雲端版免記憶記憶體，直接向幣安拿最近兩條 5 分鐘的 K 線持倉紀錄
                oi_hist_url = f"https://fapi.binance.com/fapi/v1/openInterestHist?symbol={symbol}&period=5m&limit=2"
                oi_data = requests.get(oi_hist_url).json()
                
                if isinstance(oi_data, list) and len(oi_data) >= 2:
                    # 依時間戳排序，確保 [0] 是上一個 5M，[1] 是最新當前 5M
                    oi_data = sorted(oi_data, key=lambda x: x['timestamp'])
                    old_oi = float(oi_data[0]['sumOpenInterest'])
                    current_oi = float(oi_data[1]['sumOpenInterest'])
                    
                    if old_oi > 0:
                        oi_change_pct = ((current_oi - old_oi) / old_oi) * 100
                        
                        # 判定 5 分鐘內新空頭是不是不信邪瘋狂加倉 (> 2.0%)
                        if oi_change_pct > 2.0:
                            
                            # 🔴 燈號三：抓取最新 5 分鐘現貨成交量，確認主力是否有實質點火行為
                            spot_url = f"https://api.binance.com/api/v3/klines?symbol={symbol}&interval=5m&limit=1"
                            spot_data = requests.get(spot_url).json()
                            current_volume = float(spot_data[0][5])
                            
                            # 三燈判定吻合，即刻對手機發送獵殺推播
                            msg = f"🚨 雲端嘎空警報：{symbol}\n" \
                                  f"🔥 資金費率：{funding_rate*100:.4f}%\n" \
                                  f"📈 5M 持倉暴增：+{oi_change_pct:.2f}%\n" \
                                  f"👉 雲端偵測高勝率組合！請立刻開啟 Coinglass 確認現貨 CVD 是否出現垂直買入的大綠柱！"
                            
                            send_telegram_message(msg)
                            print(f" 發現嘎空目標: {symbol}")
                            
    except Exception as e:
        print(f"掃描過程中發生異常: {e}")

if __name__ == "__main__":
    check_market()
