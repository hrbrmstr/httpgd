# httpgd

Asynchronous http server graphics device for R.

## Features

* SVG over HTTP.
* Automatic resizing.
* Plot history.
* HTML live server.
* Stateless API.
* Multiple concurrent device server.
* Multiple concurrent clients for each device.

## Demo

![demo](https://user-images.githubusercontent.com/33600480/83210273-b4c29100-a15a-11ea-8757-052dcf259a1c.gif)

## Installation

```R
devtools::install_github("nx10/httpgd")
```

Depends on `Rcpp`, `later` and `gdtools`.

SVG rendering (especially font rendering) based on `svglite` (https://github.com/r-lib/svglite).

Includes `cpp-httplib` (https://github.com/yhirose/cpp-httplib).

## Usage

Initialize graphics device and start live server with:

```R
httpgd::httpgd()
```

Plot what ever you want.

```R
x = seq(0, 3 * pi, by = 0.1)
plot(x, sin(x), type = "l")
```

Every plotting library should work.

```R
library(ggplot2)
ggplot(mpg, aes(displ, hwy, colour = class)) +
  geom_point()
```

The live server should refresh automatically and detect window size changes.

Stop the server with:

```R
dev.off()
```

### Keyboard shortcuts

| Keys | Result |
|:----:|--------|
| <kbd>&#8592;</kbd> <kbd>&#8594;</kbd> <kbd>&#8593;</kbd> <kbd>&#8595;</kbd> | Navigate plot history. |
| <kbd>del</kbd> | Clear plot history. |
| <kbd>+</kbd> / <kbd>-</kbd> | Zoom in and out. |
| <kbd>0</kbd> | Reset zoom level. |
| <kbd>ctrl</kbd> + <kbd>S</kbd> | Download the currently visible plot. |

### HTTP API

| Endpoint  | Method | Description |
|-----------|--------|-------------|
| `/`       | `GET`  | Welcome message. |
| `/live`   | `GET`  | Returns HTML/Javascript live server page. |
| `/svg`    | `GET`  | Get rendered SVG. Query parameters are listed below. |
| `/state`  | `GET`  | Get current server state. This can be used to check if there were new draw calls in R. |
| `/clear`  | `GET`  | Clear plot history. |

#### Query parameter

| Key      | Value | 
|----------|-------|
| `width`  | With in pixels. |
| `height` | Height in pixels. |
| `index`  | Plot history index. If set to `-1`, the last available plot will be returned. |
| `token`  | If security tokens are used this should be equal to the previously secret token. |

#### Server state

| Field        | Type     | Description |
|--------------|----------|-------------|
| `upid`       | `int`    | Update id. |
| `width`      | `double` | Graphics device width (pixel). |
| `height`     | `double` | Graphics device height (pixel). |
| `hrecording` | `bool`   | Whether the graphics device is recording a plot history. |
| `hsize`      | `int`    | Number of plot history entries. |
| `hindex`     | `int`    | Index of the displayed plot entry. |
| `needsave`   | `bool`   | If this is set to true the opened plot was not yet added to the history. (The plot can still be added to.) |

#### Security

A security token can be set when starting the device: 
```R
httpgd(..., token = "secret")
```
When set, each API request has to include this token inside the header `X-HTTPGD-TOKEN` or as a query param `?token=secret`.

CORS is off by default but can be enabled on startup:

```R
httpgd(..., cors = TRUE)
```

## Note

Any advice and suggestions are welcome!

## Planned features

* Use websockets to push changes directly/faster (maybe switch http server backend library).
* Optimization.

## Mac OS

It seems like a recent mac OS update (Catalina) removed shared libraries used by the package `systemfonts` we depend on (see: https://github.com/r-lib/systemfonts/issues/17).

You can install XQuartz as a workaround (https://www.xquartz.org/).