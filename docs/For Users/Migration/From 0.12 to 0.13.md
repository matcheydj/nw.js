# Migrate from 0.12 to 0.13 {: doctitle}
---

[TOC]

## Architecture Changes

+ NW.js application is running as a Chrome App internally. All chrome.* platform APIs and features can be used in NW application now. The default protocol is changed from 'file://' to 'chrome-extension://', where the host part of the URL is the generated id. The 'app://' protocol in 0.12 is replaced by 'chrome-extension://' protocol.
+ All NW specific APIs, including 'require()' is moved into a 'nw' object from the 'nw.gui' library. However, we provided a builtin wrapper library to provide compatibility for 0.12 apps. You can use 'nw.gui' library for some time before we deprecate it in 0.14 or later.
+ The Node.js context is put in the DOM context of the background page, which is shared between opening windows as in 0.12 and before. The difference is you have access to all DOM features and chrome.* platform APIs in the Node context in 0.13.
+ The entry of the application is either JS or HTML as in 0.12, but as the application is internally a Chrome App, the first window is supposed to be launched by JS from the background page. If you specify a HTML file as the entry with "main" field in package.json, NW will use a default JS to open the first window and load it.
+ If NW.js is running under [Mixed Context Mode](../Advanced/JavaScript Contexts in NW.js.md#mixed-context-mode) (boot NW.js with `--mixed-context` argument), `nw.*` is kind of mirror of `window.*`. In this mode, you **CANNOT** share variables among frames or windows by assigning it to Node context. So do **NOT** turn on Mixed Context mode if your application is heavily depending on this variable sharing feature.

## Node.js Changes

1. Node.js is bumped to 5.x in latest build. Check your NPM modules to make sure they support Node.js 5.x **especially for native modules**. There is [a list of native modules](https://github.com/nodejs/node/issues/2798) which should be migrated to latest NaN 2.

## API Changes

### Build Flavors

1. Different build flavors support different set of APIs and capabilities. See [Build Flavors](../Advanced/Build Flavors.md) to choose the right NW.js flavor or [build your own](../../For Developers/Building NW.js.md).

### Shorcut

1. `Shortcut` API does **NOT** map <kbd>Ctrl</kbd> modifier to <kbd>&#8984;</kbd> on Mac OS X. However 0.13.0 supports `Command` modifier in cross platform way. So it's your responsible to detect the OS and choose the right modifier when registering hotkeys. See [Shortcut.key](../../References/Shortcut.md#shortcutkey) for details.

### Manifest Format

1. [`single-instance`](../../References/Manifest Format.md#single-instance) is **deprecated** and it's always `true`. You **CANNOT** have multiple instances for your app.
2. [`toolbar`](../../References/Manifest Format.md#toolbar) is **deprecated** and it's always `false`. The traditional toolbar will **NOT** be supported including the reload buttons, location bar and DevTools buttons. As a workaround, you can open / close DevTools with <kbd>F12</kbd> (Windows & Linux) or <kbd>&#8984;</kbd>+<kbd>&#8997;</kbd>+<kbd>i</kbd> (Mac). And use [`win.reload()`](../../References/Window.md#winreload) and [`win.reloadDev()`](../../References/Window.md#winreloaddev) to simulate the reload buttons.
3. [`no-edit-menu`](../../References/Manifest Format.md#no-edit-menu-mac) is **deprecated**.
4. [`snapshot`](../../References/Manifest Format.md#snapshot) is **deprecated**. Use [`win.evalNWBin()`](../../References/Window.md#winevalnwbin) instead.
5. The format of [`node-remote`](../../References/Manifest Format.md#node-remote) is changed to array of [match patterns](https://developer.chrome.com/extensions/match_patterns) used by Chrome extension.

### Window

1. Event `capturepagedone` of [`Window` API](../../References/Window.md#event-capturepagedone) is **deprecated**. Use the callback with the [`win.capturePage(callback [, config ])`](../../References/Window.md#wincapturepagecallback--config-) instead.
2. [Window.open](../../References/Window.md#windowopenurl-options-callback) is changed to passing the created window as the argument of the callback.
3. [Window.showDevtools](../../References/Window.md#winshowdevtoolsiframe-headless-callback) is changed to passing the created window as the argument of the callback.
4. [win.setTransparent](../../References/Window.md#winsettransparent) is **deprecated**. You can't change the transparency after window is created.
5. [unmaximize](../../References/Window.md#event-unmaximize) and [leave-fullscreen](../../References/Window.md#event-leave-fullscreen) events of `Window` object is **deprecated** and replaced by [restore](../../References/Window.md#event-restore). When window is restored from minimized, maximized or fullscreen, `restore` event is triggered instead.
6. Window options 'always-on-top' and 'visible-on-all-workspaces' is renamed 'always_on_top' and 'visible_on_all_workspaces' respectively in package.json or as argument of Window.open().
7. Window is not inherited from EventEmitter anymore, but the methods on(), once(), removeListener() and removeAllListeners() are still supported.
