# Voiceover-Einstellungen (ElevenLabs) – Stand 16.11.2026

Gilt für alle Reels ab 2026-11-16. Schlüssel und Voice-ID liegen im Claude-Projekt „Buch schreiben“ („eleven api“, „eleven stimme“), nie im Repo.

## TTS
- Endpoint: `POST /v1/text-to-speech/<voice_id>?output_format=mp3_44100_128`
- `model_id`: `eleven_multilingual_v2`
- `voice_settings`: stability 0.55, similarity_boost 0.75, style 0.3 (Schlusssatz: 0.2), speed 0.95

## Pausen (Methode: break-Tags – funktionieren mit multilingual_v2 sauber)
- Hauptteil in EINEM Request, Sätze getrennt durch SSML-Breaks:
  - normal zwischen Sätzen: `<break time="0.4s" />` (ergibt real ca. 0,5–0,8 s)
  - nach Hook-Sätzen und vor Auflösungen („Ist es aber nicht.“, „WAHR.“, „ERFUNDEN.“): `<break time="0.7s" />` (real ca. 0,9–1,1 s)
- Schlusssatz „Stille Quelle. Jetzt auf Amazon.“ separat erzeugen (style 0.2, `previous_text` = letzter Satz des Hauptteils), mit **1,0 s Stille** davor anhängen (ffmpeg concat).
- In der Pause: Herzschlag (`sq/assets/sfx/herzschlag.mp3`, ca. −24 LUFS) + Atmo auf halbe Lautstärke abgesenkt.
- Prüfen: `silencedetect=n=-40dB:d=0.3` → Pause vor „Stille Quelle“ ≥ 0,8 s.

## Mix / Export
- Voice beginnt 0,4 s nach Videostart; Ausklang 1,3 s nach „Amazon“.
- Atmo ca. 20 dB unter der Stimme (−34 LUFS vor Endnormalisierung), Ein-/Ausblendung.
- Kompressor + Limiter, Ziel −14 LUFS integriert, Peak ≤ −1,2 dBFS.
- H.264 `-preset veryfast -crf 21`, AAC 160k, 1080×1920, 30 fps, < 20 MB.
- Untertitel: genau eine Textebene, Satz für Satz, bleibt in den Pausen stehen; Schlusskarte (Cover + „Jetzt auf Amazon“) blendet mit Beginn der Pause vor „Stille Quelle“ ein, ohne Untertitel.

## Mischung ohne Rauschen (seit 18.11.2026, Timos Wunsch)
- Atmos sind nur zulässig, wenn sie klar erkennbar sind (Vögel, Regen, Wellen) und kein gleichmäßiges Breitbandrauschen, Brummen oder Pfeifen enthalten. Jede neue Datei per Spektrogramm (showspectrumpic) prüfen; durchgehende waagrechte Linien oder eine gleichmäßige „Rauschfläche“ = verwerfen.
- Atmo höchstens −28 dB unter der Stimme mischen, mit Tiefpass (~8 kHz) und Hochpass (~80 Hz).
- Prüfung am fertigen Reel: In jeder Sprechpause (silencedetect auf voice.mp3) muss der Mix leiser als −40 dBFS RMS sein. Sonst Atmo leiser oder weglassen. Lieber keine Atmo als Rauschen.
- Lautheitsnormalisierung (loudnorm) nur auf die Stimme vor dem Mischen anwenden, nicht auf den fertigen Mix, damit Pausen nicht hochgezogen werden.
