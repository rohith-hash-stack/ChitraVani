# Phase 0 Feasibility Report — Telugu AI Voice Agent (ChitraVani)

**Date checked:** 2026-10-26
**Status:** Investigation only. No architecture is committed and no accounts, carrier settings or paid services have been touched.

## How the claims below were verified

- **Container facts** come from commands run in this session (commands and outputs in §1).
- **Provider and licence facts** come from web searches made on 2026-10-26. The search results quoted official pages (docs.oracle.com, render.com/docs, huggingface.co/docs, docs.cloud.google.com, core.telegram.org, developers.cloudflare.com).
- **Limit of that verification:** direct page fetches of those domains failed from this environment (DNS blocked), so I could not open each official page myself. Each claim is marked:
  - **[official]** when the search quoted an official page.
  - **[secondary]** when only news, blogs or forums support it.
  - **[unverified]** when I could not confirm it.
- Re-check every **[secondary]** or **[unverified]** item before acting on it.

---

## 1. Development container

| Item | Measured result | Command |
|---|---|---|
| OS | Ubuntu 24.04.5 LTS, Linux 6.18 x86_64 | `cat /etc/os-release; uname -a` |
| Python | 3.13.17 (system), pip 24.0 | `python3 --version` |
| CPU | 4 vCPU Intel Xeon @ 2.80 GHz, AVX2, AVX-512 (incl. VNNI) | `nproc; lscpu` |
| RAM | 15 GiB total, no swap | `free -h` |
| Disk | 31 GB available | `df -h` |
| GPU | None (`nvidia-smi` not found) | — |
| Audio devices | None (`/dev/snd` absent): no microphone or speaker | `ls /dev/snd` |
| Tools | ffmpeg 6.1.1, Docker 29.9.2, onnxruntime 1.29 (system), Chromium + Playwright preinstalled | — |
| Python packages (PyPI) | **Works.** fastapi 0.115.14, onnxruntime 1.31.0, sherpa-onnx 2.0.1, onnx-asr, soundfile 0.14.0 installed in a venv (222 MB) | `pip install …` |
| Model downloads | **Blocked.** `huggingface.co`, `github.com` (releases), `ollama.com` and `vaani.iisc.ac.in` all return proxy 403 (network policy) | `curl -w '%{http_code}'` |
| Persistence | **Ephemeral.** The container was fresh (uptime 4 min). Only what is committed and pushed to git survives. | `uptime`, empty repo |

### What can be tested where

| Capability | In this container | Needs deployed host / real device |
|---|---|---|
| Dialogue manager, lead validation, SQLite, API | ✅ | — |
| Browser UI audio path | ✅ Headless Chromium can feed a WAV file as a fake microphone | Real phone microphone, mobile browsers |
| Telugu ASR/TTS inference | ⚠️ Only after `huggingface.co` and `github.com` are allow-listed | ARM latency on the real host |
| Latency figures | x86 AVX-512 numbers **will not transfer** to a 2-core ARM host | Must be re-measured on the host |
| HTTPS, persistence across restart, sleep behaviour | ❌ | ✅ |
| Phone calls (PSTN) | ❌ | ❌ (see §5) |

**Blocker for Phase 2 in this environment:** the network policy denies `huggingface.co` and `github.com` release downloads. To test real models here, add `huggingface.co`, `cdn-lfs.huggingface.co`, `github.com` and `objects.githubusercontent.com` under **Allowed domains** in the environment's Network access settings. Steps: <https://code.claude.com/docs/en/cloud-environments#network-access>. Until then, model inference cannot be tested and I will not substitute mock results for it.

---

## 2. Zero-cost hosting comparison

