# 🦖 Androsaur

Old Android phones are dinosaurs. We bring them back to life.

Androsaur turns phones from your drawer into a home assistant on a motorized
stand: it wakes up on a word, turns to face you, sees, listens and talks.
It is built as a colony of small apps — each one useful on its own,
together one creature. Every module speaks plain HTTP and MCP, so an AI agent
can use it directly. A small home server ties the phones of a house together.

| Module | What it does | Status |
| --- | --- | --- |
| [androsaur](https://github.com/androsaurs/androsaur) | Main app on screen: the face and its reflexes (attentive, sad, asleep, dims the screen), clock and weather, knows who is in front, delivers voice notes to the right person | working prototype |
| [androsaur-audio](https://github.com/androsaurs/androsaur-audio) | Microphone and voice: wake word with switchable engines, offline Russian speech, voice notes left by a spoken command, MCP | working prototype |
| [androsaur-camera](https://github.com/androsaurs/androsaur-camera) | Camera service: face tracking, recognizes taught people by a permanent id, document shots, MCP | working prototype |
| [androsaur-hub](https://github.com/androsaurs/androsaur-hub) | Home server (FastAPI): admin page for settings, people and weather, syncs faces between phones — a new phone recognizes the family without teaching | first version, home network only |
| [androsaur-stand](https://github.com/androsaurs/androsaur-stand) | Motorized stand: FreeCAD model of the body, pan and tilt | first CAD mock-up |

**What works today** (measured on a Xiaomi Redmi 12, Android 15): say the wake word and the face wakes up, recognizes who
said it and listens for a command — «передай жене…» records a voice note, and the phone plays it when that person walks up;
«который час», «какая погода завтра», «таймер на десять минут», «сколько будет семь умножить на восемь» are answered at once,
without an LLM or a server. With the home server, voice notes are transcribed (Groq) and read out by the synthesizer. At the first
meeting of the day it says good morning by name, with the weather and an umbrella hint.
Above the face: the time, the date and the weather — from the home server when it is there, fetched by the phone itself
when it is not. When nobody is around the face gets sad, then falls asleep and dims the screen. Faces are kept as numbers, never as photos,
on the phones and the home server only. For the details, APIs and what is not done yet, see
[CONTEXT.md](https://github.com/androsaurs/.github/blob/main/CONTEXT.md) (written for AI agents).

**License:** the code is [AGPL-3.0](https://www.gnu.org/licenses/agpl-3.0.html) — use, change and share it, but any modified
version, including one run as a service, must stay open under the same license; the stand's models and drawings are
[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) — non-commercial. For commercial use, contact the author.

Старые Android-телефоны — динозавры. Мы возвращаем их к жизни.
Сейчас на одном телефоне: фраза будит лицо, оно узнаёт, кто его разбудил, и слушает команду — «передай жене…» записывает голосовую
записку, а телефон проиграет её, когда этот человек подойдёт; «который час», «какая погода завтра», «таймер на десять минут»,
«сколько будет семь умножить на восемь» Петя отвечает сразу, без LLM и сервера. С домашним сервером записки расшифровываются
(Groq) и читаются синтезатором. При первой встрече за день он здоровается по имени и говорит погоду. Над лицом — часы, дата и погода (с домашнего сервера, а без него телефон берёт её сам). Когда никого нет, лицо грустит, потом
засыпает и гасит экран. Домашний
сервер хранит настройки и людей: новый телефон узнаёт семью без обучения. Подробности и API для ИИ-агентов — в
[CONTEXT.md](https://github.com/androsaurs/.github/blob/main/CONTEXT.md).
