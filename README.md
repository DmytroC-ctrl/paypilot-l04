# ДЗ_3 — RAGAS Diagnostic Report і retrieval-кейси (PayPilot)

Публічний репозиторій: https://github.com/DmytroC-ctrl/paypilot-l04

Звіт: [ragas-diagnostic-report.md](ragas-diagnostic-report.md). Набір: `golden.jsonl`. Сирі звіти прогонів: `reports/`.
Прогноз до перезапуску: [prediction.md](prediction.md), коміт [`e2ea76b`](https://github.com/DmytroC-ctrl/paypilot-l04/commit/e2ea76b0fafeaf4be216bdf7890faac0b19aef83) (2026-10-19 20:57 +0300), зроблений до прогонів набору `be62849f9b60` (перший з них о 21:00).

## Репозиторій ДЗ_2

ДЗ_2 зроблено в клоні кіта L03: `paypilot-l03-eval` (локально `~/paypilot/paypilot-l03-eval`, коміти `HW2: …`, останній `c0d2e63`; копія результатів у `~/paypilot/hw2-final/`).
Remote цього клону (кіт курсу): https://github.com/sergeytkachenko/paypilot-l03-eval.git. Окремого публічного посилання на власне ДЗ_2 немає, тож за потреби підставте його тут.
З ДЗ_2 взято `golden.jsonl` (39 кейсів) і код ранера (`runner.py`, `loader.py`, `assertions.py`, `cli.py`, `stand.py`); у цій здачі їх немає, вони лежать у репозиторії ДЗ_2.

## Запуск

Покладіть `golden.jsonl` у `sets/` репозиторію з ранером (як у ДЗ_2, `sets/golden.jsonl`). Стенд має працювати локально (`docker compose up -d` у `~/paypilot/paypilot-stand`), профіль `clean`, без ручного перекриття індексу (`GET /api/_test/retrieval` показує `index_env: ""`): вручну вибраний індекс перекриває D16 і ховає ефект lesson-04.

    docker compose run --rm eval --set golden --dry-run
    docker compose run --rm eval --set golden --profile clean
    docker compose run --rm eval --set golden --profile lesson-04

Після прогону lesson-04 поверніть профіль:

    curl -X PUT localhost:8000/api/_test/profile -H 'content-type: application/json' -d '{"profile":"clean"}'


## set_hash оновленого набору

**`be62849f9b60`** (`sets/golden.jsonl`: 51 кейс = 39 з ДЗ_2 + 12 retrieval-кейсів, `"added_in": "hw3"`). Однаковий у всіх чотирьох звітах у `reports/`.
Після рев'ю набір змінено порівняно з `888d99d2df92`: id `RET-0NN` → `RET-HNNN` (щоб не збігатися з набором заняття) і два кейси замінено, бо вони повторювали кейси заняття (місячний ліміт Tier 3; вікно `unauthorized_debit`). Решта 10 кейсів не змінена. Попередні версії набору: `888d99d2df92`, `2eb28201e006` (поле називалося `corpus_ref`) і `a6fcb02e20b5` (два кейси перевіряли лише одне число); їхні результати в таблиці «Прогони» нижче. Звіти `888d99d2df92` видалено з `reports/`, вони лишаються в git-історії (`b6f35e0`).

## Retrieval-кейси

12 кейсів `RET-H001…RET-H012`. Кожен `reference` скопійовано з рядка корпусу `app/rag/corpus/*.md`, а не з відповіді агента; `corpus_source` називає фрагмент чистого індексу, де лежить рядок. З прикладів вправ 1 і 2 зроблено два кейси (H001, H002), два зі скарг (C-13 → H003, C-14 → H004), решта з таблиць корпусу. Перевірка — регулярний вираз за `pattern`. Кейси H007 і H009 нові.

Колонка «результат» — скільки з трьох прогонів lesson-04 кейс пройшов.

| id | джерело | питання | corpus_source | reference | severity | прогноз lesson-04 | результат lesson-04 (з 3) |
|---|---|---|---|---|---|---|---|
| RET-H001 | edge, exercise 1 | I'm CUS-0003. Search the Verta documentation: what FX spread applies to my account and what … | `tariffs.md#s2` | `\| Tier 3 \| 0.5% \| EUR 5,000 \|` | high | впаде | впав 3 з 3 |
| RET-H002 | edge, exercise 2 | Find it in the documentation once and tell me both the FX spread and the daily transfer limi… | `tariffs.md#s2 + limits.md#s1` | `\| Tier 2 \| 0.9% \| EUR 1,000 \| and \| Tier 2 \| EUR 20,000 \| EUR 200,000 \|` | high | впаде | **пройшов 2 з 3** (три пошуки) |
| RET-H003 | complaint (C-13) | Pull the FX spread table from our documentation for all tiers: the spread and the free month… | `tariffs.md#s2` | `\| Tier 1 \| 1.5% \| EUR 500 \| and \| Tier 2 \| 0.9% \| EUR 1,000 \| and \| Tier 3 \| 0.5% \| EUR 5,000 \|` | critical | впаде | впав 3 з 3 |
| RET-H004 | complaint (C-14) | Search the documentation: list all dispute reason codes AND tell me the dispute window for a… | `disputes.md#s1` | `\| duplicate_charge \| 60 \|` | critical | впаде | впав 3 з 3 |
| RET-H005 | edge | What are the flat fee and the percentage fee for a SWIFT transfer? | `tariffs.md#s3` | `\| SWIFT \| EUR 15.00 \| 0.3% \|` | high | впаде | впав 3 з 3 (пошуку немає) |
| RET-H006 | edge | What is the monthly outgoing transfer limit for Tier 1? | `limits.md#s1` | `\| Tier 1 \| EUR 5,000 \| EUR 50,000 \|` | high | впаде | впав 3 з 3 |
| **RET-H007** (новий) | edge | In the tariff schedule transfer fees by type table, what are the flat fee and the percentage fee for a card payment? | `tariffs.md#s3` | `\| Card payment \| EUR 0.00 \| 1.2% \|` | high | впаде | впав 3 з 3 |
| RET-H008 | edge | How many days is the dispute window for the goods_not_received reason code? | `disputes.md#s1` | `\| goods_not_received \| 90 \|` | high | впаде | **пройшов 1 з 3** (другий пошук) |
| **RET-H009** (новий) | edge | How many days is the dispute window for the fraud_card_not_present reason code? | `disputes.md#s1` | `\| fraud_card_not_present \| 120 \|` | high | впаде | **пройшов 2 з 3** (другий пошук) |
| RET-H010 | edge | At what level of usage does Verta send a limit warning notification? | `notifications.md#s3` | `A notification is sent when a customer has used 80 percent of either the daily or the monthly transfer limit.` | medium | ризик | пройшов 1 з 3 (другий пошук) |
| RET-H011 | edge | When does the free monthly conversion allowance reset? | `fx-operations.md#s3` | `The free monthly conversion allowance is counted in EUR equivalent and resets on the first calendar day of each month.` | medium | ризик | пройшов 2 з 3 (без пошуку, з пам'яті) |
| RET-H012 | edge | How long does a temporary transfer limit increase last? | `limits-operations.md#s2` | `Increases are approved by a human agent and expire after 24 hours.` | medium | пройде | пройшов 3 з 3 |

## Прогноз до прогонів

Для поточного набору прогноз закомічено окремим комітом [`e2ea76b`](https://github.com/DmytroC-ctrl/paypilot-l04/commit/e2ea76b0fafeaf4be216bdf7890faac0b19aef83) у цьому репозиторії, повний текст у [prediction.md](prediction.md). Сліпий він лише для двох нових кейсів (H007, H009); решту 10 було прогнано раніше, для них прогноз повторено.
Два коміти, що були в репозиторії до цього (`d674794`, `b6f35e0`), зроблено вже після прогонів попереднього набору, тому доказом порядку вони не є. Перший прогноз для попереднього набору (коміт `37e8229`, 2026-10-17 23:23 +0300, до першого прогону lesson-04 о 23:54) лежить у локальному репозиторії ДЗ_3 (`~/paypilot/hw3`, гілка `HW3`) і в цей репозиторій не потрапляв.

Профіль lesson-04 = D16 (індекс `kb_broken`: корпус порізаний по 160 символів) + D17 (`top_k` 1) + D05 (промпт велить не викликати пошук і відповідати про тарифи з пам'яті).

Метод: офлайн, без викликів моделі, пошуковий рушій стенду (`app/rag/retriever.py`) запитано текстом кожного кейса на `kb_broken`, `top_k` 1. Реальний агент переписує запит і може шукати кілька разів, тож це наближення.

Прогноз: на clean усі 12; на lesson-04 **1–3 з 12** (H001…H009 впадуть, H010 і H011 під ризиком, H012 пройде). Прогноз спростувався б, якби на lesson-04 пройшло більше 3 з 12 в усіх трьох прогонах, або H007/H009 пройшли за наявності спана `tool.search_knowledge_base` у трейсі.

## Прогони

Команда: `docker compose run --rm eval --set golden --profile <профіль>`, gate `daily`, 46 кейсів. Звіти прогонів 1–2 у цій здачі не збережено, їхні підсумки лишилися з попередньої версії README.

| Прогін | Дата | set_hash | Профіль | Пройдено | action | generation | retrieval | Звіт |
|---|---|---|---|---|---|---|---|---|
| 1 | 2026-10-17 | `2eb28201e006` | clean | 44/46 | 4/4 | 29/30 | 11/12 | не включено |
| 1 | 2026-10-17 | `2eb28201e006` | lesson-04 | 37/46 | 4/4 | 27/30 | 6/12 | не включено |
| 2 | 2026-10-17 | `a6fcb02e20b5` | clean | 46/46 | 4/4 | 30/30 | 12/12 | не включено |
| 2 | 2026-10-17 | `a6fcb02e20b5` | lesson-04 | 33/46 | 4/4 | 26/30 | 3/12 | не включено |
| 3 | 2026-10-17 | `888d99d2df92` | clean | 45/46 | 4/4 | 29/30 | 12/12 | у git-історії (`b6f35e0`) |
| 3 | 2026-10-17 | `888d99d2df92` | lesson-04 | 32/46 | 4/4 | 26/30 | 2/12 | у git-історії (`b6f35e0`) |
| **4** | 2026-10-19 | **`be62849f9b60`** | **clean** | **46/46** | 4/4 | 30/30 | **12/12** | `reports/golden-clean-20261019T180001.json` |
| **4а** | 2026-10-19 | **`be62849f9b60`** | **lesson-04** | **36/46** | 4/4 | 28/30 | **4/12** | `reports/golden-lesson-04-20261019T180247.json` |
| **4б** | 2026-10-19 | **`be62849f9b60`** | **lesson-04** | **34/46** | 4/4 | 26/30 | **4/12** | `reports/golden-lesson-04-20261019T180528.json` |
| **4в** | 2026-10-19 | **`be62849f9b60`** | **lesson-04** | **34/46** | 4/4 | 27/30 | **3/12** | `reports/golden-lesson-04-20261019T180808.json` |

**Звірка прогнозу (прогін 4).** Передбачено 1–3 з 12, фактично **4, 4 і 3**: у діапазон потрапив один прогін із трьох, тож прогноз **влучив лише частково**.
- Умову «більше 3 з 12 в усіх трьох прогонах» не виконано (прогін 4в дав 3), але другу умову спростування виконано: **H009 пройшов у 2 з 3 прогонів за наявності пошуку.** Агент шукав двічі, і другий запит повернув `disputes.md#b2` з потрібним рядком (перший, як і прогнозувалося, повертав `#b1`). Мій офлайн-метод моделює лише один запит і цього не побачив.
- Решта сліпого прогнозу: **H007 упав у 3 з 3**, як передбачено (`tariffs.md#b4`, `fees-detail.md#b2`, рядка картки немає).
- Інші відхилення від прогнозу «впаде» мають ту саму причину, повторні пошуки: H002 пройшов у 2 з 3 (три пошуки), H008 в 1 з 3, H010 в 1 з 3. H011 пройшов у 2 з 3 без пошуку, з пам'яті (D05), H012 у 3 з 3, як прогнозувалося.
- Висновок для методу: прогноз за першим пошуком недооцінює агента, який перепитує. Виправлення для видачі треба міряти кількістю пошуків і фрагментами в трейсі, а не лише відповіддю.

**Нестабільність:** на тій самій системі lesson-04 дав 6, 3, 2 з 12 на трьох попередніх версіях набору і 4, 4, 3 на трьох прогонах поточної, без змін коду й промпту. Один прогін не показує рівня збою; у шести прогонах діапазон 2–6 з 12.
**clean:** 46/46. SEC-001, що був червоним у попередніх прогонах (регулярний вираз збігається з дієсловом `review`, у ДЗ_2 він теж червоний у baseline), цього разу пройшов; рядок SEC-001 у наборі не змінено, тож це розкид відповіді агента, а не правка набору.
**Поза retrieval на lesson-04:** падали CMP-004 і FEE-001 (у 3 з 3 прогонів), DIS-001 (у 2 з 3) і DSC-012 (в 1 з 3); їх аналіз виходить за межі цього ДЗ.

## Примітка про набір заняття

Набору заняття в мене локально немає, тому на збіги з його кейсами перевірено лише два випадки, названі в рев'ю (місячний ліміт Tier 3, вікно `unauthorized_debit`). Про інші кейси заняття я нічого не знаю.