| Provider / tier | Resources | Always-on? | Disk | Card needed | Verdict |
|---|---|---|---|---|---|
| **Oracle Cloud Always Free, Ampere A1** | **2 OCPU / 12 GB RAM** (cut from 4/24 in June 2026) **[official + secondary]**. 1,500 OCPU-h and 9,000 GB-h per month **[official]** | Yes, but idle VMs can be reclaimed: 7-day 95th-percentile CPU, network *and* memory all under 20% **[official]** | Persistent block volume | **Yes.** Credit card or credit-like debit card for verification. Not charged unless upgraded. No prepaid or virtual cards **[official]** | **Only candidate that can run ASR+TTS.** Conditional (see below). |
| Hugging Face Spaces, CPU Basic | 2 vCPU / 16 GB, 50 GB non-persistent disk **[official]** | Pauses after 48 h idle **[official]** | Ephemeral | — | **Not ₹0 any more.** Docker/Gradio Spaces on cpu-basic now need a PRO subscription; creation fails with HTTP 402 **[official]**. Only Static Spaces are free. |
| Render Free web service | 512 MB RAM **[official]** | Spins down after 15 min idle; about 1 min cold start; 750 h/month **[official]** | Ephemeral; free Postgres expires after 30 days, then deleted **[official]** | No | Too little RAM for ASR+TTS. Usable only for a text-only enquiry form. |
| Google Cloud e2-micro | 1 GB-class VM, US regions only **[official]** | Yes | 30 GB persistent disk **[official]** | Billing account required **[official]** | Too small, and US-only means high latency from India. |
| Koyeb free | 512 MB / 0.1 vCPU, scales to zero **[secondary]** | No | — | Card-free Hobby plan reportedly removed Feb 2026 **[secondary]** | Not viable. |
| Fly.io | No free tier for new accounts **[secondary]** | — | — | Yes | Not viable. |
| Cloudflare Quick Tunnel + device at home | Uses the artist's own hardware | Only while that device is on | Local | No account needed **[official]** | ₹0 software. Electricity, internet and a spare device are needed. Random URL changes on every restart. Fallback only. |

### Oracle conditions you must accept or reject

1. **Card for sign-up.** Your policy excludes "credit-card-dependent trials". Always Free is not a trial, but sign-up needs a card and a temporary verification hold. One account is allowed per person **[official]**.
2. **Capacity.** A1 capacity in Mumbai or Hyderabad is reportedly sometimes unavailable **[secondary]**. The home region is permanent **[official]**.
3. **No SLA, no Oracle Support** for Always Free-only customers **[official]**. Oracle cut the allowance in half in 2026 without public notice **[secondary: InfoQ, heise]**. It could change again.
4. **Idle reclamation.** A quiet voice agent will usually sit below 20% utilisation. Oracle says it "may" reclaim such instances. Mitigation options need checking against Oracle's terms first.
5. **Commercial use.** I found no explicit Always Free statement allowing or forbidding commercial use **[unverified]**. Read the Oracle Cloud Services Agreement before going live.

**HTTPS needs a stable hostname** because browsers only allow microphone access over HTTPS. Free options to verify later: a DuckDNS subdomain with Let's Encrypt, or a Cloudflare named tunnel (Zero Trust free plan; whether it needs a payment method is **[unverified]**).

---

## 3. Models: Telugu support, licence and footprint

**About these sizes:** none are measured. Downloads are blocked, so the sizes come from published parameter counts and are marked *est.*

