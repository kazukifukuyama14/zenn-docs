---
title: "X(旧：Twitter)に群がるインプレゾンビを一網打尽にする悪魔のようなスクリプトを作成した件"
emoji: "💀"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [Javascript, x, Twitter, Script]
published: true
---

## はじめに

X（旧：Twitter）を使っていて、こんな経験はありませんか？

- トレンドに便乗した意味不明なリプライの嵐
- 「310万円儲けました💰」系の投資詐欺スパム
- 全く関係ない配信宣伝やゲーム誘導
- 同じアカウントが何度も連投するリプライスパム

これらのインプレゾンビたちに日々悩まされ、ついに我慢の限界を超えた私が、悪魔のような自動ブロックスクリプトを開発しました。

## 開発の経緯

### きっかけ：連投スパムの多さ

情報収集する際、以下のような連投が多くタイムラインがかなり汚染されていきました：

![image1](https://cdn.abacus.ai/images/a4ba1290-c3d8-4ffe-aa67-d08728a55d82.png)

### エスカレートする手口

時間が経つにつれ、スパムの手口はより巧妙になっていきました：

1. 投資詐欺系：「堀江氏も注目」「310万円儲けました」
2. 配信誘導系：「Let's streammmm」「going live」
3. 暗号通貨系：「Galxe Leaderboard」「quest complete」
4. 偽装日本人：外国人が日本人を装った怪しいアカウント

![image2](https://cdn.abacus.ai/images/3ec28f58-8cfb-42f1-bc9d-bf80f1385778.png)

![image3](https://cdn.abacus.ai/images/f4df1497-43c1-4dc5-b72d-d7f985bac2af.png)

### 手動ブロックの限界

毎日数十個のスパムアカウントを手動でブロックするのは現実的ではありません。  
そこで、**パターン認識による自動ブロックの発想**に至りました。

## スクリプトの機能

### 1. 連投スパム検出

同一アカウントが短時間で複数回リプライした場合、自動的にブロック対象として判定します。

### 2. 投資詐欺検出

以下のパターンを検出：

- 儲け話キーワード（「儲けました」「利益」「稼げた」）
- 具体的金額表示（「310万円」「○億円」）
- 他人IDメンション + 金額の組み合わせ
- 「堀江氏も注目」などの詐称フレーズ

### 3. 無関係リプライ検出

元ポストと全く関係ない内容のリプライを検出：

- 配信・ゲーム宣伝
- 暗号通貨・NFT関連の宣伝
- 無関係な挨拶や定型文
- 怪しいリンクの投稿

### 4. 視覚的フィードバック

ブロック時には色分けされた通知が表示されます：

- 🚫 投資詐欺: 紫色
- 🚫 無関係リプライ: 青色
- 🚫 配信宣伝: 紫色
- 🚫 暗号通貨宣伝: 緑色
- 🚫 連投検出: 赤色

## 導入方法

### Step 1: Tampermonkeyのインストール

- Chrome/Edge: [Chrome Web Store](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo)からインストール
- Firefox: [Firefox Add-ons](https://addons.mozilla.org/ja/firefox/addon/tampermonkey/)からインストール

### Step 2: スクリプトの追加

1. Tampermonkeyのダッシュボードを開く
2. 「新しいスクリプトを作成」をクリック
3. 以下のスクリプトをコピー&ペースト

:::details スクリプト

```javascript
// ==UserScript==
// @name         X Ultimate Anti-Spam Blocker
// @namespace    https://example.local
// @version      2.0.0
// @description  Auto-block all types of spam: replies, investment scams, irrelevant content
// @match        https://twitter.com/*
// @match        https://x.com/*
// @run-at       document-idle
// @grant        GM_setValue
// @grant        GM_getValue
// ==/UserScript==

(function () {
  "use strict";

  console.log("🚀 Ultimate Anti-Spam Blocker Loading...");

  try {
    // 基本設定
    const REPLY_SPAM_THRESHOLD = 3; // 連投判定の閾値
    const TIME_WINDOW_MS = 15 * 60 * 1000; // 15分間の監視窓
    const AUTO_BLOCK = true; // 自動ブロック有効
    const DEBUG_MODE = false; // デバッグモード

    // 機能ON/OFF
    const INVESTMENT_SCAM_BLOCK = true; // 投資詐欺検出
    const IRRELEVANT_REPLY_BLOCK = true; // 無関係リプライ検出

    // 投資詐欺検出パターン
    const INVESTMENT_SCAM_PATTERNS = {
      profitClaims: [
        "儲けました",
        "利益",
        "稼げた",
        "稼いだ",
        "儲かった",
        "儲かる",
        "万円儲",
        "億円儲",
        "千万円",
        "百万円",
        "利確",
        "爆益",
        "資産",
        "投資",
        "株",
        "FX",
        "仮想通貨",
        "バイナリー",
        "副業",
        "不労所得",
        "配当",
        "dividend",
        "profit",
        "earn",
        "made money",
        "earned",
        "investment",
        "trading",
        "crypto",
      ],
      moneyPatterns: [
        /\d+万円/g,
        /\d+億円/g,
        /\d+千万円/g,
        /\d+百万円/g,
        /\$\d+[KMB]?/g,
        /\d+[KMB]?\s*(USD|USDT|JPY|円)/g,
        /\d{1,3}(,\d{3})*円/g,
        /\d+\.\d+[KMB]?\s*(万|億)/g,
      ],
      suspiciousMentions: [
        /@[a-zA-Z0-9_]+.*?(儲|利益|稼|投資|株|FX)/,
        /@[a-zA-Z0-9_]+.*?\d+万円/,
        /堀江氏も注目/,
        /推奨銘柄/,
        /ストップ高/,
        /注目銘柄/,
      ],
      scamPhrases: [
        "堀江氏も注目",
        "推奨銘柄が毎日ストップ高",
        "ストップ高級",
        "注目銘柄",
        "急騰銘柄",
        "爆上げ",
        "暴騰",
        "仕手株",
        "情報料",
        "会員限定",
        "限定情報",
        "内部情報",
        "極秘情報",
        "必勝法",
        "確実に",
        "絶対に",
        "100%",
        "保証",
        "DM",
        "DMで",
        "DMください",
        "プロフィール",
        "profile",
        "link in bio",
        "リンク",
        "URL",
        "サイト",
        "site",
      ],
    };

    // 無関係リプライ検出パターン
    const IRRELEVANT_REPLY_PATTERNS = {
      streamingSpam: [
        "stream",
        "streaming",
        "streamm",
        "live",
        "going live",
        "watch me",
        "check out",
        "playing",
        "gaming",
        "game",
        "twitch",
        "youtube",
        "discord",
        "join me",
        "come watch",
        "ライブ",
        "配信",
        "ゲーム",
        "見て",
        "参加",
        "プレイ",
      ],
      cryptoSpam: [
        "crypto",
        "nft",
        "blockchain",
        "defi",
        "web3",
        "metaverse",
        "galxe",
        "leaderboard",
        "quest",
        "airdrop",
        "mint",
        "drop",
        "whitelist",
        "presale",
        "token",
        "coin",
        "stardust",
        "constellation",
        "supernova",
        "galaxy",
        "explorer",
        "暗号通貨",
        "エアドロ",
        "ミント",
        "トークン",
      ],
      promotionPhrases: [
        "check this out",
        "look at this",
        "click here",
        "visit",
        "follow me",
        "subscribe",
        "like and",
        "share this",
        "don't miss",
        "limited time",
        "exclusive",
        "special offer",
        "free",
        "giveaway",
        "contest",
        "winner",
        "prize",
        "チェック",
        "フォロー",
        "見てね",
        "限定",
        "無料",
        "プレゼント",
      ],
      genericPhrases: [
        "good morning",
        "good night",
        "have a great day",
        "jumah mubarak",
        "morning routine",
        "daily routine",
        "wake up",
        "going to bed",
        "see you",
        "catch you later",
        "おはよう",
        "おやすみ",
        "お疲れ様",
        "今日も",
        "毎日",
      ],
    };

    // 内部状態管理
    const replyTracker = new Map();
    const blockedUsers = new Set();
    const processedReplies = new Set();

    function log(...args) {
      if (DEBUG_MODE)
        console.log("[ANTI-SPAM]", new Date().toLocaleTimeString(), ...args);
    }

    // 投資詐欺検出ロジック
    function detectInvestmentScam(text, userHandle) {
      if (!INVESTMENT_SCAM_BLOCK || !text)
        return { isScam: false, score: 0, reasons: [] };

      let scamScore = 0;
      let reasons = [];

      try {
        const lowerText = text.toLowerCase();

        // 儲け話キーワード検出
        const profitMatches = INVESTMENT_SCAM_PATTERNS.profitClaims.filter(
          (keyword) => lowerText.includes(keyword.toLowerCase())
        ).length;
        if (profitMatches > 0) {
          scamScore += profitMatches * 2;
          reasons.push(`儲け話(${profitMatches})`);
        }

        // 金額パターン検出
        let moneyMatches = 0;
        INVESTMENT_SCAM_PATTERNS.moneyPatterns.forEach((pattern) => {
          const matches = text.match(pattern);
          if (matches) moneyMatches += matches.length;
        });
        if (moneyMatches > 0) {
          scamScore += moneyMatches * 3;
          reasons.push(`金額表示(${moneyMatches})`);
        }

        // 怪しいメンション検出
        const mentionMatches =
          INVESTMENT_SCAM_PATTERNS.suspiciousMentions.filter((pattern) =>
            pattern.test(text)
          ).length;
        if (mentionMatches > 0) {
          scamScore += mentionMatches * 4;
          reasons.push(`怪しいメンション(${mentionMatches})`);
        }

        // 詐欺フレーズ検出
        const scamPhraseMatches = INVESTMENT_SCAM_PATTERNS.scamPhrases.filter(
          (phrase) => lowerText.includes(phrase.toLowerCase())
        ).length;
        if (scamPhraseMatches > 0) {
          scamScore += scamPhraseMatches * 3;
          reasons.push(`詐欺フレーズ(${scamPhraseMatches})`);
        }

        // 特定危険パターン
        if (
          text.includes("@") &&
          (text.includes("万円") || text.includes("億円"))
        ) {
          scamScore += 5;
          reasons.push("他人ID+金額");
        }

        if (text.includes("堀江") && text.includes("注目")) {
          scamScore += 6;
          reasons.push("堀江氏詐称");
        }
      } catch (error) {
        log("Error in investment scam detection:", error);
      }

      return {
        isScam: scamScore >= 4,
        score: scamScore,
        reasons: [...new Set(reasons)],
      };
    }

    // 無関係リプライ検出ロジック
    function detectIrrelevantReply(text, userHandle) {
      if (!IRRELEVANT_REPLY_BLOCK || !text)
        return { isIrrelevant: false, score: 0, reasons: [] };

      let irrelevantScore = 0;
      let reasons = [];

      try {
        const lowerText = text.toLowerCase();

        // 配信・ゲーム宣伝検出
        const streamingMatches = IRRELEVANT_REPLY_PATTERNS.streamingSpam.filter(
          (keyword) => lowerText.includes(keyword.toLowerCase())
        ).length;
        if (streamingMatches > 0) {
          irrelevantScore += streamingMatches * 3;
          reasons.push(`配信宣伝(${streamingMatches})`);
        }

        // 暗号通貨・NFT宣伝検出
        const cryptoMatches = IRRELEVANT_REPLY_PATTERNS.cryptoSpam.filter(
          (keyword) => lowerText.includes(keyword.toLowerCase())
        ).length;
        if (cryptoMatches > 0) {
          irrelevantScore += cryptoMatches * 3;
          reasons.push(`暗号通貨宣伝(${cryptoMatches})`);
        }

        // 宣伝・誘導フレーズ検出
        const promotionMatches =
          IRRELEVANT_REPLY_PATTERNS.promotionPhrases.filter((phrase) =>
            lowerText.includes(phrase.toLowerCase())
          ).length;
        if (promotionMatches > 0) {
          irrelevantScore += promotionMatches * 2;
          reasons.push(`宣伝フレーズ(${promotionMatches})`);
        }

        // 無関係な挨拶・定型文検出
        const genericMatches = IRRELEVANT_REPLY_PATTERNS.genericPhrases.filter(
          (phrase) => lowerText.includes(phrase.toLowerCase())
        ).length;
        if (genericMatches > 0 && text.length < 100) {
          irrelevantScore += genericMatches * 2;
          reasons.push(`定型文(${genericMatches})`);
        }

        // 特定パターン検出
        if (
          lowerText.includes("let's stream") ||
          lowerText.includes("streamm")
        ) {
          irrelevantScore += 5;
          reasons.push("配信誘導");
        }

        if (lowerText.includes("galxe") && lowerText.includes("leaderboard")) {
          irrelevantScore += 5;
          reasons.push("Galxe宣伝");
        }
      } catch (error) {
        log("Error in irrelevant reply detection:", error);
      }

      return {
        isIrrelevant: irrelevantScore >= 5,
        score: irrelevantScore,
        reasons: [...new Set(reasons)],
      };
    }

    // DOM要素取得関数
    function getTweetId(el) {
      try {
        const links = el.querySelectorAll('a[href*="/status/"]');
        for (const link of links) {
          const match = link.href.match(/\/status\/(\d+)/);
          if (match) return match[1];
        }
        return null;
      } catch (error) {
        return null;
      }
    }

    function getUserHandle(el) {
      try {
        const selectors = [
          'a[href^="/"][role="link"]:not([href*="/status/"])',
          'a[href^="/"]',
          '[data-testid="User-Name"] a',
        ];

        for (const selector of selectors) {
          const userLink = el.querySelector(selector);
          if (userLink && userLink.href) {
            const href = userLink.getAttribute("href");
            if (href && href.startsWith("/") && !href.includes("/status/")) {
              return href.replace("/", "");
            }
          }
        }
        return null;
      } catch (error) {
        return null;
      }
    }

    // ブロック実行関数
    async function blockUser(userHandle, tweetElement, reason) {
      if (blockedUsers.has(userHandle)) return;

      log(`🚫 BLOCKING: ${userHandle} - ${reason}`);

      try {
        const moreButton = tweetElement.querySelector(
          '[aria-label*="もっと見る"], [aria-label*="More"], [data-testid="caret"]'
        );
        if (!moreButton) {
          showBlockNotification(
            userHandle,
            tweetElement,
            reason + " (UI非表示のみ)"
          );
          return false;
        }

        moreButton.click();
        await new Promise((r) => setTimeout(r, 500));

        const blockButton = Array.from(
          document.querySelectorAll('[role="menuitem"], [role="button"]')
        ).find((item) => {
          const text = item.textContent || "";
          return (
            (text.includes("ブロック") || text.includes("Block")) &&
            !text.includes("ブロック解除") &&
            !text.includes("Unblock")
          );
        });

        if (!blockButton) {
          document.body.click();
          showBlockNotification(
            userHandle,
            tweetElement,
            reason + " (UI非表示のみ)"
          );
          return false;
        }

        blockButton.click();
        await new Promise((r) => setTimeout(r, 500));

        const confirmButton = Array.from(
          document.querySelectorAll('[role="button"]')
        ).find((btn) => {
          const text = btn.textContent || "";
          return (
            (text.includes("ブロック") || text.includes("Block")) &&
            !text.includes("キャンセル") &&
            !text.includes("Cancel")
          );
        });

        if (confirmButton) {
          confirmButton.click();
          blockedUsers.add(userHandle);
          showBlockNotification(userHandle, tweetElement, reason);
          return true;
        }
      } catch (error) {
        log("Error blocking user:", error);
      }

      showBlockNotification(userHandle, tweetElement, reason + " (エラー)");
      return false;
    }

    // 通知表示関数
    function showBlockNotification(userHandle, element, reason) {
      const notification = document.createElement("div");
      notification.innerHTML = `
        <div style="font-weight: bold;">🚫 @${userHandle}</div>
        <div style="font-size: 12px; opacity: 0.8;">${reason}</div>
      `;

      let bgColor = "#e74c3c";
      if (reason.includes("投資詐欺")) bgColor = "#8e44ad";
      if (reason.includes("無関係")) bgColor = "#3498db";
      if (reason.includes("配信")) bgColor = "#9b59b6";
      if (reason.includes("暗号通貨")) bgColor = "#1abc9c";

      notification.style.cssText = `
        position: fixed;
        top: 20px;
        right: 20px;
        background: linear-gradient(135deg, ${bgColor}, ${bgColor}dd);
        color: white;
        padding: 12px 16px;
        border-radius: 8px;
        font-size: 14px;
        z-index: 10000;
        box-shadow: 0 4px 12px rgba(0,0,0,0.3);
        max-width: 320px;
        border-left: 4px solid #fff;
      `;

      document.body.appendChild(notification);

      element.style.cssText += `
        opacity: 0.15 !important;
        filter: grayscale(1) blur(2px) !important;
        border: 2px solid ${bgColor} !important;
        position: relative;
      `;

      const badge = document.createElement("div");
      badge.textContent = "🚫 SPAM";
      badge.style.cssText = `
        position: absolute;
        top: 5px;
        right: 5px;
        background: ${bgColor};
        color: white;
        padding: 4px 8px;
        border-radius: 4px;
        font-size: 12px;
        font-weight: bold;
        z-index: 1000;
      `;
      element.style.position = "relative";
      element.appendChild(badge);

      setTimeout(() => notification.remove(), 5000);
    }

    // 連投追跡関数
    function trackReply(parentTweetId, userHandle, timestamp) {
      if (!replyTracker.has(parentTweetId)) {
        replyTracker.set(parentTweetId, new Map());
      }

      const tweetReplies = replyTracker.get(parentTweetId);
      if (!tweetReplies.has(userHandle)) {
        tweetReplies.set(userHandle, []);
      }

      const userReplies = tweetReplies.get(userHandle);
      userReplies.push(timestamp);

      const cutoff = timestamp - TIME_WINDOW_MS;
      while (userReplies.length && userReplies[0] < cutoff) {
        userReplies.shift();
      }

      return userReplies.length;
    }

    // メイン処理関数
    async function processReply(replyElement) {
      try {
        const replyId = getTweetId(replyElement);
        if (!replyId || processedReplies.has(replyId)) return;

        processedReplies.add(replyId);

        const userHandle = getUserHandle(replyElement);
        if (!userHandle || blockedUsers.has(userHandle)) return;

        const tweetText = replyElement.textContent || "";

        // 1. 投資詐欺検出（最優先）
        const investmentScamDetection = detectInvestmentScam(
          tweetText,
          userHandle
        );
        if (investmentScamDetection.isScam) {
          const scamReason = `投資詐欺検出 (score: ${investmentScamDetection.score}, 理由: ${investmentScamDetection.reasons.join(", ")})`;
          await blockUser(userHandle, replyElement, scamReason);
          return;
        }

        // 2. 無関係リプライ検出
        const irrelevantDetection = detectIrrelevantReply(
          tweetText,
          userHandle
        );
        if (irrelevantDetection.isIrrelevant) {
          const irrelevantReason = `無関係リプライ検出 (score: ${irrelevantDetection.score}, 理由: ${irrelevantDetection.reasons.join(", ")})`;
          await blockUser(userHandle, replyElement, irrelevantReason);
          return;
        }

        // 3. 通常の連投検出
        const timestamp = Date.now();
        const replyCount = trackReply("general", userHandle, timestamp);

        if (replyCount >= REPLY_SPAM_THRESHOLD && AUTO_BLOCK) {
          const reason = `連投検出 (${replyCount}回)`;
          await blockUser(userHandle, replyElement, reason);
        }
      } catch (error) {
        log("Error processing reply:", error);
      }
    }

    // DOM監視設定
    const observer = new MutationObserver((mutations) => {
      for (const mutation of mutations) {
        for (const node of mutation.addedNodes) {
          if (!(node instanceof HTMLElement)) continue;

          const tweets = node.matches?.('article[data-testid="tweet"]')
            ? [node]
            : node.querySelectorAll?.('article[data-testid="tweet"]') || [];

          tweets.forEach((tweet, index) => {
            setTimeout(() => {
              processReply(tweet).catch((err) =>
                log("Error processing tweet:", err)
              );
            }, 100 * index);
          });
        }
      }
    });

    observer.observe(document.documentElement, {
      childList: true,
      subtree: true,
    });

    // 初期ロード時の処理
    setTimeout(() => {
      document
        .querySelectorAll('article[data-testid="tweet"]')
        .forEach((tweet, index) => {
          setTimeout(() => {
            processReply(tweet).catch((err) =>
              log("Error processing initial tweet:", err)
            );
          }, 150 * index);
        });
    }, 2000);

    console.log("✅ Ultimate Anti-Spam Blocker fully loaded!");
  } catch (error) {
    console.error("❌ Fatal error loading script:", error);
  }
})();
```

:::

### Step 3: 動作確認

1. Ctrl+Sでスクリプトを保存
2. X（Twitter）をリロード
3. F12 → Consoleで「✅ Ultimate Anti-Spam Blocker fully loaded!」が表示されることを確認

## カスタマイズ

### 検出感度の調整

```javascript
// より厳しく検出したい場合
const REPLY_SPAM_THRESHOLD = 2; // 2回で連投判定

// より緩く検出したい場合
const REPLY_SPAM_THRESHOLD = 5; // 5回で連投判定
```

### 機能の個別ON/OFF

```javascript
const INVESTMENT_SCAM_BLOCK = false; // 投資詐欺検出を無効化
const IRRELEVANT_REPLY_BLOCK = false; // 無関係リプライ検出を無効化
```

### デバッグモード

```javascript
const DEBUG_MODE = true; // 詳細ログを出力
```

## 注意事項

### 誤検出の可能性

パターンマッチングによる検出のため、稀に正常なユーザーを誤検出する可能性があります。気になる場合は検出感度を下げてください。

### X(Twitter)の仕様変更

X(Twitter)のDOM構造が変更された場合、スクリプトが動作しなくなる可能性があります。

### 利用は自己責任で

このスクリプトの使用により生じた問題について、作者は一切の責任を負いません。

## 効果測定

実際に1週間使用した結果：

- ブロックしたアカウント数: 約150件
- 投資詐欺検出: 約60件（40%）
- 無関係リプライ検出: 約50件（33%）
- 連投スパム検出: 約40件（27%）

タイムラインの快適性が劇的に向上しました。

## まとめ

インプレゾンビとの戦いは終わりません。しかし、このスクリプトがあれば、少なくとも自分のタイムラインを平和に保つことができます。

X(Twitter)をより快適に使いたい方は、ぜひ試してみてください。ただし、**使用は自己責任**でお願いします。

## 最後に

スクリプトはAIに提案したうえで構築し、機能確認は私が行いました。
