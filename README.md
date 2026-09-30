## Q25 Gboard Patch
Added a [new physical keyboard layout](https://github.com/aesophor/q25-gboard-patch/blob/6e4a9593d09b91c6464708e3ea2d2c147c514e9b/app/src/main/res/raw/keyboard_layout_en_us.kcm) "English (US), Q25 Gboard Patch" which fixes:
1. the sticky ALT_LEFT problem
2. the dollar sign key (previously mismapped to backtick by the firmware)

Adds the following shortcuts:
1. sym + s = symbol picker
2. sym + a = emoji picker

Note that:
1. ALT_LEFT is no longer sticky.
2. SYM (ALT_RIGHT) is still sticky, this is something I can't fix.

## Requirements
No root, no shinzuku, no accessibility needed.

## Build apk (optional)
Needs **JDK 8** (Gradle 5.6.4 + AGP 3.6.3) and SDK platform 29 with build-tools 29.0.3.
```sh
export JAVA_HOME=/path/to/jdk8
export ANDROID_HOME=/path/to/android-sdk
./gradlew assembleDebug
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

## How to use
1. Install the apk from the [release page](https://github.com/aesophor/q25-gboard-patch/releases).
2. Android will warn you if you want to install this unsafe app, enter you passcode and install it.
3. Go to Settings > System > Keyboard > Physical keyboard > Q25_keyboard, and set all languages to `English (US), Q25 Gboard Patch`

## Limitations
1. Zinwa Q25 + Gboard only.
2. English (US) layout only (for now).
   - If you need another keyboard layout, you can ask claude code to fix it.

## Compromise
ALT_LEFT isn't sticky anymore, but with the following compromises:
1. You can't hold SYM to type symbols anymore, hold ALT instead.
2. ALT + Enter won't behave like "send" as before, use SYM + Enter instead.

## Extra Goodies
How to make Gboard switch input language with physical keys?
1. download [the latest version of Key Mapper](https://github.com/keymapperorg/KeyMapper) from github, since the Google Play version can't be installed.
2. open Key Mapper, and add a new entry:
   - trigger: Shift Left (Do not remap) + Space
   - actions: Cycle keyboard language

## Credit
Forked from [ris58h/custom-keyboard-layout](https://github.com/ris58h/custom-keyboard-layout). Validate a `.kcm` with the author's [validatekeymaps](https://ris58h.github.io/validatekeymaps/).
