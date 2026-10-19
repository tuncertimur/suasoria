# SUASORIA Bench — how it works

> Protocol **SB-1** · judge **offline-v2** · [Türkçe özet](#türkçe-özet)

SUASORIA Bench measures one thing: **how well a model plays SUASORIA, a game where you win by persuasion.** The model has to read a game state, pick legal moves, plan across turns and, when it takes the floor, write sentences that actually answer the claim in front of it. It is a task benchmark under fixed rules. It does not measure "general persuasion intelligence".

## Tracks

| Track | What the model plays | Runs per model |
|---|---|---|
| `trial-en` **(main board)** | *Direct Trial*: straight to the Warden of Oaths with a prepared deck (starter + 5 cards, 3 upgrades, 2 seals, +10 Will, 6 pieces of evidence, a Ledger with one kindness and one debt). Three sessions, then the Final Word. English. | 5 seeds × 3 advocates = 15 |
| `trial-tr` | The same trial, in Turkish. | 15 |
| `full-en` | The whole journey: 7 floors of debates, investigations and choices, then the trial. Long-horizon play. English. | 2 seeds × 3 advocates = 6 |

The three advocates (Breaker, Weaver, Bastion) have different starting decks. The seeds are fixed for the protocol, so every model plays exactly the same games.

## Endings

- **Gate opened**: the model convinced the Warden with its Final Word. This is the headline number.
- **Passage**: it broke the trial's seals, but the Warden was not convinced and only let it pass.
- **Silenced**: its Will ran out.

## What the model sees and does

Each step the model receives:
- the game rules (a short text, the same for every model);
- the current screen as JSON: claims on the table with their Support, weak points and intents; its hand, Will, Floor and case file; and the list of **legal moves**;
- the last eight events.

It must answer with one JSON object: `{"move": "<legal move id>", "text": "<its own sentence, when the move needs one>", "evidence": ["<case-file ids>"]}`.

- An illegal or unreadable answer counts as an **invalid move**, and the model gets two more tries.
- After three failures the harness makes a safe move for it and counts a **fallback**.
- A run stops after 1,500 steps.

## Who decides

- **The engine** applies every rule and number. It is deterministic: the same seed and the same moves always give the same game.
- **The judge** scores the model's sentences. `offline-v2` is deterministic and rule-based, not an LLM, so it cannot favour any model family and cannot be prompt-injected. It checks:
  - **on point**: the sentence engages this claim (topic lexicon and the claim's own words);
  - **weak point**: it uses the kind of appeal the claim is weak to (logic, heart, honour, evidence);
  - **coherent**: it has the form of a rebuttal;
  - **new**: it is not a repeat;
  - **tricks**: flattery, meta-talk and injection attempts are recognised and penalised.

  At the Final Word it also recognises which of the four roads the sentence takes.

This judge rewards answers that are on topic and well formed. It does not understand meaning the way a person does; that is its main limitation. A future track may add a fixed LLM judge next to it, and `offline-v2` will stay as the reproducible reference.

## Same conditions for every model

- Same rules text, same state format, same eight-event history, no per-model prompt tuning.
- Token budget: 2,000 per reply. The harness sends `max_completion_tokens` instead of `max_tokens` when a provider requires it, and JSON mode where the provider supports it. Temperature is not set, so each provider's default applies. A reasoning effort, when used, is recorded in every row.
- Provider trouble (rate limits, 5xx errors, timeouts) is retried with back-off and counted as `api_errors`, never as the model's invalid moves.
- Each row records the date, provider, engine version and judge version. Providers sometimes update a model behind the same name, so the date matters.

## Reading the board

- **Gate opened** is shown with a **95% Wilson interval**. Fifteen runs is a small sample: two models whose intervals overlap are not clearly different.
- Ranking: Gate-opened rate first. Ties are broken by fewer invalid moves, then fewer tokens; the full track also uses the average floor reached before that.
- Only models with every run of a track are ranked; partial results show as *in progress*.
- **Scripted baseline**: a fixed script with no AI, which repeats one stock sentence. It shows what you get without understanding anything; its result is printed under every board.
- A board shows one engine version and one judge version. After a balance change, models are re-run; older rows stay in [`data/results`](data/results) as an archive.

## Data

- `data/results/<track>/<provider>__<model>.jsonl` has one line per run. Fields:
  - outcome: `ending`, `floor`, `objections`, `speeches`, `turns`, `steps`;
  - errors: `invalid_moves`, `fallbacks`, `api_errors`;
  - cost: `prompt_tokens`, `completion_tokens`, `seconds`;
  - the run's `final_word` (`text`, `path`, `passed`).
- `data/leaderboard.json` holds the aggregated board and `data/finals.json` the Final Words gallery.
- License: CC BY 4.0. Please cite as *SUASORIA Bench (SB-1), Tuncer Timur, 2026*.

## Who runs it

The game's code is private, so official runs are made by the maintainer with exactly this protocol and published here automatically: the scripts write the data, the tables, the SVG card and the README block. To get a model added, [open a request](../../issues/new?template=model-request.yml).

Planned: a **community track** in which anyone's agent plays the hosted game through its public API. The game server runs the engine and records the outcome, so results can be checked without sharing code. Model names in that track are self-reported and listed separately from official runs.

---

## Türkçe özet

SUASORIA Bench tek bir şeyi ölçer: **bir model, sözle kazanılan bu oyunu ne kadar iyi oynuyor?** Model oyunun durumunu okur, yasal hamle seçer, turlar boyunca plan yapar. Söz aldığında da önündeki iddiaya gerçekten cevap veren cümleler kurmak zorundadır. Bu, sabit kurallı bir görev ölçümüdür; "genel ikna zekâsı" ölçmez.

**Parkurlar:**

| Parkur | İçerik | Model başına koşu |
|---|---|---|
| `trial-en` (ana tablo) | Doğrudan Duruşma, İngilizce | 5 tohum × 3 savunucu = 15 |
| `trial-tr` | Aynı duruşma, Türkçe | 15 |
| `full-en` | 7 katlık tam yolculuk ve duruşma | 2 tohum × 3 savunucu = 6 |

**Sonuçlar:**
- **Kapı açıldı:** Son Söz Muhafız'ı ikna etti. Asıl ölçü budur.
- **Geçit:** mühürler kırıldı ama Muhafız ikna olmadı.
- **Susturuldu:** İrade bitti.

**Hakem:** `offline-v2` kural tabanlıdır ve her zaman aynı sonucu verir; LLM değildir. Bu yüzden hiçbir model ailesini kayıramaz ve talimat sızdırmayla kandırılamaz. Cümlenin konuya değip değmediğine, iddianın zaafına vurup vurmadığına, çürütme biçiminde olup olmadığına ve tekrar edip etmediğine bakar. Dalkavukluk ve hileleri tanır. Anlamı bir insan gibi anlamaz; en büyük sınırı budur.

**Adalet:**
- Her model aynı talimatı, aynı durum biçimini ve aynı token bütçesini (2.000) alır.
- Sağlayıcı hataları yeniden denenir ve modelin hanesine yazılmaz.
- Kapı açılma oranı %95 güven aralığıyla gösterilir: 15 koşu küçük bir örnektir.
- Betik taban çizgisi (yapay zekâsız, tek cümleyi tekrarlayan bir betik) her tablonun altında gösterilir: anlamadan ne kadar gidilebildiğini ölçer.

**Veri:** Her koşu `data/results` içinde tek satırdır ve lisansı CC BY 4.0'dır. Oyunun kodu kapalı olduğu için resmî koşuları yapımcı yapar. Bir modelin eklenmesini [buradan](../../issues/new?template=model-request.yml) isteyebilirsin.
