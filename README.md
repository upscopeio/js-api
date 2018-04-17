# js-api
Api documentation for Upscope.io

The [Upscope](https://upscope.io/) javascript API allows you to customize the way Upscope behaves on different pages.

## Functions
Once Upscope is installed on a webpage, the `window.Upscope` object will be available globally. You can call the `Upscope` function to interact with our API. All calls will be asyncronous and if you expect a return value will require a callback.

### `Upscope('init', {...});`
`Upscope('init')` is used to initialize the Upscope code. An optional second argument is passed with configuration attributes.

| Option | Default value | Description |
| ------ | ------------- | ----------- |
| `apiKey` | (Automatically set to your API key) | The API key of the account to connect to |
| `requireAuthorizationForSession` | (Set through the admin interface) | Whether to ask for user authorization before screensharing |
| `integrateWithLivechat` | `true` | Whether to integrate automatically with live chat systems |
| `sendScreenshotOnChatOpen` | (Set through the admin interface) | Whether to take a screenshot when use opens up livechat |
| `sendScreenshotOnSessionStart` | (Set through the admin interface) | Whether to take a screenshot when screensharing begins |
| `trackLiveChatEvents` | `true` | Whether to track when user opens live chat window |
| `trackEvents` | `['pageLoads', 'screenShares']` | Events to track automatically. Allowed values are `pageLoads`, `clicks`, `screenShares`. |
| `showUpscopeLink` | (Set through the admin interface) | Whether to show a "Screensharing by Upscope" link at the bottom left of the page while screen sharing |
| `useLightPointer` | `false` | Whether use an inverted color version of the pointer for darker interfaces |
| `autoconnect` | `true` | Whether to connect to Upscope server automatically |
| `trackConsole` | (Set through the admin interface) | Whether to track console content to display agents |
| `allowRemoteConsole` | (Set through the admin interface) | Whether to allow agents (who have the right permissions) to execute remote console commands |
| `allowRemoteClick` | (Set through the admin interface) | Whether to allow agents to click for the user |
| `allowRemoteScroll` | (Set through the admin interface) | Whether to allow agents to scroll for the user |
| `allowRemoteType` | (Set through the admin interface) | Whether to allow agents to type for the user |
| `crossStorageEndpoint` | `https://storage.upscope.io/` | Instance of [cross-storage](https://github.com/zendesk/cross-storage) to use to save account data |
| `lookupCodeElement` | `null` | CSS selector or HTML element object to replace text of with 4 digit lookup code |
| `injectLookupCodeButton` | (Set through the admin interface) | Whether to inject a button in the lower left of the page to show the 4 digit lookup code |
| `disconnectAfterSeconds` | `600` | Number of seconds of inactivity after which Upscope disconnects from the server |
| `proxyAssets` | (Set through the admin interface) | List of wildcard strings (e.g. `['*://localhost:*/*']`) to proxy when screensharing. This is useful to allow screensharing in development or staging environments |
| `maskedFields` | (Set through the admin interface) | List of CSS selectors (e.g. `['.credit-card']`) to mask when screensharing in addition to elements with a `no-upscope` CSS class. |
| `domChangesDelay` | `50` | Refresh rate of the page. Set to 50 so changes are shown right away, but can be higher on websites where a lot changes constantly to avoid the agent's browser slowing down |
| `debug` | `sessionStorage.getItem('__upscope:debug') === 'on'` | Whether to turn debugging on or off |
| `enableCanvases` | `true` | Whether to show canvases while screensharing |
| `canvasEncoding` | `jpeg` | Whether canvases should be encoded with `jpeg` (fast but low quality) or `png` (fast but larger images, include transparency) |

### Init functions
In addition to the above configuration, the `init` code contains functions used to change the functionality of how Upscope behaves.

#### `onSessionRequest(cb)`
`onSessionRequest(cb)` is called when `requireAuthorizationForSession` to decide whether the user wants to allow screensharing. A callback will be passed and will need to be called with a boolean representing whether authorization was granted.  **This is the default function:**
```js
function (cb) {
  console.info('Upscope.io: Screen sharing request received');
  cb(confirm('Would you like to let our agent see your screen?'));
}
```

#### `formValueMiddleware(element)`
`formValueMiddleware(element)` is called to retrieve the value of an input. **This is the default function:**
```js
function (element) {
  var value = element.value,
    maskedValue = value.replace(/./g, '*');
  if (element.type === 'password' ||
      (typeof element.getAttribute === 'function' && element.getAttribute('autocompletetype') === 'cc-number'))
    return maskedValue;

  for (let maskedField of configuration.maskedFields) {
    if (typeof element.matches === 'function' && element.matches(maskedField)) return maskedValue;
    else if (typeof element.msMatchesSelector === 'function' && element.msMatchesSelector(maskedField)) return maskedValue;
  }

  while (element) {
    if (element.className && /\bno-upscope\b/.test(element.className))
      return maskedValue;

    element = element.parentNode;
  }
  return value;
}
```
*Please note that the `configuration` object will not be available to your function, but can be retrieved with `window.Upscope._config`.*

#### `iframeMiddleware(element)`
`formValueMiddleware(element)` is called to decide whether to send an iframe's contents through Upscope. **This is the default function:**
```js
function (element) {
  while (element) {
    if (element.className && /\bno-upscope\b/.test(element.className))
      return false;

    element = element.parentNode;
  }
  return true;
}
```
#### `setLookupCode()`
`setLookupCode()` is called to generate the quick lookup code. **This is the default function:**
```js
function () {
  var code = '';
  for (let x = 0; x < 4; x++)
    code += String(Math.floor(Math.random() * 10));
  return code;
}
```
