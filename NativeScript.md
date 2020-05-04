The following are confirmed to work on NativeScript iOS project:

1. Version 5.3.4 of NativeScript ("TNS")
2. [NativeScript WebSockets driver extension](https://www.npmjs.com/package/nativescript-websockets)
3. [GunDB version >= 0.9 (April 2019)](https://github.com/amark/gun/commit/a111e20f271c011b4c0c739b8c4f41d0605fb8ca)

However it appears that only in memory storage is possible with this use case.
Testing with NativeScript's WebView and on Android will be forthcoming.

amark has reference a [NativeScriptVue example](https://github.com/gundb/feature-requests/issues/26).