| Role | Candidate | Telugu | Licence | Size / CPU estimate | Decision |
|---|---|---|---|---|---|
| ASR | **AI4Bharat IndicConformer `indicconformer_stt_te_hybrid_ctc_rnnt_large`** | Telugu-specific | **MIT** **[official: AI4Bharat API, IndiaAI AIKosh]** | 120 M params, 16 kHz mono. Community ONNX int8 conversion exists (`OpenVoiceOS/ai4bharat-indicconformer-te-onnx`; its own licence is **[unverified]**). Est. ~130 MB int8, ~0.5 GB RAM | **Primary ASR candidate** |
| ASR | `vasista22/whisper-telugu-tiny` (and small/base) | Telugu fine-tune | Apache-2.0 for tiny **[official card]**; others **[unverified]** | 39 M params; FLEURS-te WER 17.15 (tiny, model card). An int8 sherpa-onnx pack exists (99 MB) | Fallback / comparison |
| ASR | IISc-ARTPARK **SraVaani** (Project Vaani) | 63 languages including Telugu | MIT reported, but **gated** download **[secondary]** | ~430 M params: heavier on 2 ARM cores | Evaluate later if accuracy demands it |
| TTS | **AI4Bharat Indic-TTS** (FastPitch + HiFi-GAN, Telugu) | Yes (13 languages) | Repo **MIT** **[official GitHub About]**. Telugu checkpoint file `te.zip` **[unverified]** | Est. tens of millions of params; needs Coqui-TTS + PyTorch (~1 GB RAM). ONNX export to be attempted | **Primary TTS candidate** (licence of the checkpoint to confirm) |
| TTS | AI4Bharat **Indic Parler-TTS** | Yes | Apache-2.0, gated **[secondary]** | ~0.9 B params, autoregressive; "GPU highly recommended" | Too slow for 2 CPU cores (est.) |
| TTS | Meta **MMS-TTS** (`mms-tts-tel`) | Yes | **CC-BY-NC 4.0** on the MMS collection **[official HF tags]** | Small | **Excluded: non-commercial licence** |
| TTS | Piper | No verified Telugu voice found | MIT engine, per-voice licences | — | Not usable now |
| LLM | Gemma 3 1B / Qwen3 0.6–1.7B / Sarvam-1 (GGUF) | No Telugu benchmark found | Gemma terms / Apache-2.0 / **[unverified]** | Q4 est. 0.5–1.2 GB RAM. Telugu tokenises poorly, so generation is slow on 2 cores | **Not in the critical path** (see §4) |

### Estimated RAM on Oracle A1 (12 GB)

| Component | Estimated RAM |
|---|---|
| OS | ~0.5 GB |
| FastAPI | ~0.2 GB |
| ASR | ~0.5 GB |
| TTS with PyTorch | ~1.5 GB |
| Optional 1–2 B LLM | ~1.5 GB |
| **Total** | **~4–5 GB** |

- **Fits in 12 GB.** It would not fit on Render (512 MB) or GCP e2-micro.
- **CPU is the constraint, not RAM.** I estimate a few seconds per turn on 2 ARM cores. This has to be measured, so I make no "real-time" claim.

---

## 4. Recommended architecture (pending your approval)

```
Phone browser (HTTPS page, push-to-talk; mic via getUserMedia)
   │  WAV/WebM upload per turn (HTTPS POST; WebSocket only if latency later justifies it)
   ▼
FastAPI on Oracle A1 (Mumbai/Hyderabad)
   ├─ audio validation (format, 16 kHz mono, size/duration limits) → temp file, deleted after use
   ├─ ASR adapter  → IndicConformer-te ONNX int8   (swappable: whisper-telugu-tiny)
   ├─ Conversation manager (deterministic state machine + slot extraction, Telugu/English keywords)
   │     business facts only from config; unknown = "artist will confirm"
   ├─ TTS adapter  → Indic-TTS Telugu (cached audio for fixed prompts)
   ├─ Lead store   → SQLite on persistent block volume, validated with pydantic
   └─ Notifier     → console (dev); Telegram Bot API (free, opt-in) for production
Future: Telephony adapter → same conversation manager (not ₹0, see §5)
```

### Why the dialogue manager is deterministic, not LLM-based

1. Your rules forbid invented prices, dates and promises. A rules-based manager drawing only on configured facts makes those guarantees testable.
2. Small local LLMs have no verified Telugu quality.
3. On 2 ARM cores, Telugu generation would dominate latency.

An optional LLM can be added later for paraphrasing or slot extraction, behind validation, if testing shows a need.

### Fallbacks

