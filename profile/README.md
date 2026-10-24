# 🦖 Androsaur

Old Android phones are dinosaurs. We bring them back to life.

Androsaur turns phones from your drawer into a home assistant on a motorized
stand: it wakes up on a word, turns to face you, sees, listens and talks.
It is built as a colony of small apps — each one useful on its own,
together one creature. Every module speaks plain HTTP and MCP, so an AI agent
can use it directly.

| Module | What it does | Status |
| --- | --- | --- |
| [androsaur](https://github.com/androsaurs/androsaur) | Main app on screen: the face, reflexes, shows who woke it | skeleton works |
| [androsaur-camera](https://github.com/androsaurs/androsaur-camera) | Camera service: front/back camera, shots, face tracking, recognizes up to 3 taught people, MCP | working prototype |
| [androsaur-audio](https://github.com/androsaurs/androsaur-audio) | Microphone service: wake word with switchable engines and a phrase you can change over the API, MCP | working prototype |
| androsaur-stand | Motorized stand: ESP32 firmware and 3D models | planned |

**What works today** (measured on a Xiaomi Redmi 12, Android 15): say the wake word and the face on the screen wakes up and,
a moment later, shows the name of the person who said it — recognized on the phone from a dozen samples of their face taken once while teaching;
otherwise the eyes just follow whoever is in front. Nothing leaves the phone. For the details, APIs and what is not done yet, see
[CONTEXT.md](https://github.com/androsaurs/.github/blob/main/CONTEXT.md) (written for AI agents).

Старые Android-телефоны — динозавры. Мы возвращаем их к жизни.
Всё уже работает на одном телефоне: фраза будит лицо на экране, оно называет того, кто его разбудил, а пока никто не говорит —
следит глазами за человеком. Подробности и API для ИИ-агентов — в [CONTEXT.md](https://github.com/androsaurs/.github/blob/main/CONTEXT.md).
