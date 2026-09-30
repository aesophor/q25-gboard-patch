## The Problem

Android's `MetaKeyKeyListener` latches `KEYCODE_ALT_LEFT`: tap ALT once and it applies to the next key, tap twice and it locks. On the Q25 you press ALT constantly to type symbols, so the latch is almost always armed. That caused:

- **ALT+DEL wiped the whole line** — `BaseKeyListener.backspace()` does that by design when ALT is active.
- **Backspace appeared dead** after typing a symbol.
- **Invisible `U+0008` characters** inserted into fields that don't treat it as a backspace (renaming a file in Google Files, for example).

The latch is stored as spans on the `Editable`, not in the key event, so it's invisible to `KeyEvent.getMetaState()` — which is why no `alt:` workaround in a `.kcm` can fix it properly.

## The fix

Remap the ALT key so it isn't ALT any more:

```
map key 56 FUNCTION
```

`KEYCODE_FUNCTION` is a plain held modifier that the latch machinery doesn't track, so no latch can form. The symbol layer moves from `alt:` to `fn:`, generated from the device's own `/system/usr/keychars/Q25_keyboard.kcm` — holding ALT types the same symbols as before.

Also fixes the **`$` key**, which the firmware maps to `KEYCODE_GRAVE` and which typed a backtick. It now types `$`; backtick moves to hold-ALT + that key.

## Trade-off

App shortcuts bound to ALT+*key* (LINE's ALT+ENTER to send, for instance) no longer fire from the left ALT key, since Android no longer sees it as ALT.

Use **SYM** instead — it's still `KEYCODE_ALT_RIGHT` and sets `META_ALT_ON`.

## Editing the layout

`app/src/main/res/raw/keyboard_layout_en_us.kcm` is the whole patch.

Two things to watch:

- Write `\u0008` as six literal characters. A raw `0x08` byte makes the file unparseable, and Android then silently falls back to the stock layout — which looks exactly like the fix regressing. Check with:
  ```sh
  LC_ALL=C grep -c '[^[:print:][:space:]]' app/src/main/res/raw/keyboard_layout_en_us.kcm   # want 0
  ```
- An overlay `key X { }` block **replaces** the key's entire definition, so copy `base:`, `shift:` and the rest even when you're only adding one line.

## Building

Gradle 5.6.4 + AGP 3.6.3 require **JDK 8** — Java 11+ fails. Homebrew has no ARM bottle for `openjdk@8`, so this machine uses a hand-unpacked Zulu 8. Neither tool is on PATH:

```sh
export JAVA_HOME=~/Library/Java/JavaVirtualMachines/zulu8.96.0.205-ca-jdk8.0.504-macosx_aarch64/Contents/Home
export ANDROID_HOME=/opt/homebrew/share/android-commandlinetools
./gradlew assembleDebug
```

`adb` lives at `$ANDROID_HOME/platform-tools/adb`.

Two things that make a change look broken when it isn't:

- **Android caches the `.kcm`.** After installing, re-select the layout in Settings -> System -> Languages & input -> Physical keyboard, or you are testing the previous file.
- **A malformed `.kcm` still builds clean.** Gradle never parses it; Android rejects it at selection time and silently falls back to the stock layout. Symptoms look exactly like the fix regressing, so suspect a parse error first. Not every keycode name is accepted either -- `EMOJI_PICKER` is rejected even on Android 14.

`applicationId` is `dev.aesophor.q25gboardpatch`, so this installs as a separate app from any build still using upstream's `ris58h.custom_keyboard_layout`.
