# hiwalay

> Gamitin ang mga modyul ng pangunahing proseso mula sa proseso ng tagabigay.

Mga proseso: [Renderer](../glossary.md#renderer-process)

Ang modyul ng `remote` ay nagbibigay ng isang simpleng paraan na gumawa ng maki-prosesong komyunikasyon (IPC) sa pagitan ng proaesong tagabigay (pahina ng web) at ng pangunahing proseso.

Sa Electron, ang mga modyul na may kaugnayan sa GUI (tulad ng `dialog`, `menu` atbp.) ay makukuha lamang sa pangunahing proseso, hindi sa prosesong tagabigay. Sa ayos ng paggamit sa kanila mula sa prosesong tagabigay, ang modyul ng `ipc` ay kinakailangan na magpadala ng mga maki-prosesong mensahe sa pangunahing proseso. Kasama ang modyul ng `remote`, maaari mong hingiin ang mga pamamaraan ng mga bagay sa pangunahing proseso nang walang nakakapansin na nakapagpadala ng maki-prosesong mensahe, katulad ng [RMI](http://en.wikipedia.org/wiki/Java_remote_method_invocation) ng Java. Ang isang halimbawa ng paglikha ng isang browser window mula sa isang prosesong tagabigay:

```javascript
const {BrowserWindow} = kailangan('electron').remote
let win = new BrowserWindow({width: 800, height: 600})
win.loadURL('https://github.com')
```

**Note:** Sa kabaligtaran (i-akses ang prosesong tagabigay mula sa pangunahing proseso) maaari mong gamitin ang [webContents.executeJavascript](web-contents.md#contentsexecutejavascriptcode-usergesture-callback).

## Mga bagay ng Remote

Bawat bagay (kasama ang mga punsyon) ay nagbabalik dahil ang modyul ng `remote` ay kumakatawan sa isang bagay sa pangunahing proseso (tinatawag natin itong malayong bagay o malayong punsyon). Kapag iyong hiningi ang mga pamamaraan ng isang remote na bagay, tawagin ang isang remote na punsyon, o gumawa ng isang bagong bagay kasama ang remote na tagagawa (punsyon), ikaw ay talagang nagpapadala ng sabay-sabay na mga maki-prosesong mensahe.

Sa mga halimbawa sa itaas, kapuwa ang `BrowserWindow` at ang `win` ay mga remote na bagay at ang `new BrowserWindow` ay hindi gumawa ng isang bagay sa `BrowserWindow` sa prosesong tagabigay. Sa halip, ito ay gumawa ng isang bagay ng `BrowserWindow` sa pangunahing proseso at ibinalik ang nararapat na remote na bagay sa prosesong tagabigay, kagaya ng bagay sa `win`.

**Note:** Ang [enurable properties](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties) lamang na kung saan ay naroroon nang ang remote na bagay ay unang isinangguni na mapupuntahan sa pamamagitan ng remote.

**Note:** Ang mga hanay at mga Buffer ay kinopya sa ibabaw ng IPC kapag na-access sa pamamagitan ng modyul ng `remote`. Ang pagbabago sa kanila sa prosesong tagabigay ay hindi magpapabago sa kanila sa pangunahing proseso at kahit sa kabaligtaran.

## Ang tagal ng buhay ng mga Remote na Bagay

Sinisigurado ng Electron na habang ang remote na bagay ay nandoon pa sa prosesong tagabigay (sa ibang salita, ay hindi pa ibinabasura), ang nararapat na bagay sa pangunahing proseso ay hindi mailalabas. Kapag ang remote na bagay ay naibasura na, ang nararapat na bagay sa pangunahing proseso ay hindi isasangguni.

Kung ang remote na bagay ay nakalabas sa prosesong tagabigay (hal. itinago sa isang balangkas ngunit hindi nailabas), ang nararapat na bagay sa pangunahing proseso ay makakalabas din, kaya kailangan mo ng matinding pag-iingat na hindi makalabas ang mga remote na bagay.

Ang mga uri ng pangunahing halaga tulad ng mga string at mga numero, gayunpaman, ay ipinapadala sa pamamagitan ng pagkopya.

## Ipinapasa ang gantingtawag patungo sa pangunahing proseso

Ang kodigo sa pangunahing proaeso ay maaaring tanggapin ang mga gantingtawag mula sa tagabigay - sa isang pagkakataon ang modyul ng `remote` - ngunit kailangan mo ng matinding pag-iingat kapag ginagamit ang katangian na ito.

Una, upang maiwasan ang mga dedlak, ang mga gantingtawag na naipadala sa pangunahing proseso ay tatawagin ng magkakahiwalay. Hindi mo dapat asahan ang pangunahing proseso na kuhanin ang naibalik na halaga ng naipasang mga gantingtawag.

Sa isang pagkakataon hindi mo magagamit ang isang punsyon mula sa prosesong tagabigay sa isang `Array.map` na tinawag sa pangunahing proseso:

```javascript
// main process mapNumbers.js
exports.withRendererCallback = (mapper) => {
  return [1, 2, 3].map(mapper)
}

exports.withLocalCallback = () => {
  return [1, 2, 3].map(x => x + 1)
}
```

```javascript
// proseso ng tagabigay
const mapNumbers = kailangan('electron').remote.require('./mapNumbers')
const withRendererCb = mapNumbers.withRendererCallback(x => x + 1)
const withLocalCb = mapNumbers.withLocalCallback()

console.log(withRendererCb, withLocalCb)
// [undefined, undefined, undefined], [2, 3, 4]
```

Kagaya ng iyong nakikita, ang magkakasabay na nagbalik na halaga ng gantingtawag ng tagabigay ay hindi inaasahan, at hindi tumutugma ang nagbalik na halaga ng isang kilalang gantingtawag na naroroon sa pangunahing proseso.

Pangalawa, ang mga gantingtawag na ipinasa sa pangunahing proseso ay mananatili hanggang sila ay ibasura ng pangunahing proseso.

Halimbawa, ang mga sumusunod na kodigo ay mukhang inosente sa unang tingin. Nag-install ito ng isang gantingtawag para sa event ng `close` sa isang remote na bahay:

```javascript
require('electron').remote.getCurrentWindow().on('close', () => {
  // window was closed...
})
```

But remember the callback is referenced by the main process until you explicitly uninstall it. If you do not, each time you reload your window the callback will be installed again, leaking one callback for each restart.

To make things worse, since the context of previously installed callbacks has been released, exceptions will be raised in the main process when the `close` event is emitted.

Upang maiwasan ang problema, siguraduhin burahin anumang kaugnayan sa mga binalikang tawag na ipinasa sa pangunahing proseso. This involves cleaning up event handlers, or ensuring the main process is explicitly told to deference callbacks that came from a renderer process that is exiting.

## Accessing built-in modules in the main process

The built-in modules in the main process are added as getters in the `remote` module, so you can use them directly like the `electron` module.

```javascript
const app = require('electron').remote.app
console.log(app)
```

## Pamamaraan

The `remote` module has the following methods:

### `remote.require(modyul)`

* `modyul` Lubid

Returns `any` - The object returned by `require(module)` in the main process. Modules specified by their relative path will resolve relative to the entrypoint of the main process.

halimbawa.

    proyekto/
    ├── pangunahing
    │   ├── foo.js
    │   └── index.js
    ├── package.json
    └── tagabigay
        └── index.js
    

```js
// main process: main/index.js
const {app} = require('electron')
app.on('ready', () => { /* ... */ })
```

```js
// some relative module: main/foo.js
module.exports = 'bar'
```

```js
// renderer process: renderer/index.js
const foo = require('electron').remote.require('./foo') // bar
```

### `remote.getCurrentWindow()`

Returns [`BrowserWindow`](browser-window.md) - The window to which this web page belongs.

### `remote.getCurrentWebContents()`

Returns [`WebContents`](web-contents.md) - The web contents of this web page.

### `remote.getGlobal(name)`

* `name` String

Returns `any` - The global variable of `name` (e.g. `global[name]`) in the main process.

## Mga Katangian

### `remote.process`

The `process` object in the main process. This is the same as `remote.getGlobal('process')` but is cached.