- **Text-only enquiry form.** This works even when the models are down.
- **If you reject the Oracle card requirement,** the closest honest ₹0 option is:
  - A text-only form, on Render free or a static page with a small backend.
  - A voice backend on a spare device at home through a Cloudflare tunnel, available only while that device runs.

### Notifications

| Channel | Status |
|---|---|
| Telegram Bot API | Free. Limit of about 30 messages/second for bulk sending **[official FAQ]**. The artist needs a Telegram account, and lead text passes through Telegram's servers. |
| SMS, WhatsApp Business API | Not assumed free. Not used. |

---

## 5. Objective B — answering calls to the existing mobile number

**Conclusion: a ₹0-recurring route could not be verified, and it is very likely impossible.** Every route needs at least one recurring paid item.

| Route | Blocker |
|---|---|
| Cloud telephony (Plivo, Exotel) with call forwarding | Plivo India: ₹250/month number rental plus about ₹0.40–0.60/min **[official pricing page; two rate cards conflict]**. Trial credit is not permanent. Exotel uses paid bill plans **[official]**. |
| Free SIP account + forwarding | Indian rules restrict VoIP–PSTN interconnection for internet-service licensees **[secondary: older sources]**. No free Indian inbound DID was found. |
| Spare Android phone as gateway | Third-party apps cannot capture call audio (`VOICE_CALL` needs `CAPTURE_AUDIO_OUTPUT`, system apps only) **[secondary]**. Needs root or system privileges. Not reliable. |
| GSM module (SIM800/SIM7600) + Raspberry Pi at home | One-time hardware cost. A second SIM needs **recurring prepaid recharge** to stay active. Needs always-on power and internet at home. SIM-box style setups are a regulatory grey area in India **[secondary]**: check with the carrier first. |
| Call forwarding cost | Jio help reportedly says forwarding is free on an active plan. Airtel charges for some forwarding cases **[secondary]**. Forwarded legs are often billed as outgoing. **Must be confirmed with your carrier**, and the destination still costs money. |

**Closest ₹0 alternative:** publish the web-agent link where customers already reach the artist, for example:
- the WhatsApp Business app greeting or away message (the free app, not the paid API; check the feature yourself), or
- the voicemail greeting.

Nothing will be enabled without your approval.

---

## 6. Cost table

| Item | Guaranteed ₹0 | Conditional / possibly chargeable |
|---|---|---|
| Code, models (MIT/Apache), FastAPI, SQLite, onnxruntime | ✅ | Indic-TTS checkpoint and ONNX conversion licences still to confirm |
| Hosting | — | Oracle Always Free: card required, capacity, reclamation, terms can change |
| Domain/HTTPS | DuckDNS + Let's Encrypt (to verify) | Custom domain (paid) not required |
| Notifications | Console; Telegram bot | SMS / WhatsApp API: paid, not used |
| Telephony (Objective B) | ❌ | Number rental, per-minute charges, SIM recharge, forwarding charges |
| Development container | Provided | Container is ephemeral |

---

## 7. Decisions I need from you

1. **Oracle Cloud:** do you accept a card verification at sign-up (no charge unless you upgrade)?
   - If **yes**, Oracle A1 in Mumbai/Hyderabad becomes the target host.
   - If **no**, no verified free host can run Telugu ASR+TTS, and we use the fallback in §4.
2. **Allow-list model hosts** (`huggingface.co`, `cdn-lfs.huggingface.co`, `github.com`, `objects.githubusercontent.com`) in this environment so I can measure real model accuracy, latency and RAM.
3. **Objective B:** confirm it is deferred and documented as not achievable at ₹0.

**Phase 1** (deterministic Telugu dialogue manager, validated SQLite leads, text mode, more than 30 scripted scenarios) does not depend on any of these decisions. I can start it next.

---

## Sources (accessed 2026-10-26)

