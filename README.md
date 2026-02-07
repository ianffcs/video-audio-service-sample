# Video + Audio Service (CLJD Flutter Sample)

A ClojureDart (CLJD) Flutter sample that plays a network video while keeping audio in the background. Audio is handled by `just_audio` + `audio_service`, video display by `video_player`. The UI includes play/pause/seek/rewind/fast-forward and keeps video/audio positions in sync.

## Features
- Background audio with media notification and lock-screen controls
- Video playback with muted video track
- Seek bar and playback state display
- Demo media: Big Buck Bunny (public URLs)

## Project Layout
- `src/sample/main.cljd`: primary app source (ClojureDart)
- `lib/main.dart`: Flutter entrypoint; exports generated Dart
- `lib/cljd-out/`: generated Dart output (do not edit by hand)

## Editing CLJD Sources
If you edit `src/sample/main.cljd`, regenerate Dart output before running Flutter:

```bash
clj -M:cljd init
clj -M:cljd flutter run
```

Then run cljd as above.

## Notes
- Background audio behavior is driven by `audio_service` and may require platform-specific tweaks.
- The sample media is remote; network access is required.
