# MEXC 現貨策略可行性初篩

日期：2026-10-29  
範圍：MEXC USDT 現貨、約 100 USDT、本金不使用槓桿。  
目的：在開始編寫或部署交易程式前，先篩選有研究依據的策略假設。

## 結論

目前沒有足夠證據證明任何候選策略能在加密貨幣現貨長期、穩定地打敗 BTC 買進持有。若目標是控制回撤，低頻趨勢過濾值得保留為首要候選；Dan Zanger 放量突破則暫不列為核心策略。

## 候選策略證據

| 策略 | 公開樣本外結果 | 初篩判斷 |
|---|---|---|
| 日線動能／均線趨勢 | BTC、ETH 等權組合研究：每筆成本 15 bps 後，均線交叉 CAGR 27.1%、最大回撤 -55.9%；買進持有 CAGR 29.3%、最大回撤 -81.4%。風險調整報酬差異未達統計顯著。 | 保留為回撤控制候選，不宣稱可提高報酬。 |
| 加密貨幣動能突破 | Qullamaggie 類突破研究測試約 671,500 組組合。三個事前鎖定版本在 2024-07 至 2026-07 holdout 全部虧損，單筆約 -0.13R 至 -0.33R。 | 不採為核心策略；若保留，只能作比較組。 |
| 多幣種趨勢跟隨 | 一項四幣研究報告凍結參數後 263 日正報酬，但曝險高度集中 BTC，且該區間包含原 holdout 與其後延伸資料，不是全新 holdout。 | 次要參考；先不移植其複雜配置。 |

## 主要來源

1. [Crypto Strategy Backtest：BTC/ETH 日線趨勢與動能樣本外比較](https://github.com/pooriag/crypto-strategy-backtest)  
   研究使用 walk-forward、每筆 15 bps 成本、bootstrap 比較與多重測試修正。作者結論是沒有策略能以統計信心勝過買進持有，但趨勢策略可降低部分回撤。
2. [Momentum Breakout Postmortem：Qullamaggie 類加密突破策略](https://github.com/HodlHook/momentum-breakout-postmortem)  
   使用約 671,500 組測試組合及未碰過的 2024-07 至 2026-07 holdout。該類策略在 holdout 失效；作者也揭露倖存者偏誤等限制。
3. [Systematic Cryptocurrency Research：多幣種趨勢與 BTC 市占均值回歸](https://github.com/PeterLP123/systematic-crypto-research)  
   凍結策略的 263 日觀察中，趨勢組合為正，BTC 市占均值回歸為負；趨勢組合主要曝險於 BTC，且觀察期間不是全新獨立測試集。
4. [MEXC 官方費率調整公告](https://www.mexc.com/announcements/article/adjustment-to-trading-fees-for-innovation-zone-assessment-zone-and-meme-17827791538769)  
   API 費率依交易區域與交易對而異；部分區域於 2026-09-30 起調整。回測應以實際幣對適用費率為準，不套用單一費率到所有標的。

## 初篩決策與下一步

- 首要候選：低頻日線趨勢／均線規則，研究目標先定為降低回撤，不假設能打敗 BTC。
- 比較組：BTC 買進持有、ETH 買進持有，以及 Dan Zanger 放量突破的可程式化版本。
- 放量突破目前不進入核心策略，因相近的加密貨幣突破研究在未見樣本外失效。
- 只測 BTC、ETH 等大型標的，避免一開始就納入低流動性幣種和倖存者偏誤。
- 下一階段先檢查候選 GitHub 專案的訊號、成交時點、交易成本、資料處理及樣本切分，再決定是否能重用程式；不從零建立回測框架。

## 證據限制

上述數字來自公開 GitHub 專案作者的研究報告與程式碼，尚未在此環境獨立重跑或由第三方驗證。不同研究使用不同交易所、資產池、費率和樣本期間，不能直接把績效數字當成同條件競賽排名。MEXC 的實際費率與成交滑價仍須按帳戶與交易對確認。
