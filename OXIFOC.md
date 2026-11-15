# OxiFOC integration branch

Based on upstream 0.20.0 (`7c7054702d11f00fe99143c4ea14a0fb6ba71340`).

- `dioxus-blec` and its Android Manganis module use Dioxus
  `97adf7915e1adfef4e2da436c9bd0b6add0dfa39` (0.8 alpha), matching OxiFOC.
- Dropping a discovery receiver stops scanning instead of panicking and
  leaving the adapter in scanning state. Peripheral enumeration errors also
  finish the scan normally. Regression covered by the BLE handler suite.

OxiFOC uses base `blec` in its host library. NUS framing, device selection,
reconnect policy and telemetry remain there. Dioxus hooks are optional UI
helpers; their latest-value notification signals are not a protocol queue.
Android hosts must package `crates/blec/android/lib` (minSdk 26). Dioxus's
wrapper carries that module through Manganis. Other hosts can include the
same Gradle module; `embedded-dex` is an explicit alternative for cargo-apk.

Connection setup now opens the notification stream before returning, and
subscription holds the listener lock while enabling CCCD. This preserves the
first notification even when it arrives immediately. The regression test
intentionally omits the scheduling delay used by the older test helpers.

OS-wide connect events for foreign devices are ignored. Only a connection
arriving within 60 seconds of our own abandoned pending attempt is torn down.
This avoids disconnecting peripherals owned by another desktop application.
The foreign-connection and randomized stress invariants cover this boundary;
late completion of our timed-out connect remains covered separately.

For Android WebView hosts, enable `dioxus-blec/android-webview`. Its
`prepare_android()` dispatches through Wry with the current Activity before
initializing the bridge; `use_ble` invokes it automatically. Custom shared
backends call it before their own initialization. Dioxus 0.8 stores an
Application in `ndk-context`, which cannot itself present runtime permissions.
The standalone Dioxus example also follows 0.8 and its named-key API.