**Oracle Cloud**
- [Always Free Resources](https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm) · [resourceref](https://docs.oracle.com/en-us/iaas/Content/FreeTier/resourceref.htm)
- [Free Tier FAQ](https://www.oracle.com/cloud/free/faq/) · [Sign-up](https://docs.oracle.com/en-us/iaas/Content/GSG/Tasks/signingup_topic-Sign_Up_for_Free_Oracle_Cloud_Promotion.htm) · [Oracle India Free Tier](https://www.oracle.com/in/cloud/free/)
- News on the 2026 cut: [InfoQ](https://www.infoq.com/news/2026/07/oracle-cloud-free-tier-limits/) · [heise](https://www.heise.de/en/news/Oracle-halves-free-cloud-resources-11334516.html)

**Other hosting**
- Hugging Face: [Manage Spaces (402 / PRO requirement)](https://huggingface.co/docs/huggingface_hub/main/en/guides/manage-spaces) · [Spaces overview](https://huggingface.co/docs/hub/spaces-overview) · [forum thread](https://discuss.huggingface.co/t/new-free-accounts-cannot-create-cpu-basic-gradio-spaces-only-zerogpu-available/177629)
- Render: [Deploy for Free](https://render.com/docs/free)
- Google Cloud: [Free features](https://docs.cloud.google.com/free/docs/free-cloud-features?hl=en)
- Koyeb / Fly.io (secondary): [free-tier comparison Sep 2026](https://flaviocopes.com/hosting-free-tiers.md) · [agentdeals](https://agentdeals.dev/hosting-free-tier-comparison-2026)
- Cloudflare: [Tunnels FAQ](https://developers.cloudflare.com/cloudflare-one/faq/cloudflare-tunnels-faq/) · [Quick Tunnels](https://try.cloudflare.com/)

**Models**
- IndicConformer: [AIKosh entry](https://aikosh.indiaai.gov.in/home/models/details/ai4bharat_indicconformer_stt_te_hybrid_ctc_rnnt_large_telugu_automatic_speech_recognition_model.html) · [AI4Bharat API](https://admin.models.ai4bharat.org/models/IndicConformer/?format=api) · [ONNX conversion](https://huggingface.co/OpenVoiceOS/ai4bharat-indicconformer-te-onnx)
- Whisper Telugu: [whisper-telugu-tiny](https://huggingface.co/vasista22/whisper-telugu-tiny/blob/refs%2Fpr%2F1/README.md?code=true) · [sherpa-onnx pack](https://huggingface.co/ippocode/indic-asr-onnx)
- SraVaani: [SraVaani](https://huggingface.co/ARTPARK-IISc/SraVaani) · [news](https://letsdatascience.com/news/iisc-team-releases-sravaani-asr-for-65-indian-languages-a6aa9c5b)
- Indic-TTS: [Indic-TTS](https://github.com/AI4Bharat/Indic-TTS) · [checkpoint release](https://github.com/AI4Bharat/Indic-TTS/releases/tag/v1-checkpoints-release)
- Other TTS: [Indic Parler-TTS](https://huggingface.co/ai4bharat/indic-parler-tts) · [MMS-TTS licence tag](https://huggingface.co/facebook/mms-tts/tree/main)

**Notifications**
- [Telegram Bots FAQ](https://core.telegram.org/bots/faq)

**Telephony**
- Pricing: [Plivo India pricing](https://www.plivo.com/voice/pricing/in/) · [Exotel incoming charges](https://docs.exotel.com/business-phone-system/why-am-i-charged-for-incoming-calls)
- Call forwarding: [forwarding guide (secondary)](https://getautobot.app/blog/call-forwarding-codes)
- Regulation and Android limits: [VoIP routes (secondary)](https://en.wikipedia.org/wiki/White,_black_and_grey_routes) · [Android audio capture](https://developer.android.com/media/platform/sharing-audio-input) · [MITRE T1429](https://redteam.y-security.de/MITRE/techniques/T1429/)
