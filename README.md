# SAL · Enkoder Opus (wielokanałowy / AmbiX)

Samodzielna aplikacja SAL: konwersja plików **WAV → Opus / WebM** w przeglądarce, z pełną kontrolą
parametrów i objaśnieniem każdego z nich. W przeciwieństwie do `MediaRecorder` (maks. 2 kanały)
zachowuje **wszystkie kanały** — więc 4-kanałowy AmbiX FOA (W,Y,Z,X) przechodzi przez kodek bez
downmixu. Działa też dla stereo, mono i HOA.

Silnik: **WASM libopus** przez ffmpeg.wasm. Kodowanie dzieje się lokalnie — plik nie jest nigdzie
wysyłany.

## Pliki

- `index.html` — cała aplikacja (jeden plik: HTML + CSS + JS inline).
- Rdzeń ffmpeg (~31 MB) ładowany z CDN (jsDelivr) przy pierwszym kodowaniu.

## Wdrożenie w ekosystemie SAL

Wrzuć folder jako podstronę (np. `/opus-encoder/`) w repo strony SAL — działa na GitHub Pages bez
buildu. Nagłówek ma link `← Hub` do `spatial-audio-lab.github.io`. Fonty Lexend + Azeret Mono i
stopka KPO są zgodne z manifestem v3.0.

Opcjonalnie dodaj `logo-cutout.png` obok `index.html`, by pokazać monogram SAL w nagłówku (obecnie
jest strzałka tekstowa).

## Parametry (wszystkie objaśnione w UI)

- **Kontener**: `.opus` (OGG, zalecane do audio) / `.webm`.
- **Mapowanie kanałów**: `255 discrete` (zalecane dla AmbiX — bez downmixu, W,Y,Z,X 1:1) / `1 surround` / `0 mono-stereo`.
- **Bitrate**: 16–510 kb/s (dla 4-kan. FOA sensownie 128–256 k).
- **Tryb**: VBR / Constrained VBR / CBR.
- **Złożoność** 0–10, **długość ramki** 2.5–60 ms, **zastosowanie** audio/voip/lowdelay, **cutoff**.

Pod maską uruchamiane jest (przykład):
`ffmpeg -i in.wav -c:a libopus -mapping_family 255 -b:a 160k -vbr on -compression_level 10 -frame_duration 20 -application audio -f opus out.opus`
Aplikacja pokazuje dokładną komendę użytą dla danego pliku.

## Uwaga o mapowaniu ambisonicznym (family 2)

Natywne mapowanie ambisoniczne Opus (channel mapping family 2/3) nie jest wystawione przez ten
enkoder — dlatego domyślnie i zalecanie używamy **discrete (255)**, które i tak przenosi wszystkie
kanały bez zmian (sprawdzone: 4-kan. WAV → Opus discrete → 4-kan. z powrotem, na tej samej bibliotece
libopus). Do archiwum bezstratnego zostań przy WAV — Opus jest stratny.

## Self-hosting silnika (opcjonalnie, bez CDN)

Aby uniezależnić się od CDN, skopiuj obok `index.html` pięć plików do `vendor/`:
`ffmpeg.js`, `814.ffmpeg.js`, `util.js`, `core/ffmpeg-core.js`, `core/ffmpeg-core.wasm`
(z paczek `@ffmpeg/ffmpeg@0.12.10`, `@ffmpeg/util@0.12.1`, `@ffmpeg/core@0.12.6`) i otwórz stronę
z `?local=1`. Ta ścieżka jest wbudowana w kod (przełącznik na górze `<script>`).

## Status weryfikacji (uczciwie)

- Parametry i wynik kodowania sprawdzone na **natywnym libopus** (identyczna biblioteka co w ffmpeg.wasm):
  4-kan. WAV → 4-kan. Opus, poprawny round-trip do 4 kanałów, ~15:1 kompresji.
- Kod JS przechodzi kontrolę składni; wzorzec ładowania ffmpeg.wasm (`classWorkerURL`/`coreURL`/`wasmURL`
  przez `toBlobURL`) jest standardowy dla wersji 0.12.
- Pełnego kodowania „w przeglądarce" nie udało się odpalić w moim środowisku testowym (limit pamięci
  przy 31 MB WASM w trybie headless) — przetestuj u siebie otwierając `index.html`. Jeśli coś nie
  zagra, dawaj znać: mogę też zakodować konkretny plik od ręki natywnym ffmpeg.
