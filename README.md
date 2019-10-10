# Upscope's Javascript co-browsing API
[Upscope](https://upscope.io/) is a co browsing service that allows you to build screen sharing into your website. Once installed, your support agents will be able to instantly see the user's browser without them having to do anything.

The [Upscope](https://upscope.io/) javascript API allows you to customize the way Upscope behaves on different pages.

Once Upscope is installed on a webpage, the `window.Upscope` object will be available globally. You can call the `Upscope` function to interact with our API. All calls will be asyncronous and if you expect a return value will require a callback.

## TOC
- [`Upscope('init', {...});`](#upscopeinit-): Initiate Upscope and set configuration
- [`Upscope('updateConnection', {...});`](#upscopeupdateconnection-): Update user info, log in or log out user in SPA
- [`Upscope('stopSession');`](#upscopestopsession): Stop screensharing
- [`Upscope('getUserId', cb);`](#upscopegetuserid-cb): Get the Upscope User ID
- [`Upscope('getWatchLink', cb);`](#upscopegetwatchlink-cb): Get the Upscope watch URL
- [`Upscope('getConnectionId', cb);`](#upscopegetconnectionid-cb): Get the Upscope connection id
- [`Upscope('getLookupCode', cb);`](#upscopegetlookupcode-cb): Get the Upscope lookup code
- [`Upscope('reset');`](#upscopereset): Resets the connection to Upscope
- [`Upscope('requestAgent');`](#upscoperequestagent): Requests an agent's help
- [`Upscope('cancelAgentRequest');`](#upscopecancelagentrequest): Cancels the request for agent help
- [`Upscope('on', [...events], cb);`](#upscopeon-events-cb): Listen for Upscope events

## `Upscope('init', {...});`
`Upscope('init')` is used to initialize the Upscope code. An optional second argument is passed with configuration attributes.

| Option | Default value | Description |
| ------ | ------------- | ----------- |
| `apiKey` | (Automatically set to your API key) | The API key of the account to connect to |
| `identities` | `undefined` | A list of strings to identify the user by (e.g. `['Joe Smith', 'joe@smith.com']`). If set to `null`, the identity info is cleared. If not set, nothing is changed. With [Intercom](https://intercom.com) live chat this is set automatically if left empty. |
| `uniqueId` | `undefined` | A strings you uniquely identify the user by (e.g. `123`). If set to `null`, the id is cleared. If not set, nothing is changed. With [Intercom](https://intercom.com) live chat this is set automatically if left empty. |
| `tags` | `undefined` | A list of strings to tag the user with (e.g. `['#visitor', '#high-value']`). Tags can only be alphanumeric characters, and can contain no spaces. If set to `null`, the identity info is cleared. If not set, nothing is changed. |
| `requireAuthorizationForSession` | (Set through the admin interface) | Whether to ask for user authorization before screen sharing |
| `authorizationPromptTitle` | (Set through the admin interface) | The title to display on the authorization prompt |
| `authorizationPromptMessage` | (Set through the admin interface) | The text to display on the authorization prompt |
| `endOfScreenshareMessage` | (Set through the admin interface) | If set to a string, it will `alert()` the string when a screen sharing session ends |
| `callWaitMessage` | (Set through the admin interface) | This text will be shown to the user when they follow a call link |
| `integrateWithLivechat` | `true` | Whether to integrate automatically with live chat systems |
| `showUpscopeLink` | (Set through the admin interface) | Whether to show a "Screen sharing by Upscope" link at the bottom left of the page while screen sharing |
| `useLightPointer` | `false` | Whether use an inverted color version of the pointer for darker interfaces |
| `autoconnect` | `true` | Whether to connect to Upscope server automatically |
| `trackConsole` | (Set through the admin interface) | Whether to track console content to display agents |
| `allowRemoteConsole` | (Set through the admin interface) | Whether to allow agents (who have the right permissions) to execute remote console commands |
| `allowRemoteClick` | (Set through the admin interface) | Whether to allow agents to click for the user |
| `allowRemoteScroll` | (Set through the admin interface) | Whether to allow agents to scroll for the user |
| `allowRemoteType` | (Set through the admin interface) | Whether to allow agents to type for the user |
| `crossStorageEndpoint` | `https://storage.upscope.io/` | Instance of [cross-storage](https://github.com/zendesk/cross-storage) to use to save account data |
| `lookupCodeElement` | (Set through the admin interface) | CSS selector or HTML element object to replace text of with 4 digit lookup code |
| `injectLookupCodeButton` | (Set through the admin interface) | Whether to inject a button in the lower left of the page to show the 4 digit lookup code |
| `disconnectAfterSeconds` | `600` | Number of seconds of inactivity after which Upscope disconnects from the server |
| `proxyAssets` | (Set through the admin interface) | List of wildcard strings (e.g. `['*://localhost:*/*']`) to proxy when screen sharing. This is useful to allow screen sharing in development or staging environments |
| `maskedElements` | (Set through the admin interface) | List of CSS selectors (e.g. `['.credit-card']`) to mask when screen sharing in addition to elements with a `no-upscope` CSS class. |
| `domChangesDelay` | `50` | Refresh rate of the page. Set to 50 so changes are shown right away, but can be higher on websites where a lot changes constantly to avoid the agent's browser slowing down |
| `debug` | `sessionStorage.getItem('__upscope:debug') === 'on'` | Whether to turn debugging on or off |
| `enableCanvases` | `true` | Whether to show canvases while screen sharing |
| `canvasEncoding` | `png` | Whether canvases should be encoded with `jpeg` (fast but low quality) or `png` (fast but larger images, include transparency) |

## Init functions
In addition to the above configuration, the `init` code contains functions used to change the functionality of how Upscope behaves.

### `onSessionRequest(cb, agentName)`
`onSessionRequest(cb)` is called when `requireAuthorizationForSession` to decide whether the user wants to allow screen sharing. A callback will be passed and will need to be called with a boolean representing whether authorization was granted.  **This is the default function:**
```js
function (cb) {
  console.info('Upscope.io: Screen sharing request received');
  cb(confirm(configuration.authorizationPromptMessage));
}
```
*Please note that the `configuration` object will not be available to your function, but can be retrieved with `window.Upscope._config`.*

## `Upscope('updateConnection', {...});`
`Upscope('updateConnection', {...});` is used to update the authentication information of the user. It is used like `Upscope('init', {...});`, but can only accept the following properties: `identities`, `uniqueId`, and `tags`.

### Examples
```js
// Logging in a user in a SPA
Upscope('updateConnection', {
  identities: ['Joe Smith'],
  uniqueId: 1323
});

// Signing out a user in a SPA
Upscope('updateConnection', {
  identities: null,
  uniqueId: null
});
```

## `Upscope('connect');`
`Upscope('connect');` is used to initiate a connection to the Upscope server if `autoconnect` is set to `false`.

## `Upscope('stopSession');`
`Upscope('stopSession');` is used to stop a currently active screen sharing session.

## `Upscope('startDebug');`
`Upscope('startDebug');` is used to start debugging.

## `Upscope('stopDebug');`
`Upscope('stopDebug');` is used to stop debugging.

## `Upscope('getUserId', cb);`
`Upscope('getUserId', cb);` is used to retrieve the Upscope user ID for the user.

### Example
```js
Upscope('getUserId', function (userId) {
  alert('To see screen go to https://upscope.io/w/' + userId);
});
```

## `Upscope('getWatchLink', cb);`
`Upscope('getWatchLink', cb);` is used to retrieve the Upscope link to see the user screen.

### Example
```js
Upscope('getWatchLink', function (link) {
  alert('To see screen go to ' + link);
});
```

## `Upscope('getConnectionId', cb);`
`Upscope('getConnectionId', cb);` is used to retrieve the Upscope connection ID of the current tab.

## `Upscope('getLookupCode', cb);`
`Upscope('getLookupCode', cb);` is used to retrieve the Upscope lookup code for the user.

## `Upscope('requestAgent');`
`Upscope('requestAgent');` is used to initiate a request for agent help.

## `Upscope('cancelRequestAgent');`
`Upscope('cancelRequestAgent');` is used to cancel a request for agent help.

### Example
```html
<p><a href="#" id="upscopeLookupCode">Click here</a> for your screen sharing code</p>
<script>
  document
    .getElementById('upscopeLookupCode')
    .addEventListener('click',
      function(evt) {
        evt.preventDefault();
        Upscope('getLookupCode', function(code) {
          alert("Please tell our agent code " + code);
        })
      });
</script>
```

## `Upscope('reset');`
`Upscope('reset');` is used to reset the Upscope connection. It will change the ID of the visitor.

## `Upscope('on', [...events], cb);`
`Upscope('on', [...events], cb);` is used to listen for Upscope events.

### Events
| Event name | Description |
| ---------- | ----------- |
| `connection` | Triggered when Upscope connects to server |
| `connectionReset` | Triggered when the connection to Upscope is reset |
| `sessionRequest` | Triggered when a request for screen sharing is received **(Use `Upscope('init', {...});` to specify a different authorization behavior).** |
| `sessionStart` | Triggered when screen sharing starts |
| `sessionContinue` | Triggered when screen sharing continues after the user navigates to a different page |
| `sessionEnd` | Triggered when screen sharing ends |
| `waitingForCall` | Triggered when we detect the visitor used a call link and is waiting for the agent |
| `callAccepted` | Triggered when the agent has responded to the call and the session is to begin shortly |
| `agentRequestUpdate` | Triggered when the visitor is requesting an agent or the status of the request is updated (a status string will be provided in the callback) |
| `agentsAvailable` | Triggered when we agents are online on Upscope |
| `callStart` | Triggered when the agent initiates an audio call |
| `callEnd` | Triggered when the audio call is terminated |

### Example
```html
<div id="upscope-control" style="display: none;">
  Screen sharing active.
  <a href="#" id="upscope-control-stop">Stop now.</a>
</div>
<script>
var controlElement = document.getElementById('upscope-control');
var stopElement = document.getElementById('upscope-control-stop');

Upscope('on', 'sessionStart', 'sessionContinue', function() {
  controlElement.style.display = 'block';
});
Upscope('on', 'sessionEnd', function() {
  controlElement.style.display = 'none';
});

stopElement.addEventListener('click',
  function(evt) {
    evt.preventDefault();
    Upscope('stopSession');
  });
</script>
```
