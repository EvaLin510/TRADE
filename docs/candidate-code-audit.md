# 候選策略程式碼稽核與 MEXC 移植決策

更新日期：2026-10-30  
研究範圍：MEXC 現貨、約 100 USDT、只做多、不使用槓桿；本階段是策略與回測設計評估，不代表已證明可獲利。

## 結論


候選專案 `pooriag/crypto-strategy-backtest` 值得借用的是研究流程與風控基準，不是其回測績效可直接外推至 MEXC。程式包含訊號延遲、按換手計成本及 walk-forward 選參數；但成交時序測試沒有證明收盤訊號後的實際成交價格，資料為每日參考價，且忽略組合 sleeve 再平衡成本、資產僅 BTC/ETH。其報告也未顯示策略能以統計把握度勝過買進持有。

本案固定測試 BTC 現貨 20/100 日均線倉位開關：快線高於慢線持有 BTC，否則持有 USDT。它目前只適合作為回撤管理候選，不宣稱 alpha。暫不以 Dan Zanger/Qullamaggie 突破策略作首選：現有成本與樣本外證據不足以支持部署。

## 程式實作核對

依候選專案 `src/cryptobt/signals.py`、`backtest.py`、`walkforward.py` 原始碼與 `tests/test_backtest.py` 檢視：

| 項目 | 實作 | 對本案的判讀 |
|---|---|---|
| 訊號 | TSMOM 看回溯報酬正負；SMA 快慢線交叉；現貨倉位為 0 或 1 | 規則簡單可解釋；固定 20/100 日 MA 作基準，避免全歷史挑參數 |
| 成交時點 | `backtest.py` 使用 `held = target.shift(delay)`，再乘 close-to-close 報酬；測試只確認 position lag 一日 | 只證明倉位序列錯開，沒有驗證收盤訊號後的實際成交價格或下一根開盤成交。不能據此認定已避免成交時點偏誤；移植版須明確建模下一根日線開盤或實際成交 |
| 成本 | 預設單邊費用 10 bps + 滑價 5 bps，按倉位變化絕對值扣成本 | 重用成本架構前，改用帳戶及交易對費率；滑價以 MEXC 點差/成交資料或壓力值估算 |
| 樣本外 | warm-up 365 天；滾動約三年選歷史 Sharpe 最高參數，固定約 182 天交易，再彙報 OOS 日 | 仍需保留最終未碰過的封存 holdout；本案 20/100 結果不是 OOS |
| 波動控倉 | 30 日波動率、40% 目標、1 倍上限、10% no-trade band | 可列為後續風險控制測試；先確認 100 USDT 下的下單粒度 |
| 多資產 | 每日平均各資產淨報酬，忽略 sleeve 間再平衡成本 | 單測 BTC 可避開；納入 ETH 前需補上再平衡與最小下單量模型 |

### 候選專案作者報告（非本案重跑）

OOS 區間 2018-01 至 2026-05，BTC/ETH 等權，單邊費用與滑價共 15 bps：

| 策略 | CAGR | Sharpe | 最大回撤 | Sharpe bootstrap p 值 |
|---|---:|---:|---:|---:|
| BTC/ETH 買進持有 | 29.3% | 0.72 | -81.4% | — |
| 動能 + 波動度目標 | 25.5% | 0.91 | -50.7% | 0.24 |
| 均線交叉 | 27.1% | 0.76 | -55.9% | 0.47 |
| 均線交叉 + 波動度目標 | 17.3% | 0.69 | -42.9% | 0.57 |

作者報告顯示，趨勢策略在該樣本降低部分回撤，但 CAGR 未高於買進持有；Sharpe 差異未達統計顯著。動能加波動控倉 Sharpe 在雙倍成本壓力測試由 0.91 降至 0.46。這些是作者報告數字，不是 MEXC 實盤或本案獨立重跑。

## MEXC 移植與驗證清單

1. 確認交易對最小名義金額、數量步進、精度與帳戶實際費率。
2. 使用 MEXC BTC/USDT 資料，核對 UTC 日界、缺漏、重複並排除未收完 K 線。
3. 同期比較 BTC 買進持有、全程 USDT、固定 20/100 均線開關；使用同樣初始資金、成本與成交時點。
4. 鎖定規則後再做 walk-forward 與至少 12 個月未觸碰 holdout；回報 CAGR、Sharpe、最大回撤、曝險、換手、成本與逐年結果。
5. 模擬餘額、最小單位、零碎資產、部分成交及拒單。紙上策略可行不等於 100 USDT 能有效執行。

## 來源與限制

- 候選程式：[signals.py](https://github.com/pooriag/crypto-strategy-backtest/blob/main/src/cryptobt/signals.py)、[backtest.py](https://github.com/pooriag/crypto-strategy-backtest/blob/main/src/cryptobt/backtest.py)、[walkforward.py](https://github.com/pooriag/crypto-strategy-backtest/blob/main/src/cryptobt/walkforward.py)、[test_backtest.py](https://github.com/pooriag/crypto-strategy-backtest/blob/main/tests/test_backtest.py)。
- 候選報告：[README.md](https://github.com/pooriag/crypto-strategy-backtest)。資料標示為 Coin Metrics Community 版（CC BY-NC 4.0）；程式碼標示 MIT。此報告未在本案環境獨立重跑。
- 專案限於 BTC/ETH、單一歷史路徑與日頻參考價，無 MEXC order book/fill；bootstrap 無法生成樣本中未出現的新市場制度。

## 下一階段

把 MEXC 20/100 結果視為探索性基準；接著預先固定 holdout、檢查額外一日延遲和小額下單/零碎資產影響，再決定是否值得延伸至多資產。現階段不接帳戶 API 或送單。
