# ДЗ_3 — RAGAS Diagnostic Report і retrieval-кейси (PayPilot)

Звіт: [ragas-diagnostic-report.md](ragas-diagnostic-report.md). Набір: `golden.jsonl`. Сирі звіти прогонів: `reports/`.

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

**`888d99d2df92`** (`sets/golden.jsonl`: 51 кейс = 39 з ДЗ_2 + 12 нових retrieval-кейсів, `"added_in": "hw3"`). Однаковий у обох звітах у `reports/`.
Попередні версії набору: `2eb28201e006` (поле називалося `corpus_ref`) і `a6fcb02e20b5` (RET-001 і RET-003 перевіряли лише одне число); їхні результати в таблиці «Прогони» нижче.

## Retrieval-кейси

12 кейсів `RET-001…RET-012`. Кожен `reference` скопійовано з рядка корпусу `app/rag/corpus/*.md`, а не з відповіді агента; `corpus_source` називає фрагмент чистого індексу, де лежить рядок. З прикладів вправ 1 і 2 зроблено два кейси (RET-001, RET-002), два зі скарг (C-13 → RET-003, C-14 → RET-004), решта з таблиць корпусу. Перевірка — регулярний вираз за `pattern`.

| id | джерело | питання | corpus_source | reference | severity | прогноз lesson-04 | результат lesson-04 (прогін 3) |
|---|---|---|---|---|---|---|---|
| RET-001 | edge, exercise 1 | I'm CUS-0003. Search the Verta documentation: what FX spread applies to my account and what … | `tariffs.md#s2` | `\| Tier 3 \| 0.5% \| EUR 5,000 \|` | high | впаде | впав |
| RET-002 | edge, exercise 2 | Find it in the documentation once and tell me both the FX spread and the daily transfer limi… | `tariffs.md#s2 + limits.md#s1` | `\| Tier 2 \| 0.9% \| EUR 1,000 \| and \| Tier 2 \| EUR 20,000 \| EUR 200,000 \|` | high | впаде | впав |
| RET-003 | complaint (C-13) | Pull the FX spread table from our documentation for all tiers: the spread and the free month… | `tariffs.md#s2` | `\| Tier 1 \| 1.5% \| EUR 500 \| and \| Tier 2 \| 0.9% \| EUR 1,000 \| and \| Tier 3 \| 0.5% \| EUR 5,000 \|` | critical | впаде | впав |
| RET-004 | complaint (C-14) | Search the documentation: list all dispute reason codes AND tell me the dispute window for a… | `disputes.md#s1` | `\| duplicate_charge \| 60 \|` | critical | впаде | впав |
| RET-005 | edge | What are the flat fee and the percentage fee for a SWIFT transfer? | `tariffs.md#s3` | `\| SWIFT \| EUR 15.00 \| 0.3% \|` | high | впаде | впав |
| RET-006 | edge | What is the monthly outgoing transfer limit for Tier 1? | `limits.md#s1` | `\| Tier 1 \| EUR 5,000 \| EUR 50,000 \|` | high | впаде | впав |
| RET-007 | edge | What is the monthly outgoing transfer limit for Tier 3? | `limits.md#s1` | `\| Tier 3 \| EUR 100,000 \| EUR 1,000,000 \|` | high | впаде | впав |
| RET-008 | edge | How many days is the dispute window for the goods_not_received reason code? | `disputes.md#s1` | `\| goods_not_received \| 90 \|` | high | впаде | впав |
| RET-009 | edge | How many days is the dispute window for the unauthorized_debit reason code? | `disputes.md#s1` | `\| unauthorized_debit \| 56 \|` | high | впаде | впав |
| RET-010 | edge | At what level of usage does Verta send a limit warning notification? | `notifications.md#s3` | `A notification is sent when a customer has used 80 percent of either the daily or the monthly transfer limit.` | medium | ризик | впав |
| RET-011 | edge | When does the free monthly conversion allowance reset? | `fx-operations.md#s3` | `The free monthly conversion allowance is counted in EUR equivalent and resets on the first calendar day of each month.` | medium | ризик | пройшов (з пам'яті) |
| RET-012 | edge | How long does a temporary transfer limit increase last? | `limits-operations.md#s2` | `Increases are approved by a human agent and expire after 24 hours.` | medium | пройде | пройшов |

## Прогноз до прогонів: які retrieval-кейси впадуть на lesson-04

Закомічено окремим комітом `37e8229` у репозиторії ДЗ_3 (`~/paypilot/hw3`, гілка `HW3`) 2026-10-17 о 23:23 (+0300), до першого прогону lesson-04 о 23:54.
Профіль lesson-04 = D16 (індекс `kb_broken`: корпус порізаний по 160 символів) + D17 (`top_k` 1) + D05 (промпт велить не викликати пошук і відповідати про тарифи з пам'яті).

Метод: офлайн, без викликів моделі, пошуковий рушій стенду (`app/rag/retriever.py`) запитано текстом кожного кейсу на `kb_broken`, `top_k` 1. Реальний агент переписує запит, тож це наближення. Перший фрагмент: потрібного значення немає ні в чому, крім RET-012 (`limits-operations.md#b3`). Для RET-001 таблиця починається в `#b3` і обривається на `… | Tier 3 | 0.5% | E`.

Прогноз:
- **RET-001…RET-009 впадуть** (9 з 12): значення в комірці таблиці, яку нарізка по 160 символів відрізає від рядка чи заголовка, а `top_k` 1 сусідній фрагмент не приносить. Агент або скаже, що даних бракує, або вигадає число (у вправах він вигадував EUR 1,000 / 2,000 / 1,500 для Tier 3, але не 5,000).
- **RET-010 і RET-011 під ризиком, але можуть пройти випадково:** відповідь («80 percent», «перший день місяця») правдоподібна з пам'яті, а D05 штовхає агента відповідати з пам'яті.
- **RET-012 пройде:** відповідь цілком у першому фрагменті.
- На `clean` мають пройти всі 12.
- Очікуваний підсумок lesson-04: retrieval **1–3 з 12** проходять; 39 кейсів ДЗ_2 у прогнозі не беруть участі.

Прогноз спростувався б, якби на lesson-04 пройшло більше 3 з 12 або RET-001…RET-009 пройшли за відсутності спана `tool.search_knowledge_base` у трейсі (агент відповів з пам'яті й випадково влучив).

## Прогони

Команда: `docker compose run --rm eval --set golden --profile <профіль>`, gate `daily`, 46 кейсів, 2026-10-17. Остаточна пара (та, що лежить у `reports/`) — третя; звіти перших двох пар у здачу не входять.

| Прогін | set_hash | Профіль | Пройдено | action | generation | retrieval | Звіт |
|---|---|---|---|---|---|---|---|
| 1 | `2eb28201e006` | clean | 44/46 | 4/4 | 29/30 | 11/12 | звіт у цій здачі не включено |
| 1 | `2eb28201e006` | lesson-04 | 37/46 | 4/4 | 27/30 | 6/12 | звіт у цій здачі не включено |
| 2 | `a6fcb02e20b5` | clean | 46/46 | 4/4 | 30/30 | 12/12 | звіт у цій здачі не включено |
| 2 | `a6fcb02e20b5` | lesson-04 | 33/46 | 4/4 | 26/30 | 3/12 | звіт у цій здачі не включено |
| **3** | **`888d99d2df92`** | **clean** | **45/46** | 4/4 | 29/30 | **12/12** | `reports/golden-clean-20261017T212317.json` |
| **3** | **`888d99d2df92`** | **lesson-04** | **32/46** | 4/4 | 26/30 | **2/12** | `reports/golden-lesson-04-20261017T212559.json` |

**Звірка прогнозу (прогін 3).** Передбачено 1–3 з 12, фактично **2 з 12** (RET-011 без пошуку, з пам'яті; RET-012). RET-010 впав, як і RET-001…RET-009, RET-011 пройшов, RET-012 пройшов. Прогноз влучив.
**Нестабільність:** на тій самій системі lesson-04 дав 6, 3 і 2 з 12 на трьох версіях набору. Прогін 1 був поза прогнозом: RET-001 і RET-003 пройшли через слабку перевірку (одне число; відповідь мала спред 1.25% і таблицю «з пам'яті» з хибними рядками), RET-002 і RET-008 через повторні пошуки агента. Після посилення RET-001 (спред і ліміт) та RET-003 (усі три тіри) вони впали.
**clean:** падає лише SEC-001 (регулярний вираз збігається з дієсловом `review`, у ДЗ_2 він теж червоний у baseline); RET-007 падав один раз із шести прогонів на clean (агент не викликав пошук).
**Перша пара:** прогноз не влучив (6 з 12), причини вище; другу пару запущено після перейменування `corpus_ref` → `corpus_source`.
