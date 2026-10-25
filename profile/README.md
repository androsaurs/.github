# 🦖 Androsaur

Old Android phones are dinosaurs. We bring them back to life.

Androsaur turns phones from your drawer into a home assistant on a motorized
stand: it wakes up on a word, turns to face you, sees, listens and talks.
It is built as a colony of small apps — each one useful on its own,
together one creature. Every module speaks plain HTTP and MCP, so an AI agent
can use it directly. A small home server ties the phones of a house together.

| Module | What it does | Status |
| --- | --- | --- |
| [androsaur](https://github.com/androsaurs/androsaur) | Main app on screen: the face and its reflexes (attentive, sad, asleep, dims the screen), knows who is in front, delivers voice notes to the right person | working prototype |
| [androsaur-audio](https://github.com/androsaurs/androsaur-audio) | Microphone and voice: wake word with switchable engines, offline Russian speech, voice notes left by a spoken command, MCP | working prototype |
| [androsaur-camera](https://github.com/androsaurs/androsaur-camera) | Camera service: face tracking, recognizes taught people by a permanent id, document shots, MCP | working prototype |
| [androsaur-hub](https://github.com/androsaurs/androsaur-hub) | Home server (FastAPI): admin page for settings and people, syncs faces between phones — a new phone recognizes the family without teaching | first version, home network only |
| [androsaur-stand](https://github.com/androsaurs/androsaur-stand) | Motorized stand: FreeCAD model of the body, pan and tilt | first CAD mock-up |

**What works today** (measured on a Xiaomi Redmi 12, Android 15): say the wake word and the face wakes up, recognizes who
said it and listens for a command — «передай жене…» records a voice note, and the phone plays it when that person walks up.
When nobody is around the face gets sad, then falls asleep and dims the screen. Faces are kept as numbers, never as photos,
on the phones and the home server only. For the details, APIs and what is not done yet, see
[CONTEXT.md](https://github.com/androsaurs/.github/blob/main/CONTEXT.md) (written for AI agents).

Старые Android-телефоны — динозавры. Мы возвращаем их к жизни.
Сейчас на одном телефоне: фраза будит лицо, оно узнаёт, кто его разбудил, и слушает команду — «передай жене…» записывает голосовую
записку, а телефон проиграет её, когда этот человек подойдёт. Когда никого нет, лицо грустит, потом засыпает и гасит экран. Домашний
сервер хранит настройки и людей: новый телефон узнаёт семью без обучения. Подробности и API для ИИ-агентов — в
[CONTEXT.md](https://github.com/androsaurs/.github/blob/main/CONTEXT.md).
