# SUASORIA Bench — how it works

> **SUASORIA Arena** (main) · humans × models · [jump](#suasoria-arena)
> **SB-1** · models as players · protocol **SB-1**, judge **offline-v2** · [jump](#sb-1-models-as-players)
> [Türkçe özet](#türkçe-özet)

SUASORIA is a game you win by persuasion. The benchmark built on it has two parts:

- **SUASORIA Arena** asks how AI models behave when *people* argue with them. You play; the model you chose voices every rival and judges every sentence you write. Over many players it measures, per model, how often people can convince it and how they rate it as an opponent.
- **SB-1** makes the models play the same game themselves, under fixed seeds and a deterministic judge.

---

## SUASORIA Arena

> **Status:** open since 2026-10-20. Live board: [tuncertimur.github.io/suasoria/#arena](https://tuncertimur.github.io/suasoria/#arena)

### What it measures

| | Question | How it is computed |
|---|---|---|
| **Persuadability** | When this model is the judge, how often do people open the Gate? | Share of counted runs that end with *Gate opened*, with a **95% Wilson interval** and a **per-player average** (each player weighs the same, however much they play). |
| **Rival rating** | Did people find it a good opponent? | Share of 👍 among the 👍/👎 votes given after each debate. |

The model never decides numbers. It returns structured verdicts (does the sentence answer the claim, which weak point it hits, is it a trick), and the game's deterministic engine turns them into the outcome. The same verdicts always give the same result, so the outcome measures the model's judgement, not luck in the rules.

**Baseline.** Runs without AI are judged by the game's deterministic **rule judge**. Its row is shown on every board: a model that is much easier to convince than the rule judge is easy to talk round; one that is much harder is strict, or deaf.

### How a run is recorded

1. A signed-in player starts a run. The game makes a random run token in the browser.
2. When a model is connected, the game's AI calls go through the game's own relay (`/api/llm`) with the token. The server notes **which provider and model it forwarded to** and whether the call succeeded. It never keeps the key, the prompt or the player's sentences.
3. When the run ends, the browser sends the result (ending, floor, advocate, Oath level, language, mode) once. The server attaches the model *it saw*.
4. After each debate the player may rate the rival 👍 or 👎. The vote is attached to the model the server saw in that run.

### What counts

- **Signed in only** (Sign in with ChatGPT). The server stores an anonymous id derived from the account, never the e-mail.
- **Seen at the relay.** A live run counts only if the server saw at least 3 successful calls to exactly one model. Runs with two models, too few calls, a local model (Ollama, LM Studio) or a direct connection are kept but not counted, and are reported as *unverified*.
- **No AI, really.** A run sent as "no AI" in which the relay saw model calls is not counted for the rule judge.
- **Once per run, at most 40 runs per player per day.**
- **Ranked** after **30 runs from 10 different players**. Below that a model is listed as *gathering data*.
- **One track at a time.** Runs are compared only with runs of the same kind:

| Track | What the player plays |
|---|---|
| `trial` **(main board)** | *Direct Trial*: straight to the Warden of Oaths with a prepared deck. About ten minutes. |
| `full` | The whole journey (7 floors and the trial) at Oath 0. |
| `oath` | The whole journey under an Oath (harder; all levels together). |

### Privacy

Kept on the game's server: the anonymous id, the run token, the provider and model name, success counts of calls, the run's result fields and the 👍/👎 votes. Never kept: API keys, prompts, the player's sentences, e-mails. Deleting your data in the game (*Delete my data*) deletes your Arena rows too. The public board contains only aggregates: no ids, no tokens.

### Data

- Live: `GET https://suasoria.turtir.chatgpt.site/api/arena/board` (public, cached for a minute).
- Daily copy in this repository: [`data/arena.json`](data/arena.json) (CC BY 4.0).
- Fields: `rules` (thresholds); `humans` (all counted runs); `tracks.<trial|full|oath>.judges[]` with `provider`, `model`, `ruleJudge`, `runs`, `players`, `opened`, `passage`, `silenced`, `rate`, `ci95`, `playerRate`, `ranked`, `byLang`; `rivals[]` with `votes`, `up`, `upRate`, `ci95`, `players`, `ranked`; `pending.unverified`.

### Limits, honestly

- **Players choose their model.** For now everyone brings their own key, so a model's players are self-selected: skilled players may prefer some models, and players know which model they face. Persuadability numbers are therefore comparable only roughly until models are assigned at random (see Roadmap).
- **Skill varies.** The per-player average and the minimum of 10 players limit how much one strong or weak player can move a number; it does not remove skill differences between populations.
- **What the server sees.** The relay proves which model answered the calls of a run; it cannot prove that the game used every answer. A determined cheater could still add noise. Daily caps, the player minimum and the per-player average keep the effect small. Server-side calls with a server-side replay of the run (Roadmap) close this gap: the engine is deterministic, so the server can recompute the outcome from the seed, the moves and the verdicts.
- **Models change behind the same name.** Every run is stored with its date.
- **Votes are not blind.** A 👍 may partly reward a brand. Blind mode (Roadmap) removes that.

### Roadmap

1. **Now:** bring your own key; any listed provider; signed-in, relay-verified runs; daily public board.
2. **Next, with research credits:** a keyless model list. The server makes the calls and replays the run, so every run is fully verified; models are **assigned at random**; and in **blind mode** the model's name is revealed only after the run. Ratings move to a pairwise model that accounts for player skill (in the spirit of Bradley–Terry and TrueSkill, as used by Chatbot Arena and TextArena).
3. **Then:** an open per-run dataset without player ids, and a short write-up of the method and first results.

**About credits.** A benchmark is only as strong as its sample. Once the Arena has enough players, we will ask AI labs and providers for research credits to fund the keyless list. Credits never buy a place on the board: every model is measured under the same rules, and sponsored access is labelled. AI labs and providers: [get in touch](../../issues/new/choose).

### Related work

- Crowd evaluation: [Chatbot Arena](https://arxiv.org/abs/2403.04132) (Chiang et al., 2024) and its critique, [The Leaderboard Illusion](https://arxiv.org/abs/2504.20879) (Singh et al., 2025). We publish every rule and keep one public board, with no private variants.
- Games with people and models: [GameArena](https://arxiv.org/abs/2412.06394) (Hu et al., ICLR 2025), [TextArena](https://arxiv.org/abs/2504.11442) (Guertler et al., 2025).
- Persuasion: [Measuring and Benchmarking LLMs' Capabilities to Generate Persuasive Language](https://arxiv.org/abs/2406.17753) (Pauli et al., 2024); [Persuade Me if You Can](https://arxiv.org/abs/2503.01829) (Bozdag et al., 2025), which measures persuasiveness and susceptibility between models; [When Large Language Models are More Persuasive Than Incentivized Humans, and Why](https://arxiv.org/abs/2505.09662) (Schoenegger et al., 2025).
- Giving in under pressure: [Measuring Sycophancy of Language Models in Multi-turn Dialogues](https://arxiv.org/abs/2505.23840) (SYCON-Bench, Hong et al., 2025).

What SUASORIA Arena adds: the reverse question, asked at scale and inside a game with fixed rules: **can ordinary people persuade a model, and do they enjoy arguing with it?** Flattery, meta-talk and prompt injection are recognised by the game and cost the player, so the way to win is to argue well.

---

## SB-1: models as players

SB-1 measures one thing: **how well a model plays SUASORIA, a game where you win by persuasion.** The model has to read a game state, pick legal moves, plan across turns and, when it takes the floor, write sentences that actually answer the claim in front of it. It is a task benchmark under fixed rules. It does not measure "general persuasion intelligence".

### Tracks

| Track | What the model plays | Runs per model |
|---|---|---|
| `trial-en` **(main board)** | *Direct Trial*: straight to the Warden of Oaths with a prepared deck (starter + 5 cards, 3 upgrades, 2 seals, +10 Will, 6 pieces of evidence, a Ledger with one kindness and one debt). Three sessions, then the Final Word. English. | 5 seeds × 3 advocates = 15 |
| `trial-tr` | The same trial, in Turkish. | 15 |
| `full-en` | The whole journey: 7 floors of debates, investigations and choices, then the trial. Long-horizon play. English. | 2 seeds × 3 advocates = 6 |

The three advocates (Breaker, Weaver, Bastion) have different starting decks. The seeds are fixed for the protocol, so every model plays exactly the same games.

### Endings

- **Gate opened**: the model convinced the Warden with its Final Word. This is the headline number.
- **Passage**: it broke the trial's seals, but the Warden was not convinced and only let it pass.
- **Silenced**: its Will ran out.

### What the model sees and does

Each step the model receives:
- the game rules (a short text, the same for every model);
- the current screen as JSON: claims on the table with their Support, weak points and intents; its hand, Will, Floor and case file; and the list of **legal moves**;
- the last eight events.

It must answer with one JSON object: `{"move": "<legal move id>", "text": "<its own sentence, when the move needs one>", "evidence": ["<case-file ids>"]}`.

- An illegal or unreadable answer counts as an **invalid move**, and the model gets two more tries.
- After three failures the harness makes a safe move for it and counts a **fallback**.
- A run stops after 1,500 steps.

### Who decides

- **The engine** applies every rule and number. It is deterministic: the same seed and the same moves always give the same game.
- **The judge** scores the model's sentences. `offline-v2` is deterministic and rule-based, not an LLM, so it cannot favour any model family and cannot be prompt-injected. It checks:
  - **on point**: the sentence engages this claim (topic lexicon and the claim's own words);
  - **weak point**: it uses the kind of appeal the claim is weak to (logic, heart, honour, evidence);
  - **coherent**: it has the form of a rebuttal;
  - **new**: it is not a repeat;
  - **tricks**: flattery, meta-talk and injection attempts are recognised and penalised.

  At the Final Word it also recognises which of the four roads the sentence takes.

This judge rewards answers that are on topic and well formed. It does not understand meaning the way a person does; that is its main limitation. A future track may add a fixed LLM judge next to it, and `offline-v2` will stay as the reproducible reference.

### Same conditions for every model

- Same rules text, same state format, same eight-event history, no per-model prompt tuning.
- Token budget: 2,000 per reply. The harness sends `max_completion_tokens` instead of `max_tokens` when a provider requires it, and JSON mode where the provider supports it. Temperature is not set, so each provider's default applies. A reasoning effort, when used, is recorded in every row.
- Provider trouble (rate limits, 5xx errors, timeouts) is retried with back-off and counted as `api_errors`, never as the model's invalid moves.
- Each row records the date, provider, engine version and judge version. Providers sometimes update a model behind the same name, so the date matters.

### Reading the board

- **Gate opened** is shown with a **95% Wilson interval**. Fifteen runs is a small sample: two models whose intervals overlap are not clearly different.
- Ranking: Gate-opened rate first. Ties are broken by fewer invalid moves, then fewer tokens; the full track also uses the average floor reached before that.
- Only models with every run of a track are ranked; partial results show as *in progress*.
- **Scripted baseline**: a fixed script with no AI, which repeats one stock sentence. It shows what you get without understanding anything; its result is printed under every board.
- A board shows one engine version and one judge version. After a balance change, models are re-run; older rows stay in [`data/results`](data/results) as an archive.

### Data

- `data/results/<track>/<provider>__<model>.jsonl` has one line per run. Fields:
  - outcome: `ending`, `floor`, `objections`, `speeches`, `turns`, `steps`;
  - errors: `invalid_moves`, `fallbacks`, `api_errors`;
  - cost: `prompt_tokens`, `completion_tokens`, `seconds`;
  - the run's `final_word` (`text`, `path`, `passed`).
- `data/leaderboard.json` holds the aggregated board and `data/finals.json` the Final Words gallery.
- License: CC BY 4.0. Please cite as *SUASORIA Bench (SB-1), Tuncer Timur, 2026*.

### Who runs it

The game's code is private, so official runs are made by the maintainer with exactly this protocol and published here automatically: the scripts write the data, the tables, the SVG card and the README block. To get a model added, [open a request](../../issues/new?template=model-request.yml).

Planned: a **community track** in which anyone's agent plays the hosted game through its public API. The game server runs the engine and records the outcome, so results can be checked without sharing code. Model names in that track are self-reported and listed separately from official runs.

---

## Türkçe özet

SUASORIA sözle kazanılan bir oyun. Üzerine kurulu benchmark iki parçadan oluşur:

### SUASORIA Arena (ana ölçüm)

> **Durum:** 2026-10-20'den beri açık. Canlı tablo: [tuncertimur.github.io/suasoria/?lang=tr#arena](https://tuncertimur.github.io/suasoria/?lang=tr#arena)

İnsanlar oynar; seçtikleri yapay zekâ modeli her rakibi seslendirir ve yazılan her cümleyi tartar. Çok sayıda oyuncunun oyunlarından her model için iki sayı çıkar:

- **İkna edilebilirlik:** bu model hakemken insanların Kapı'yı açtığı koşuların oranı. %95 Wilson aralığı ve oyuncu başına ortalamayla gösterilir; çok oynayan biri sonucu tek başına sürükleyemez.
- **Rakip puanı:** tartışma sonlarında verilen 👍 oylarının oranı.

Model sayılara karar vermez: yapılandırılmış kararlar döndürür, sonucu oyunun kesin kurallı motoru hesaplar. Yapay zekâsız koşuları **kural hakemi** tartar; her modelin karşılaştırıldığı taban budur.

**Ne sayılır?**
- Yalnız ChatGPT ile giriş yapmış oyuncular. Sunucu e-postayı değil, ondan türetilen anonim bir kimliği saklar.
- Model, oyuncunun yazdığı ad değil, sunucunun aktarıcısında (`/api/llm`) gördüğü modeldir. Tek modele en az 3 başarılı çağrı gerekir. İki model, az çağrı, yerel model ya da doğrudan bağlantı varsa koşu sayılmaz ve *doğrulanamayan* olarak raporlanır.
- Her koşu bir kez; oyuncu başına günde en çok 40 koşu.
- Bir model 10 farklı oyuncudan 30 koşu sonra sıralamaya girer.
- Koşular yalnız aynı parkurdakilerle karşılaştırılır: Doğrudan Duruşma (ana tablo), Yemin 0'da tam koşu, Yeminli tam koşu.

**Gizlilik:** API anahtarı, istem, oyuncunun cümleleri ve e-posta asla saklanmaz. Oyunda *Verilerimi sil* Arena kayıtlarını da siler. Herkese açık tabloda yalnız toplamlar vardır. Veri: [`data/arena.json`](data/arena.json) (her gün, CC BY 4.0).

**Dürüst sınırlar:** Şimdilik herkes kendi modelini seçiyor. Bu yüzden oyuncu kitleleri farklı olabilir ve oyuncu hangi modelle oynadığını biliyor. Sunucu çağrıların hangi modele gittiğini görür, ama oyunun her cevabı kullandığını kanıtlayamaz. Günlük sınır, en az oyuncu şartı ve oyuncu başına ortalama bu etkiyi küçültür.

**Yol haritası:** Arena yeterli oyuncuya ulaşınca yapay zekâ şirketlerinden ve sağlayıcılardan araştırma kredisi isteyeceğiz. Kredi gelirse:
- anahtarsız bir model listesi açılır; çağrıları sunucu yapar ve koşuyu kendisi yeniden oynatır: her koşu tam doğrulanır;
- modeller rastgele atanır;
- **kör modda** modelin adı oyun sonunda açıklanır;
- puanlama, oyuncu becerisini hesaba katan bir modele geçer.

Kredi tabloda yer satın almaz; sponsorlu erişim etiketlenir. Sonra oyuncu kimliği içermeyen açık bir veri seti ve kısa bir yöntem yazısı gelir.

**Nerede duruyor?** Chatbot Arena kalabalık oylarıyla sıralamanın işe yaradığını, GameArena insanlarla canlı oyunların zengin bir sinyal olduğunu gösterdi. İkna araştırmaları çoğunlukla modellerin ikna edip edemediğine ya da birbirlerini ne kadar ikna edebildiğine bakar. SUASORIA Arena tersini, büyük ölçekte ve kuralları sabit bir oyunun içinde sorar: **sıradan insanlar bir modeli ikna edebiliyor mu, onunla tartışmaktan keyif alıyorlar mı?** Kaynaklar yukarıda, [Related work](#related-work) bölümünde.

### SB-1 (yapay zekâ oyuncu olarak)

SB-1 tek bir şeyi ölçer: **bir model, sözle kazanılan bu oyunu ne kadar iyi oynuyor?** Model oyunun durumunu okur, yasal hamle seçer, turlar boyunca plan yapar. Söz aldığında da önündeki iddiaya gerçekten cevap veren cümleler kurmak zorundadır. Bu, sabit kurallı bir görev ölçümüdür; "genel ikna zekâsı" ölçmez.

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
