# Video + Audio Service (CLJD Flutter)

A ClojureDart Flutter video player with optional background audio.
Audio is handled by `just_audio` + `audio_service`, video display by `video_player`.

## Quick start

```bash
clj -M:cljd flutter
```

## Project layout

```
src/sample/
  video_player.cljd   — reusable video player package (widgets, lifecycle, playback logic)
  main.cljd           — sample app wiring VideoPlayer with a custom builder
lib/main.dart         — Flutter entrypoint; exports generated Dart
lib/cljd-out/         — generated Dart output (do not edit)
```

## API

The public API lives in `sample.video-player` (aliased as `vpl`).

### VideoPlayer

`VideoPlayer` is a lifecycle widget that manages `AudioPlayer`, `VideoPlayerController`,
and optionally the `audio_service` for lock screen controls. It requires a `::builder`
function that receives the **initialized-video** map.

```clojure
(vpl/VideoPlayer {::vpl/url     "https://example.com/video.mp4"
                  ::vpl/title   "My Video"
                  ::vpl/album   "Album"
                  ::vpl/artist  "Artist"
                  ::vpl/art-uri "https://example.com/art.jpg"
                  ::vpl/builder my-view-fn})
```

### Config keys

| Key                    | Required | Description                                   |
|------------------------|----------|-----------------------------------------------|
| `::vpl/url`            | yes      | Network URL of the video/audio source         |
| `::vpl/title`          | yes      | Media title                                   |
| `::vpl/album`          | yes      | Album name                                    |
| `::vpl/artist`         | yes      | Artist name                                   |
| `::vpl/art-uri`        | yes      | Album art URL                                 |
| `::vpl/builder`        | yes      | `(fn [initialized-video] -> Widget)`          |
| `::vpl/background-audio` | no    | `false` to skip audio service (default: true) |

### The initialized-video map

Your `::builder` function receives a map with all config keys above, plus:

| Key                        | Type                       | Description                              |
|----------------------------|----------------------------|------------------------------------------|
| `::vpl/audio-player`       | `ja/AudioPlayer`           | Managed audio player instance            |
| `::vpl/video-controller`   | `vp/VideoPlayerController` | Managed video controller (muted)         |
| `::vpl/audio-service-handler` | `MyAudioHandler` or nil | Lock screen handler (nil without bg)     |
| `::vpl/media-stream`       | `Stream`                   | `{:duration Duration :position Duration}` |
| `::vpl/playing-stream`     | `Stream<bool>`             | Reactive playing/paused state            |
| `::vpl/on-seek`            | `(fn [Duration])`          | Seek to position                         |
| `::vpl/on-play`            | `(fn [])`                  | Start playback                           |
| `::vpl/on-pause`           | `(fn [])`                  | Pause playback                           |
| `::vpl/on-stop`            | `(fn [])`                  | Stop playback                            |
| `::vpl/on-rewind`          | `(fn [])`                  | Rewind 10 seconds                        |
| `::vpl/on-fast-forward`    | `(fn [])`                  | Fast-forward 10 seconds                  |

### Built-in widgets

All widgets accept the initialized-video map directly:

| Widget              | Description                                                |
|---------------------|------------------------------------------------------------|
| `vpl/VideoLoader`   | Shows video when initialized, loading spinner while buffering |
| `vpl/BasicTitle`    | Reactive title from audio service, or static `::title`     |
| `vpl/PlayerControls`| Rewind, play/pause, stop, fast-forward buttons             |
| `vpl/SeekBar`       | Draggable slider with formatted time                       |
| `vpl/PlayerState`   | Text displaying current playing/paused state               |

### Background audio

By default, `VideoPlayer` initializes `audio_service` for lock screen controls
and background playback. Pass `::vpl/background-audio false` to skip it:

```clojure
(vpl/VideoPlayer {::vpl/url     "https://example.com/video.mp4"
                  ::vpl/title   "My Video"
                  ::vpl/album   "Album"
                  ::vpl/artist  "Artist"
                  ::vpl/art-uri "https://example.com/art.jpg"
                  ::vpl/background-audio false
                  ::vpl/builder my-view-fn})
```

Without background audio:
- No lock screen controls or media notifications
- Controls talk directly to `AudioPlayer` instead of through the audio service handler
- Rewind/fast-forward seek by 10 seconds on the player directly
- `::vpl/audio-service-handler` will be `nil` in the initialized-video map
- `BasicTitle` falls back to the static `::vpl/title` value

## Examples

### Using the default layout

```clojure
(ns my-app
  (:require [sample.video-player :as vpl]
            ["package:flutter/material.dart" :as m]))

(defn default-layout [initialized-video]
  [(vpl/VideoLoader initialized-video)
   (m/SizedBox .height 20)
   (vpl/BasicTitle initialized-video)
   (m/SizedBox .height 10)
   (vpl/PlayerControls initialized-video)
   (m/SizedBox .height 10)
   (vpl/SeekBar initialized-video)
   (m/SizedBox .height 20)
   (vpl/PlayerState initialized-video)])

(defn my-view [initialized-video]
  (m/SingleChildScrollView
    .child
    (m/Padding
      .padding (m/EdgeInsets.all 16.0)
      .child
      (m/Column
        .crossAxisAlignment m/CrossAxisAlignment.stretch
        .children (default-layout initialized-video)))))

;; Usage
(vpl/VideoPlayer {::vpl/url     "..."
                  ::vpl/title   "..."
                  ::vpl/album   "..."
                  ::vpl/artist  "..."
                  ::vpl/art-uri "..."
                  ::vpl/builder my-view})
```

### Custom builder with extra widgets

```clojure
(defn custom-view
  [{::vpl/keys [on-play on-pause playing-stream] :as iv}]
  (m/Column
    .children
    [(vpl/VideoLoader iv)
     (vpl/SeekBar iv)
     ;; Custom button using callbacks from the map
     (f/widget
       :watch [playing playing-stream]
       (m/ElevatedButton
         .onPressed (if playing on-pause on-play)
         .child (m/Text (if playing "Pause" "Play"))))]))
```

### Accessing initialized-video before rendering

```clojure
(vpl/VideoPlayer
  {::vpl/url "..."
   ::vpl/title "..."
   ::vpl/album "..."
   ::vpl/artist "..."
   ::vpl/art-uri "..."
   ::vpl/builder
   (fn [{::vpl/keys [audio-player media-stream] :as iv}]
     ;; Inspect any key before deciding what to render
     (println "Audio player:" audio-player)
     (my-custom-view iv))})
```

## Notes

- Background audio behavior is driven by `audio_service` and may require platform-specific configuration.
- The sample media is remote; network access is required.
- IDE diagnostics (clj-kondo) show false warnings for ClojureDart interop types — compile warnings from `clj -M:cljd flutter` are the real signal.