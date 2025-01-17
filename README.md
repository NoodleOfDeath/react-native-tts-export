# React Native TTS + Audio File Export <!-- omit in toc -->

<p align="center">
  <a href="https://github.com/facebook/react-native/blob/HEAD/LICENSE">
    <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="React Native is released under the MIT license." />
  </a>
  <a href="https://www.npmjs.org/package/react-native-tts-export">
    <img src="https://img.shields.io/npm/v/react-native-tts-export?color=brightgreen&label=npm%20package" alt="Current npm package version." />
  </a>
  <a href="https://www.npmjs.org/package/react-native-tts-export">
    <img src="https://img.shields.io/npm/dt/react-native-tts-export" alt="Npm downloads." />
  </a>
  <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs welcome!" />
</p>

React Native TTS Export is a text-to-speech library fork of [react-native-tts](https://github.com/ak1394/react-native-tts)
 for [React Native](https://facebook.github.io/react-native/) on iOS, Android, and Windows **with added audio file export functionality.**


## Table of Contents <!-- omit in toc -->

- [Install](#install)
  - [Automatic Linking](#automatic-linking)
- [Usage](#usage)
  - [Windows](#windows)
  - [Speaking](#speaking)
  - [Exporting to Audio File](#exporting-to-audio-file)
  - [Waiting for initialization](#waiting-for-initialization)
  - [Ducking](#ducking)
  - [List Voices](#list-voices)
  - [Set default Language](#set-default-language)
  - [Set default Voice](#set-default-voice)
  - [Set default Speech Rate](#set-default-speech-rate)
  - [Set default Pitch](#set-default-pitch)
  - [Controls the iOS silent switch behavior](#controls-the-ios-silent-switch-behavior)
  - [Events](#events)
  - [Support for multiple TTS engines](#support-for-multiple-tts-engines)
  - [Install (additional) language data](#install-additional-language-data)
- [Troubleshooting](#troubleshooting)
  - [No text to speech engine installed on Android](#no-text-to-speech-engine-installed-on-android)
- [Example](#example)

## Install

```shell
npm install --save react-native-tts-export
```

```shell
yarn add react-native-tts-export
```

### Automatic Linking

```shell
react-native link react-native-tts-export
```

## Usage

```js
import Tts from 'react-native-tts-export';
```

### Windows

1. In `windows/myapp.sln` add the `RNTTS` project to your solution:

   - Open the solution in Visual Studio 2019
   - Right-click Solution icon in Solution Explorer > Add > Existing Project
   - Select `node_modules\react-native-tts\windows\RNTTS\RNTTS.vcxproj`

2. In `windows/myapp/myapp.vcxproj` add a reference to `RNTTS` to your main application project. From Visual Studio 2019:

   - Right-click main application project > Add > Reference...
   - Check `RNTTS` from Solution Projects.

3. In `pch.h` add `#include "winrt/RNTTS.h"`.

4. In `app.cpp` add `PackageProviders().Append(winrt::RNTTS::ReactPackageProvider());` before `InitializeComponent();`.

### Speaking

Add utterance to TTS queue and start speaking. Returns promise with utteranceId.

```js
Tts.speak('Hello, world!');
```

Additionally, `speak()` allows to pass platform-specific options.

```js
// iOS
Tts.speak('Hello, world!', {
  iosVoiceId: 'com.apple.ttsbundle.Moira-compact',
  rate: 0.5,
});
// Android
Tts.speak('Hello, world!', {
  androidParams: {
    KEY_PARAM_PAN: -1,
    KEY_PARAM_VOLUME: 0.5,
    KEY_PARAM_STREAM: 'STREAM_MUSIC',
  },
});
```

For more detail on `androidParams` properties, please take a look at [official android documentation](https://developer.android.com/reference/android/speech/tts/TextToSpeech.Engine.html). Please note that there are still unsupported key with this wrapper library such as `KEY_PARAM_SESSION_ID`. The following are brief summarization of currently implemented keys:

- `KEY_PARAM_PAN` ranges from `-1` to `+1`.

- `KEY_PARAM_VOLUME` ranges from `0` to `1`, where 0 means silence. Note that `1` is a default value for Android.

- For `KEY_PARAM_STREAM` property, you can currently use one of `STREAM_ALARM`, `STREAM_DTMF`, `STREAM_MUSIC`, `STREAM_NOTIFICATION`, `STREAM_RING`, `STREAM_SYSTEM`, `STREAM_VOICE_CALL`,

The supported options for iOS are:

- `iosVoiceId` which voice to use, check [`voices()`](#list-voices) for available values
- `rate` which speech rate this line should be spoken with. Will override [default rate](#set-default-speech-rate) if set for this utterance.

Stop speaking and flush the TTS queue.

```js
Tts.stop();
```

### Exporting to Audio File

*(not supported in Windows yet)*

Exports an utterance to an audio file that can be used for track playing with [react-native-track-player](https://www.npmjs.com/package/react-native-track-player) or used by another app. This file will be generated in the app's cache directory.

Additionally, `export()` takes in all of the same parameters as [`speak()`](#speaking).

- iOS, will generate a `.caf` file.
- Android, will generate a `.wav` file.

```js
const filepath = await Tts.export('Hello, world!', {
  filename: 'myfile', 
  overwrite: true, // optional (not passing true will return the already existing file)
  ...,
  iosVoiceId: 'com.apple.ttsbundle.Moira-compact',
  rate: 0.5,
  androidParams: {
    ...
  },
});
console.log(filepath); 
// /storage/emulated/0/.../cache/myfile.wav
// /var/mobile/Containers/.../Caches/myfile.caf
```

### Waiting for initialization

On some platforms it could take some time to initialize TTS engine, and `Tts.speak()` will fail to speak until the engine is ready.

To wait for successfull initialization you could use `getInitStatus()` call.

```js
Tts.getInitStatus().then(() => {
  Tts.speak('Hello, world!');
});
```

### Ducking

Enable lowering other applications output level while speaking (also referred to as "ducking").

*(not supported on Windows)*

```js
Tts.setDucking(true);
```

### List Voices

Returns list of available voices

*(not supported on Android API Level < 21, returns empty list)*

```js
Tts.voices().then(voices => console.log(voices));

// Prints:
//
// [ { id: 'com.apple.ttsbundle.Moira-compact', name: 'Moira', language: 'en-IE', quality: 300 },
// ...
// { id: 'com.apple.ttsbundle.Samantha-compact', name: 'Samantha', language: 'en-US' } ]
```

| Voice field               | Description                                                                                      |
| ------------------------- | ------------------------------------------------------------------------------------------------ |
| id                        | Unique voice identifier (e.g. `com.apple.ttsbundle.Moira-compact`)                               |
| name                      | Name of the voice *(iOS only)*                                                                   |
| language                  | BCP-47 language code (e.g. 'en-US')                                                              |
| quality                   | Voice quality (300 = normal, 500 = enhanced/very high)                                           |
| latency                   | Expected synthesizer latency (100 = very low, 500 = very high) *(Android only)*                  |
| networkConnectionRequired | True when the voice requires an active network connection *(Android only)*                       |
| notInstalled              | True when the voice may need to download additional data to be fully functional *(Android only)* |


### Set default Language

```js
Tts.setDefaultLanguage('en-IE');
```

### Set default Voice

Sets default voice, pass one of the voiceId as reported by a call to `Tts.voices()`

*(not available on Android API Level < 21)*

```js
Tts.setDefaultVoice('com.apple.ttsbundle.Moira-compact');
```

### Set default Speech Rate

Sets default speech rate. The rate parameter is a float where where 0.01 is a slowest rate and 0.99 is the fastest rate.

```js
Tts.setDefaultRate(0.6);
```

There is a significant difference to how the rate value is interpreted by iOS, Android and Windows native TTS APIs. To provide unified cross-platform behaviour, translation is applied to the rate value. However, if you want to turn off the translation, you can provide optional `skipTransform` parameter to `Tts.setDefaultRate()` to pass rate value unmodified.

Do not translate rate parameter:

```js
Tts.setDefaultRate(0.6, true);
```

### Set default Pitch

Sets default pitch. The pitch parameter is a float where where 1.0 is a normal pitch. On iOS min pitch is 0.5 and max pitch is 2.0. On Windows, min pitch is 0.0 and max pitch is 2.0.

```js
Tts.setDefaultPitch(1.5);
```

### Controls the iOS silent switch behavior

Platforms: iOS

- "inherit" (default) - Use the default behavior
- "ignore" - Play audio even if the silent switch is set
- "obey" - Don't play audio if the silent switch is set

```js
Tts.setIgnoreSilentSwitch("ignore");
```

### Events

Subscribe to TTS events

```js
Tts.addEventListener('tts-start', (event) => console.log("start", event));
Tts.addEventListener('tts-progress', (event) => console.log("progress", event));
Tts.addEventListener('tts-finish', (event) => console.log("finish", event));
Tts.addEventListener('tts-cancel', (event) => console.log("cancel", event));
```

### Support for multiple TTS engines

Platforms: Android

Functions to list available TTS engines and set an engine to use.

```js
Tts.engines().then(engines => console.log(engines));
Tts.setDefaultEngine('engineName');
```

### Install (additional) language data

Shows the Android Activity to install additional language/voice data.

```js
Tts.requestInstallData();
```

## Troubleshooting

### No text to speech engine installed on Android

On Android, it may happen that the Text-to-Speech engine is not (yet) installed on the phone.
When this is the case, `Tts.getInitStatus()` returns an error with code `no_engine`.
You can use the following code to request the installation of the default Google Text to Speech App.
The app will need to be restarted afterwards before the changes take affect.

```js
Tts.getInitStatus().then(() => {
  // ...
}, (err) => {
  if (err.code === 'no_engine') {
    Tts.requestInstallEngine();
  }
});
```

## Example

There is an example project which shows use of react-native-tts on Android/iOS/Windows: https://github.com/themostaza/react-native-tts-example
