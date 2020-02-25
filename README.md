# Seatmap integration and communication
This document describes how to integrate seatmap HTML page (further "seatmap") into any website, Android or iOS mobile app. Plus, communication between the seatmap and a parent layer that embeds seatmap (further just "parent layer").

![](https://github.com/Kwiket/jets-vector-setmap-integration/blob/master/.wiki-resources/funny-css-bug.jpg)
>_They say seats with letter F are CSS bug. I say it's a smoking area_ 😜

&nbsp;
## Table of contents
* [Website integration](#website-integration)
* [Android integration](#android-integration)
* [iOS integration](#ios-integration)
* [i18n internationalization](#i18n-internationalization)
* [Color themes](#color-themes)
* [Available communication message types](#available-communication-message-types)
* [Initialisation and workflow](#initialisation-and-workflow)


&nbsp;
## Integration
This section explains how to integrate seatmap into a website, Android and iOS mobile apps.


&nbsp;
### <a name="website-integration"></a>Website integration

Embed seatmap into HTML page via `<iframe>`:
```typescript
const iframe = document.createElement('iframe');

iframe.addAttribute('frameborder', '0');
iframe.addAttribute('marginheight', '0');
iframe.addAttribute('marginwidth', '0');
iframe.addAttribute('scrolling', 'yes');
iframe.id = 'seatmap';
iframe.addEventListener('load', handleSeatmapLoad, false);
iframe.src = 'http://demo-seatmap.quicket.me/htmls/dev/htmls/1764.html';
document.body.appendChild(iframe);
```

>⚠️&nbsp; It's important to listen to `load` event, ensuring iframe contents is loaded before publishing messages to it.

&nbsp;
When iframe contents is loaded, **publish [message](#available-communication-message-types) down to the embed seatmap** via `postMessage()` function:
```typescript
function handleSeatmapLoad(e: BrowserEvent) {
  const targetIframe = e.currentTarget as HTMLIframeElement;
  const targetIframeWindow = targetIframe.contentWindow;
  const message = { // Shall match `IMessage` standard interface.
    type: 'SYNC_PASSENGERS',
    data: {
      flightGuid: '1ABC',
      passengers: [
        {
          id: '1',
          seatPreference: null,
          seat: null
        }
      ]
    }
  };

  targetIframeWindow.postMessage(JSON.stringify(message), '*');
}
```

&nbsp;
**Subscribe to [message](#available-communication-message-types) emitted by the seatmap:**
```typescript
function handleMessage(e: MessageEvent): void {
  const message = JSON.parse(e.data);

  // `message`, matches `IMessage` standard interface with data received from the seatmap.
}

window.addEventListener('message', handleMessage, false);
```






&nbsp;
### <a name="android-integration"></a>Android integration
Seatmap shall be rendered using WebView.

```java
WebView webview = new WebView(this);

setContentView(webview);
webview.loadUrl("1764.html?<b>platform=android</b>");
```
>⚠️&nbsp; It's important and required to include `?platform=android` GET parameter into loadable url.

&nbsp;
When WebView contents is loaded, it's possible to publish messages to the seatmap and receive messages back from it.

&nbsp;
**Publish [message](#available-communication-message-types) into the seatmap by executing public JavaScript function within WebView** — `web.sendMessage(message: string)`. Seatmap has this function available for you.

```javascript
webView.evaluateJavascript("web.sendMessage(\'{\"type\":\"SYNC_PASSENGERS\",\"data\":{\"flightGuid\":\"1ABC\",\"passengers\":[{\"id\":\"1\",\"seatPreference\":null,\"seat\":null}]}}\');
```

&nbsp;
**To receive [message](#available-communication-message-types) from the seatmap add JavaScript interface into the seatmap.** Normally, `webView.addJavascriptInterface()` native method can be used to inject and map Java interface inside JavaScript source code in a WebView. Interface shall contain `handleSeatmapEvent()` public method. Make sure the name used to expose the object in JavaScript is  `android`.

Here's approximately how to add JavaScript interface into WebView:

```java
import org.json.JSONObject;
…

class JsObject {
  @JavascriptInterface
  public void handleSeatmapEvent(String: serializedMessage) {
    JSONObject message = new JSONObject(serializedMessage);

    // `message`, matches `IMessage` standard interface with data received from the seatmap.
  }
}

webView.addJavascriptInterface(new JsObject(), "android");
```
Now when interface is available within JavaScript code inside WebView, seatmap is going to execute `android.handleSeatmapEvent(serializedMessage)` method to send messages up to Android. You shall be able to receive every incoming message and build necessary business logic to handle it.


&nbsp;
### <a name="ios-integration"></a>&nbsp; iOS integration
Seatmap shall be rendered within WKWebView.

```swift
import WebKit
…

var webView = WKWebView()
var url = NSURL(string:"1764.html?<b>platform=ios</b>")
var req = NSURLRequest(URL:url)

webView!.loadRequest(req)
```
>⚠️&nbsp; It's important and required to include `?platform=ios` GET parameter into loadable url.

&nbsp;
**Publish [message](#available-communication-message-types) into the seatmap by executing public JavaScript function** — `web.sendMessage(message: string)`. Seatmap has this function available for you.

```swift
webView.evaluateJavaScript("web.sendMessage(\'{\"type\":\"SYNC_PASSENGERS\",\"data\":{\"flightGuid\":\"1ABC\",\"passengers\":[{\"id\":\"1\",\"seatPreference\":null,\"seat\":null}]}}\')
```

&nbsp;
**To receive [message](#available-communication-message-types) from the seatmap, follow [this article](https://kinderas.com/technology/2014/6/15/wkwebview-and-javascript-in-ios-8-using-swift).**
In general, WKWebView automatically injects `webkit.messageHandlers.callbackHandler.postMessage()` method into the seatmap. This method connects WKWebView contents and Swift native source code. Seatmap is going to execute this method to send messages up to iOS. You shall be able to build necessary business logic to handle incoming messages.

`postMessage(serializedMessage)` is going to receive the only parameter, that is serialised JSON object matching [`IMessage` standard interface](#available-communication-message-types). Learn to handle incoming messages and implement necessary business logic.

&nbsp;
## <a name="i18n-internationalization"></a>i18n internationalization &nbsp; 🇱🇻 &nbsp; 🇺🇸 &nbsp; 🇩🇪 &nbsp; 🇺🇦
Seatmap is able to display labels (especially seat descriptions) in multiple languages — English `en_US`, Russian `ru_RU` and Chinese Simplified `cn_CN`. Default language is English.

**To enforce seatmap labels display in a specific language, include** `?language=` GET **parameter into seatmap url/path rendered inside iframe, WebView or WKWebView.**

When seatmap is integrated into a website:
<pre>
iframe.src = 'http://demo-seatmap.quicket.me/htmls/dev/htmls/1764.html?<b>language=ru_RU</b>';
</pre>

When integrated into Android mobile app:
```java
webview.loadUrl("1764.html?platform=android&<b>language=ru_RU</b>");
```

Or within iOS app:
```swift
var url = NSURL(string:"1764.html?platform=ios&<b>language=ru_RU</b>")
var req = NSURLRequest(URL:url)

webView!.loadRequest(req)
```






&nbsp;
### <a name="color-themes"></a>Color themes
Seatmap styling could be changed using color themes. At the moment, we have 4 different color themes:
* default — [1764.html?colorTheme=default](http://demo-seatmap.quicket.me/htmls/dev/htmls/1764.html?colorTheme=default)
* kayak — [1764.html?colorTheme=kayak](http://demo-seatmap.quicket.me/htmls/dev/htmls/1764.html?colorTheme=kayak)
* momondo — [1764.html?colorTheme=momondo](http://demo-seatmap.quicket.me/htmls/dev/htmls/1764.html?colorTheme=momondo)
* skyscanner — [1764.html?colorTheme=skyscanner](http://demo-seatmap.quicket.me/htmls/dev/htmls/1764.html?colorTheme=skyscanner)

You can try to [create own color theme here](http://theme-editor.jets.kwiket.com/) and export a JSON file - we can built-in it in next release.

**To enforce seatmap use one of these color themes, include** `?colorTheme=` GET **parameter into seatmap url/path rendered inside iframe, WebView or WKWebView.**

When seatmap is integrated into a website:
```javascript
iframe.src = 'http://demo-seatmap.quicket.me/htmls/dev/htmls/1764.html?<b>colorTheme=skyscanner</b>';
```

When integrated into Android mobile app:
```java
webview.loadUrl("1764.html?platform=android&<b>colorTheme=kayak</b>&language=cn_CN");
```

Or within iOS app:
```swift
var url = NSURL(string:"1764.html?platform=ios&<b>colorTheme=momondo</b>&language=ru_RU")
var req = NSURLRequest(URL:url)

webView!.loadRequest(req)
```





&nbsp;
### <a name="available-communication-message-types"></a>📢&nbsp; Available communication message types
All messages follow standard `IMessage` interface that is used for bidirectional communication:
```typescript
interface IMessage {
  readonly type: string;
  readonly data: {};
}
```

`type` represents one of available message types (see below) and `data` is custom to every message.

Seatmap is able to receive and emit the following messages:
* [DECK\_\_SWITCH](#deck__switch)
* [SEAT\_AVAILABILITY\_AVAILABLE](#seat_availability_available)
* [SEAT\_MAP\_\_DATA](#seat_map__data)
* [SEAT\_MAP\_\_LOADED](#seat_map__loaded)
* [SYNC\_PASSENGERS](#sync_passengers)
* [WANNA\_CLOSE\_SEATMAP](#wanna_close_seatmap)
* [WANNA\_SEE\_VR](#wanna_see_vr)
* [SEAT\_\_JUMP\_TO](#seat__jump_to)
* [SEAT\_MAP\_\_SCROLL_TO](#seat_map__scroll_to)
* [SEAT\_\_PRESSED](#seat__pressed)

---
&nbsp;

#### <a name="seat_map__loaded"></a>SEAT\_MAP\_\_LOADED

This message fires up when content (DOM tree, images etc.) is loaded. It also provides height and width of document view.

Interface, describing data types:

```typescript
interface IWebViewData {
  deviceDPI: number;
  devicePixelRatio: number;
  heightInPx: number;
  planeId: number;
  platform: string; // "web" / "android" / "ios"
  scrollLeftInPx: number;
  scrollTopInPx: number;
  widthInPx: number;
}
```

Example of data parent layer receives:

```javascript
{
  "data": {
    "deviceDPI": 288,
    "devicePixelRatio": 3,
    "heightInPx": 736,
    "planeId": 1719,
    "platform": "ios",
    "scrollLeftInPx": 0,
    "scrollTopInPx": 1456,
    "widthInPx": 414
  },
  "type": "SEAT_MAP__LOADED"
}
```

---
&nbsp;

#### <a name="seat_map__data"></a>SEAT\_MAP\_\_DATA

This message works in both directions (request–response). It is similar to SEAT\_MAP\_\_LOADED except you can call it anytime.

Interface, describing data types:

```typescript
interface IWebViewData {
  deviceDPI: number;
  devicePixelRatio: number;
  heightInPx: number;
  planeId: number;
  platform: string;
  scrollLeftInPx: number;
  scrollTopInPx: number;
  widthInPx: number;
}
```

Example how to call it:

```javascript
{
  "data": {},
  "type": "SEAT_MAP__DATA"
}
```

Example of data parent layer receives:

```javascript
{
  "data": {
    "deviceDPI": 288,
    "devicePixelRatio": 3,
    "heightInPx": 736,
    "planeId": 1719,
    "platform": "ios",
    "scrollLeftInPx": 0,
    "scrollTopInPx": 1456,
    "widthInPx": 414
  },
  "type": "SEAT_MAP__DATA"
}
```

---
&nbsp;

#### <a name="wanna_see_vr"></a>WANNA\_SEE\_VR

This message gets bubbled up to the parent layer when 360° icon is clicked inside the seatmap. Parent layer shall implement logic to display 360° panorama.

![360° icon within seatmap](https://github.com/Kwiket/jets-vector-setmap-integration/blob/master/.wiki-resources/seatmap-360-icon.png)

Interface describing data received by the parent layer:
```typescript
interface IData {
  fragment: string; // Filename of the aerial panorama photo
}
```

Example of data parent layer receives via WANNA\_SEE\_VR message bubbled up from the embed seatmap:
```javascript
{
  "data": {
    "fragment": "6N"
  },
  "type": "WANNA_SEE_VR"
}
```

---
&nbsp;

#### <a name="wanna_close_seatmap"></a>WANNA\_CLOSE\_SEATMAP

Bubbles up to the parent layer when user wishes to close the seatmap and get back to the previous screen. Parent layer shall implement logic to close seatmap, returning user back to the previous state.

Optionally, there are some controls within the seatmap, that lets user emit message with this message.

Example of data parent layer receives:
```javascript
{
  "data": {},
  "type": "WANNA_CLOSE_SEATMAP"
}
```

---
&nbsp;

#### <a name="sync_passengers"></a>SYNC\_PASSENGERS

This message works both directions (bidirectional):

* When posted from parent layer, it enables passengers allocation within seatmap.
* Seatmap emits this message, when particular passenger picks the seat. Actually, every time when passenger picks its seat or preferences, seatmap emits this message with updated list of passengers.

Interfaces, describing data types:
```typescript
interface IAllocatablePassengers {
  readonly flightGuid: string;
  readonly passengers: IPassenger[];
}

interface IPassenger {
  readonly id: string;
  seatPreference: ISeatPreference;
  seat: ISeat;
}

interface ISeatPreference {
  cabinSection: TCabinSection;
  seatPosition: TSeatPosition;
  seatLabel: string;
}

interface ISeat {
  price: number;
  seatLabel: string;
}

type TCabinSection = 'NOSE' | 'MIDDLE' | 'TAIL';
type TSeatPosition = 'WINDOW' | 'MIDDLE' | 'AISLE';
```

Example of data when SYNC\_PASSENGERS message is fired:
```javascript
{
  "data": {
    "flightGuid": "1ABC",
    "passengers": [
      {
        "id": "1",
        "seatPreference": null,
        "seat": null
      },
      {
        "id": "2",
        "seatPreference": null,
        "seat": {
          "price": 0,
          "seatLabel": "12F"
        }
      },
      {
        "id": "3",
        "seatPreference": {
          "cabinSection": "TAIL",
          "seatPosition": "WINDOW",
          "seatLabel": "2A"
        },
        "seat": null
      }
    ]
  },
  "type": "SYNC_PASSENGERS"
}
```

Or even like this:

```javascript
{
  "data": {
    "flightGuid": "1ABC",
    "passengers": [
      { "id": "1" },
      { "id": "2" },
      { "id": "3" }
    ]
  },
  "type": "SYNC_PASSENGERS"
}
```

Please notice, that seatmap identifies how many passengers to allocate by a length of `passengers` array. Therefore, for example, to allocate 2 passengers without any predefined seat or its preferences, `passengers` array shall contain 2 items.

---
&nbsp;

#### <a name="seat_availability_available"></a>SEAT\_AVAILABILITY\_AVAILABLE

Parent layer is able to post this message to the embed seatmap, guiding seatmap to enable some seats for reservation, while disable others.

Interfaces, describing data types:
```typescript
interface IIncomingAvailibility {
  errors: string;
  results: IIncomingResult[];
}

interface IIncomingResult {
  availableClasses: IIncomingClass[];
  notAvailableClasses: IIncomingClass[];
}

interface IIncomingClass {
  classCode: string;
  seats: IIncomingSeat[];
}

interface IIncomingSeat {
  currency: string;
  label: string;
  price: number;
}
```

Example of data posted from parent layer to the seatmap via SEAT\_AVAILABILITY\_AVAILABLE message:

```javascript
{
  "data": {
    "errors": "",
    "results": [
      {
        "availableClasses": [
          {
            "classCode": "E",
            "seats": [
              {
                "currency": "USD",
                "label": "12E",
                "price": 33
              },
              {
                "currency": "USD",
                "label": "12F",
                "price": 13
              }
            ]
          }
        ]
      }
    ]
  },
  "type": "SEAT_AVAILABILITY_AVAILABLE"
}
```

You can pass seat with `label` __"*"__. This label work like __all__ selector for seats. In example below all seats with `classCode` equals __"E"__ will be enabled with `price` of _50_. However next seat configuration __"12F"__ will override `price` and set it to _75_.

```javascript
{
  "data": {
    "errors": "",
    "results": [
      {
        "availableClasses": [
          {
            "classCode": "E",
            "seats": [
              {
                "currency": "USD",
                "label": "*",
                "price": 50
              },
              {
                "currency": "USD",
                "label": "12F",
                "price": 75
              }
            ]
          }
        ]
      }
    ]
  },
  "type": "SEAT_AVAILABILITY_AVAILABLE"
}
```

---
&nbsp;

#### <a name="deck__switch"></a>DECK\_\_SWITCH

This event can be sent to embed seat map to switch plane deck. When event is sent field `deckId` is in priority. It should be numeric field equals 1 or 2. Currently there is only double-deck aircrafts exists. If  `deckId` is not defined or not valid application attempts to select deck by `seatLabel`.

Interfaces:

```typescript
interface IEventSwitchDeck {
  readonly data: {
    readonly deckId?: number; // 1 or 2 and only if plane has second deck
    readonly seatLabel?: string;
  };
  readonly type: string;
}
```

Example of data posted from parent layer to the seat map via DECK\_\_SWITCH message:
```javascript
{
  "data": {
    "deckId": 2, // this field is priority
    "seatLabel": "14D"
  },
  "type": "SWITCH_DECK"
}
```

---
&nbsp;

#### <a name="seat__jump_to"></a>SEAT\_\_JUMP\_TO

The event is called from the outside from parent layer. This event switch view to plane deck on which seat is located. Then try to scroll (jump) possibly close to the seat trying to fit into center of view. The seat may not be scrolled completely to the center depending on the layout or scroll position. If `tooltip` option is passed reservation window will be shown and view will be scrolled to fit it in. `effect` responsible for animation during jump. If ``

Data:

| Name | Type | Required | Default | Possible values |
|---|---|---|---|---|
| **label**  | _string_  | ✔️ | |
| **tooltip** | _boolean_ | ✖️ | `false` | `false`, `true` |
| **effect** | _string_ | ✖️ | `"none"` | `"none"` `"linear"` `"swing"` |
| **duration** | _integer_ | ✖️ | `400` | |

Interfaces:

```typescript
interface IEventSeatJumpTo {
  readonly data: {
    readonly label: string;
    readonly tooltip?: boolean;
    readonly effect?: string;
    readonly duration?: number;
  };
  readonly type: string;
}
```

Example of data posted from parent layer to the seat map via SEAT\_\_JUMP\_TO message:

> Instant jump to seat

```javascript
{
  "data": {
    "label": "18E",
    "tooltip": true,
  },
  "type": "SEAT__JUMP_TO"
}
```

> Motion scrolling to seat in half a second

```javascript
{
  "data": {
    "label": "18E",
    "tooltip": true,
    "effect": "swing",
  },
  "type": "SEAT__JUMP_TO"
}
```

> Linear scroll speed to seed in 600 ms.

```javascript
{
  "data": {
    "label": "18E",
    "tooltip": true,
    "effect": "linear",
    "duration": 600
  },
  "type": "SEAT__JUMP_TO"
}
```

---
&nbsp;

#### <a name="seat_map__scroll_to"></a>SEAT\_MAP\__SCROLL\_TO

Since version __0.4.0__.

This event post to parent layer information about scroll position changes.

Message has 250ms delay (debounce) after event happened.

If it's `Airbus A380` build or color theme of URL parameter is set to `&iflya380=true` - no scroll changes will be made and only message will posted to parent layer.

Interfaces:

```typescript
export interface IEventSeatMapScrollToData {
  /**
   * @description Element height to fit into view. If set and above zero, height might be important.
   * @default 0
   * @readonly
   * @type number
   */
  readonly height?: number;
  /**
   * @description Element width to fit into view. If set and above zero, width might be important.
   * @default 0
   * @readonly
   * @type number
   */
  readonly width?: number;
  /**
   * @description Vertical position of the scroll bar that should be set
   * @readonly
   * @type number
   */
  readonly x: number;
  /**
   * @description Horizontal position of the scroll bar that should be set
   * @readonly
   * @type number
   */
  readonly y: number;
}

export interface IEventSeatMapScrollTo extends IEvent {
  readonly data: IEventSeatMapScrollToData;
  readonly type: MessageType;
}
```

Example of data posted to parent layer to the seat map via SEAT\_MAP\__SCROLL\_TO message:

```javascript
{
  "data": {
    "x": 138.15,
    "y": 4074
  },
  "type": "SEAT_MAP__SCROLL_TO"
}
```

```javascript
{
  "data": {
    "height": 292,
    "width": 940.890625,
    "x": 0,
    "y": 4074
  },
  "type": "SEAT_MAP__SCROLL_TO"
}
```

---
&nbsp;

#### <a name="seat__pressed"></a>SEAT\_\_PRESSED

Interfaces:

```typescript
export interface IEventSeatPressed {
  readonly classLetter: string;
  readonly deckLevel: number;
  readonly fragment: string;
  readonly label:string;
  readonly type: string;
  readonly x: number;
  readonly y: number;
}

```

Example of data posted to parent layer to the seat map via SEAT\_\_PRESSED message:

```javascript
{
  "data": {
    "classLetter": "E",
    "deckLevel": 1,
    "fragment": "234876",
    "label": "60B",
    "type": "Economy",
    "x": 55.8,
    "y": 1108.88
  },
  "type": "SEAT__PRESSED"
}
```

---
&nbsp;
&nbsp;
### <a name="initialisation-and-workflow"></a>Initialisation and workflow
* When seatmap is rendered and communication is set up, seatmap will be optionally waiting for [`SEAT_AVAILABILITY_AVAILABLE`](#seat_availability_available) and [`SYNC_PASSENGERS`](#sync_passengers) messages. After receiving first message, seatmap is going to disable some seats, keeping others enabled. When second message arrives, seatmap is going unlock user with an opportunity to pick seat for reservation for every given passenger.

* Every time user picks particular seat for reservation, seatmap emits [`SYNC_PASSENGERS`](#sync_passengers) message back to the parent layer. Message contains information with picked seat for every initialised passenger.

* If [`SYNC_PASSENGERS`](#sync_passengers) message arrived, seatmap displays horizontal bar at the top. It contains ← icon (back button). When clicked, seatmap emits [`WANNA_CLOSE_SEATMAP`](#wanna_close_seatmap) message up to the parent layer, signalling user wishes to close the seatmap and return back to the previous view.

* Some seatmaps are going to contain 360° dots and VR buttons within seat tooltip. When user clicks these, seatmap emits [`WANNA_SEE_VR`](#wanna_see_vr) message.
