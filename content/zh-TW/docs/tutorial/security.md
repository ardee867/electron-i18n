# 安全性、原生功能及你的責任

身為網頁開發人員，我們常常受惠於瀏覽器建立的強大安全網，我們的程式再怎麼搞，所能引起的風險都微乎其微。 我們的網站被限制在獨立的沙盒環境中運作，我們相信使用者都習慣於享受由龐大工程師團隊開發維護的瀏覽器，能在第一時間快速處理新發現的安全性威脅。

而在使用 Electron 時，千萬要記住一點: Electron 並不是網頁瀏覽器。 它讓你能用熟悉的網頁技術打造出功能完善的桌面應用程式，只不過你的程式碼具有更大的能力。 JavaScript 能存取檔案系統，使用者 Shell 等等。 你能做出高品質的原生應用程式，但是你的程式碼被賦與的能力越強，相對的安全性問題也會越重。

請謹記在心，顯示不能完全信任的來源產生的任意內容會有極大的安全風險，Electron 也沒打算處理。 事實上，流行的 Electron 應用程式 (Atom, Slack, Visual Studio Code 等) 幾乎都只顯示本機的內容 (或受信任、安全且沒跟 Node 整合的遠端內容) ，如果你的應用程式會執行網路上抓來的程式碼，你要負責確保那些程式碼中不會有惡意內容。

## 回報安全性問題

如何正確揭露 Electron 漏洞的相關資訊可參考 [SECURITY.md](https://github.com/electron/electron/tree/master/SECURITY.md)

## Chromium 安全性議題及升級

雖然 Electron 致力於盡快支援新版的 Chromium，但開發人員都應該知道，升級是件嚴肅的事，可能要手動修改數十，甚至上百個檔案。 以現有的資源及貢獻程度來看，Electron 通常沒辦法跟到最新版的 Chromium，可能會落後數日或數週。

我們認為現在的 Chromium 元件升級機制，已經在有限資源及多數桌面應用程式需求之間取得平衡。 我們真的很想聽到大家又把 Electron 應用在什麼特別的地方。 隨時歡迎各位提出 Pull Request 或是其他貢獻。

## 先不管上面那些建議

由遠端取得程式碼並在本機執行就會有安全議題。 一起想像一下，有個遠端網站的內容將在瀏覽器視窗裡顯示。 如果攻擊者有辦法改變上述內容 (可能是直接攻擊來源，或是藏在你的應用程式與實際目的地中間)，他們就有機會在使用者的機器上執行原生程式。

> :warning: 無論如何，你都不該在啟用 Node 整合的情況下，由遠端載入並執行程式碼。 如果需要執行 Node 程式碼，請只用本機檔案 (跟你的應用程式打包在一起的那些)。 如果要顯示遠端內容，請用 `webview` 標籤，並確定已經停掉 `nodeIntegration`。

#### 檢查清單

不保證就能金槍不入，但至少你應該要:

* 只顯示安全連線 (https) 的內容
* 在所有會顯示遠端內容的畫面轉譯程式中停用 Node 整合功能 (將 `webPreferences` 中的 `nodeIntegration` 設為 `false`)
* 在所有會顯示遠端內容的畫面轉譯器中啟用內容隔離功能 (將 `webPreferences` 中的 `contextIsolation` 設為 `true`)
* 在所有會載入遠端內容的 Session 中使用 `ses.setPermissionRequestHandler()`
* 不要停用 `webSecurity`。否則同源政策也會同時被停用。
* 定義 [`Content-Security-Policy`](http://www.html5rocks.com/en/tutorials/security/content-security-policy/)，並使用限制性規則 (例如 `script-src 'self'`)
* [覆寫並停用 `eval`](https://github.com/nylas/N1/blob/0abc5d5defcdb057120d726b271933425b75b415/static/index.js#L6-L8)，否則可以透過它將字串視為程式碼執行。
* 不要將 `allowRunningInsecureContent` 設為 true。
* 不要啟用 `experimentalFeatures` 或 `experimentalCanvasFeatures`，除非你很清楚自己在幹嘛。
* 別用 `blinkFeatures`，除非你很清楚自己在幹嘛。
* WebViews: 不要加 `nodeintegration` 屬性。
* WebViews: 不要用 `disablewebsecurity`。
* WebViews: 不要用 `allowpopups`。
* WebViews: 不要在遠端 CSS/JS 上用 `insertCSS` 或 `executeJavaScript`。
* WebViews: 透過 `will-attach-webview` 事件連結任何 `<webview>` 標籤時，驗證所有的選項及參數:

```js
app.on('web-contents-created', (event, contents) => {
  contents.on('will-attach-webview', (event, webPreferences, params) => {
    // 拿掉用不著的預載腳本，或是確認它們的位置是安全正確的
    delete webPreferences.preload
    delete webPreferences.preloadURL

    // 停用 Node 整合
    webPreferences.nodeIntegration = false

    // 驗證將要載入的 URL
    if (!params.src.startsWith('https://yourapp.com/')) {
      event.preventDefault()
    }
  })
})
```

再次強調，這份清單只能幫你降低風險，並沒辦法完全將風險排除。如果你的目標只是要顯示網站，那麼瀏覽器會是比較安全的選項。