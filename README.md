# Chatter Remote

Chatter Remote is a Chrome extension for chatting with a Qwen3 LLM that runs on **your own server** through [OpenVINO Model Server (OVMS)](https://docs.openvino.ai/), from a side panel or floating popup. Answers stream in as they are generated. Nothing goes to a cloud service and no API key is needed.

It is the self-hosted sibling of [Key Chatter](https://github.com/paulchi-intel/key-chatter) and shares its look and workflow, together with [voice-clone-remote](https://github.com/paulchi-intel/voice-clone-remote) and [speech-to-text-remote](https://github.com/paulchi-intel/speech-to-text-remote).

## Features

- 💬 **Streaming chat** with the OVMS model, with a **Stop** button while it answers
- 🧠 **Thinking mode** switch — the model reasons before answering; the thought process is shown in a collapsible block
- 🌐 **Webpage, clipboard and YouTube-transcript context** — load it, then ask questions
- ⚡ **Quick questions** and reusable **saved prompts**
- 🗂️ Multiple persistent **conversation tabs**
- 📝 Markdown rendering and **Markdown session export**
- 🪟 Side-panel and floating-popup modes
- 🔔 Update notification from GitHub Releases
- 🌐 UI languages: 繁體中文, 简体中文, English (with 繁體中文, the model is asked to answer in Traditional Chinese)

---

## How it works

```
Chrome extension (your PC)
  sidepanel.js ──HTTP, SSE stream──▶  http://<SERVER_IP>:8003/v3/chat/completions
                                              │
                          server: OVMS container (e.g. ovms-qwen3-27b)
                          OpenVINO/Qwen3.8-27B-int4-ov on an Intel GPU
```

The extension uses OVMS's OpenAI-compatible API:

| Endpoint | Used for |
|---|---|
| `GET /v3/models` | find the served model and check the connection |
| `POST /v3/chat/completions` (`stream: true`) | chat; the model's thinking arrives in `delta.reasoning_content`, the answer in `delta.content` |

The thinking switch sends `chat_template_kwargs: {"enable_thinking": true|false}`.

There is nothing to install on the server: any OVMS instance serving a chat model works. OVMS already sends CORS headers, and the extension has host access, so no proxy is needed.

> **No authentication.** An OVMS port is open to anyone who can reach it. Keep it on a trusted network.

---

## Install

1. Download the latest ZIP from [GitHub Releases](https://github.com/paulchi-intel/chatter-remote/releases/latest).
2. Extract it to a permanent folder.
3. Open `chrome://extensions` and enable **Developer mode**.
4. Select **Load unpacked** and choose the extracted folder.
5. Click the Chatter Remote icon. On first run it asks for the **server address**, e.g. `http://<SERVER_IP>:8003`, and connects. The address is stored only in your browser (`chrome.storage`), never in this repository.

Change the address later from the **⚙** button on the status bar, or by clicking the **Chatter Remote** title.

## Use

1. Check the status bar says **Connected · \<model\>**. Click the status text to reconnect.
2. Optionally load the current webpage (🌐) or the clipboard (📋) as context. Very long content is cut to about 24k tokens and a notice tells you how much was kept.
3. Ask a question, or use a quick question or saved prompt.
4. Turn on **Thinking** for harder questions: better answers, but slower. Leave it off for quick answers.
5. Press **Stop** to cut an answer short. Stopping while the model is still thinking discards that turn and puts your question back in the box.
6. Use **+** for another conversation tab, **⊞/◫** to switch between side panel and popup, and **💾** to export the conversation as Markdown.

---

## Update

Chatter Remote checks the latest GitHub Release at startup and caches the result for six hours. When a newer version exists, a banner links to the download. To update: unzip the new ZIP over the existing folder, then click **Reload** on the extension in `chrome://extensions`. Conversations and settings stay in `chrome.storage.local`.

The check reads the Release without logging in, so it works because this repository is public.

## Release

1. Update `version` in `manifest.json` and commit it.
2. Create and push a matching tag, e.g. `git tag v1.0.0 && git push origin v1.0.0`.
3. GitHub Actions ([`release.yml`](.github/workflows/release.yml)) checks the tag matches `manifest.json` and that the JavaScript parses, then publishes `chatter-remote-vX.Y.Z.zip` and its SHA-256 checksum.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| "Set the server address first" | Click ⚙ (or the title) and enter the OVMS address |
| "Failed to load models: …" | Check the container is running and `curl --noproxy '*' http://<SERVER_IP>:8003/v3/models` answers |
| Works with curl but not from Chrome | Your PC's proxy may be intercepting the server address; add it to the proxy bypass list, or use an SSH tunnel (`ssh -L 8003:localhost:8003 <user>@<SERVER_IP>`) and set the address to `http://localhost:8003` |
| "(no answer)" message after a long thinking phase | The model used up its output length while thinking. Turn Thinking off or shorten the question |
| Answer cuts off or the server returns an error with a huge page | The loaded content plus conversation exceeded the model's context; load a smaller page or start a new tab |

---

## License

MIT — see [LICENSE](LICENSE)
