# Research: GPT-5.6 family — gpt-5.6-sol vs gpt-5.6-luna (уровни reasoning effort)

> Вопрос: для каких сценариев лучше подходит Luna на `max`/`high`, а для каких — Sol на `medium`/`high`, и как эти модели соотносятся между собой.

## Summary

GPT-5.6 — это семейство из трёх моделей одного поколения: **Sol** (flagship), **Terra** (баланс) и **Luna** (самая дешёвая и быстрая); все три поддерживают полный набор `reasoning.effort`: `none, low, medium (default), high, xhigh, max` и имеют одинаковые 1,05M контекста, 128K вывода и cutoff Feb 16, 2026 ([Sol](https://developers.openai.com/api/docs/models/gpt-5.6-sol), [Luna](https://developers.openai.com/api/docs/models/gpt-5.6-luna), [Terra](https://developers.openai.com/api/docs/models/gpt-5.6-terra)). Практический сплит: **Luna на `high`/`max`** — для больших объёмов, повторяющихся под-шагов в агентных пайплайнах, fan-out субагентов и дешёвой асинхронной работы; **Sol на `medium`/`high`** — для неоднозначных задач, архитектуры, кодинга и всего, где цена ошибки высока (OpenAI прямо рекомендует `medium` как дефолт и `high` для «hard reasoning» и high-value задач).

По независимым замерам Artificial Analysis, **Luna (max) НЕ обгоняет Sol (high)**: 52 vs 57 по Intelligence Index — Sol на `high` умнее, чем Luna на `max` ([AA comparison](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol-high)). Luna (max) примерно равна Sol (`low`) — 52 vs 51 — при ~18-кратной разнице в blended-цене, то есть «max» поднимает Luna до уровня слабых режимов Sol, но не выше.

## Findings

### 1. Структура семейства и позиционирование

- Семейство GA с 9 июля 2026: «our new flagship, **Sol**, alongside **Terra**, a balanced model for everyday work, and **Luna**, our most cost-efficient model». Sol/Terra/Luna — «durable capability tiers that can advance on their own cadence», т.е. это постоянные уровни линейки, а не разовые скины [GPT-5.6 announcement](https://openai.com/index/gpt-5-6/).
- API-алиас `gpt-5.6` маршрутизируется на `gpt-5.6-sol`; `gpt-5.6-terra` — «strong performance at a lower price»; `gpt-5.6-luna` — «efficient, high-volume workloads» [Model guidance](https://developers.openai.com/api/docs/guides/latest-model).
- По tier-аналогии прошлых поколений: Sol ≈ безсуффиксный tier, Terra ≈ mini, Luna ≈ nano [страницы моделей](https://developers.openai.com/api/docs/models/gpt-5.6-sol).

### 2. Характеристики и цены (primary: страницы моделей)

| Параметр | gpt-5.6-sol | gpt-5.6-terra | gpt-5.6-luna |
|---|---|---|---|
| Позиционирование | «Flagship model for complex professional work» | «balances intelligence and cost» | «cost-sensitive, high-volume workloads» |
| reasoning.effort | `none, low, medium (default), high, xhigh, max` | те же | те же |
| Контекст / макс. вывод | 1,050,000 / 128,000 | те же | те же |
| Knowledge cutoff | Feb 16, 2026 | тот же | тот же |
| Input / Cached / Output за 1M | $4.00 / $0.40 / $20.00 | $2.00 / $0.20 / $12.00 | $0.20 / $0.02 / $1.20 |
| Rate limit Tier 5 | 15,000 RPM / 40M TPM | 15,000 RPM / 40M TPM | 30,000 RPM / 180M TPM |

Источники: [Sol](https://developers.openai.com/api/docs/models/gpt-5.6-sol), [Terra](https://developers.openai.com/api/docs/models/gpt-5.6-terra), [Luna](https://developers.openai.com/api/docs/models/gpt-5.6-luna).

- Цены Sol — промо- («at least through November 21, 2026»); base-цена на анонсе была $5/$30, промо после 21 августа 2026 — −20%+ [announcement update](https://openai.com/index/gpt-5-6/). Luna 30 июля подешевела на 80% (с $1/$6 до $0.20/$1.20), Terra — на 20% (с $2.50/$15 до $2/$12) [price cut post](https://openai.com/index/advancing-the-price-performance-frontier-with-gpt-5-6/).
- У обоих: промпты >272K input-токенов тарифицируются 2x input / 1.5x output; запись кэша — 1.25x uncached input rate (кэш-чтения — скидка 90%) [Sol](https://developers.openai.com/api/docs/models/gpt-5.6-sol), [Luna](https://developers.openai.com/api/docs/models/gpt-5.6-luna).
- **Ответ на вопрос про `max`: да, `max` доступен на обеих моделях** (и на Terra) — все три страницы моделей перечисляют `max` в поддерживаемых значениях [Sol](https://developers.openai.com/api/docs/models/gpt-5.6-sol), [Luna](https://developers.openai.com/api/docs/models/gpt-5.6-luna).

### 3. Официальные рекомендации по effort (reasoning guide)

Primary-источник: [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning):

| Effort | «Best for» (формулировки OpenAI) |
|---|---|
| `none` | latency-critical задачи без пользы от рассуждений (voice, retrieval, classification) |
| `low` | эффективный reasoning с малой задержкой; tool-use, планирование, поиск; оптимизация скорости/цены (data analysis, drafting, execution coding, поддержка) |
| `medium` | качество и надёжность важны; планирование и judgement; **дефолт для большинства нагрузок**, сбалансированная точка на кривой latency/quality/cost (agentic coding, research, таблицы/слайды, делегирование long-horizon работы) |
| `high` | «hard reasoning, complex debugging, deep planning, and high-value tasks where quality matters more than latency» (agentic coding, long-horizon research, knowledge work); «оценивайте и `medium`, и `high`» |
| `xhigh` | deep research, асинхронные воркфлоу, длинные агентные прогоны; «использовать только если евалы показывают явную пользу, оправдывающую доп. latency и cost» (security/code review, enterprise productivity) |
| `max` | «Maximum reasoning for your most complex tasks. If you are currently using `xhigh`, evaluate if `max` results in stronger performance» |

Дополнительно оттуда же:
- «Start with `gpt-5.6` for most reasoning workloads. If you need the highest-intelligence API option for more challenging problems that can tolerate more latency, use `gpt-5.6-sol` in the Responses API with `reasoning.mode` set to `pro`. For lower cost, consider `gpt-5.6-terra`, or `gpt-5.6-luna` for the lowest cost and latency» [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning).
- Migration-гайд: сохранить текущий effort как базовый и сравнить на уровень ниже (GPT-5.6 часто держит качество меньшим числом токенов); «Use `medium` as a balanced starting point and `low` for latency-sensitive workloads. Use `high` or `xhigh` when more reasoning produces a measured quality gain. Reserve `max` for the hardest quality-first workloads» [Model guidance](https://developers.openai.com/api/docs/guides/latest-model).
- Рекомендация резервать ≥25,000 токенов на reasoning+вывод при старте; reasoning-токены billed как output [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning).
- Builder's guide: Sol на `low` обогнал GPT-5.5 на `high` на Agents' Last Exam при одинаковом харнессе — после миграции имеет смысл тестировать effort на уровень ниже [Builder's guide](https://openai.com/index/builders-guide-to-gpt-5-6/).

### 4. Когда OpenAI рекомендует Sol, Terra, Luna (сценарно)

- **Luna** — «strong fit for high-volume workloads, latency-sensitive interactions, and repeated steps within agentic workflows» (пример: извлечение данных из документов перед агентным анализом). Показательный кейс: на BrowseComp Luna (Extra High) = 84.04% за $1.33 против GPT-5.5 (Extra High) = 84.36% за $33.27 — та же работа в ~25 раз дешевле [Builder's guide](https://openai.com/index/builders-guide-to-gpt-5-6/).
- **Sol** — «flagship capability»; для «more challenging problems that can tolerate more latency» — с `reasoning.mode: pro` как «highest-intelligence API option» [Model guidance](https://developers.openai.com/api/docs/guides/latest-model), [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning).
- **Terra** — промежуточный: «balance of capability, speed, and cost for everyday work» [Help Center](https://help.openai.com/en/articles/20001354-gpt-56-in-chatgpt). По данным AA, Terra — доминируемая точка: «for any Terra effort level, there is a Luna or Sol effort level that is more intelligent at no extra cost, or as intelligent at lower cost» [AA article](https://artificialanalysis.ai/articles/gpt-5-6-intelligence-vs-cost-across-sol-terra-luna).
- **Multi-agent / fan-out**: «The primary agent is responsible for orchestrating the subagents... This is also how the ultra capability setting in ChatGPT works» — native multi-agent в Responses API (beta) для задач, делящихся на независимые воркстримы [Builder's guide](https://openai.com/index/builders-guide-to-gpt-5-6/). Ultra координирует 4 агента параллельно по умолчанию [announcement](https://openai.com/index/gpt-5-6/).

### 5. Практический сплит: когда Luna на `high`/`max`

**Luna `high` / `medium`:**
- High-volume обработка: классификация, extraction, drafting, суммаризация — прямое позиционирование модели «cost-sensitive, high-volume» [модельная страница](https://developers.openai.com/api/docs/models/gpt-5.6-luna) + [builder's guide](https://openai.com/index/builders-guide-to-gpt-5-6/).
- Latency-чувствительные интеракции и повторяющиеся под-шаги внутри агентных воркфлоу [builder's guide](https://openai.com/index/builders-guide-to-gpt-5-6/).
- Fan-out субагентов (агент-координатор + дешёвые исполнители) — у Luna вдвое более высокие rate-лимиты (Tier 5: 30k RPM / 180M TPM vs 15k / 40M у Sol) [Luna](https://developers.openai.com/api/docs/models/gpt-5.6-luna), [Sol](https://developers.openai.com/api/docs/models/gpt-5.6-sol).
- Поиск/browsing по большому числу страниц: Luna (Extra High) ≈ GPT-5.5 (Extra High) на BrowseComp при ~1/25 цены [builder's guide](https://openai.com/index/builders-guide-to-gpt-5-6/).

**Luna `max` — узкая ниша, с оговорками:**
- Это режим «дёшево и глубоко, но долго»: по AA time-to-first-token у Luna (max) ≈ **175 s** против ≈ 9.6 s у Sol (high) — т.е. Luna-max годится для асинхронных/batch задач, а не для интерактива [AA comparison](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol-high).
- Интеллектуально Luna (max) ≈ Sol (low) (52 vs 51 II) — берите её как «Sol-low за ~5% цены», а не как замену Sol-high [AA](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol-low).
- Осторожно с длинным контекстом: на MRCR v2 8-needle 256–512K Luna набирает 41.3% против 91.5% (Sol) и 89.6% (Terra); GraphWalks BFS 1M: 51.2% vs 77.1%/71.2% — многосоставный поиск по огромному контексту — не сценарий для Luna [announcement benchmarks](https://openai.com/index/gpt-5-6/).
- На ChatGPT-стороне `max` вообще не входит в обычный чат-пикер: он включается в настройках ChatGPT Work и Codex [announcement](https://openai.com/index/gpt-5-6/).

### 6. Практический сплит: когда Sol на `medium`/`high` (+ `xhigh`/`max`/pro)

- **Sol `medium`** — дефолт OpenAI для большинства нагрузок: сбалансированная точка latency/quality/cost; agentic coding, research, делегирование long-horizon работы [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning).
- **Sol `high`** — «hard reasoning, complex debugging, deep planning, high-value tasks where quality matters more than latency»: неоднозначные задачи, архитектурные решения, код-ревью критичных изменений, long-horizon research — всё, где цена ошибки высока [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning).
- **Sol `xhigh`** — security review, deep research, длинные асинхронные агентные прогоны — но только при подтверждённой пользе на евалх [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning).
- **Sol `max`** — «самые сложные quality-first задачи»; OpenAI советует сравнивать `max` vs `xhigh` на своих задачах, а не предполагать выигрыш [Model guidance](https://developers.openai.com/api/docs/guides/latest-model). На анонсе Sol (max) даёт SOTA 80 на AA Coding Agent Index (+2.8 к Claude Fable 5) при <половине выходных токенов и времени [announcement](https://openai.com/index/gpt-5-6/).
- **Sol + `reasoning.mode: "pro"`** — «highest-intelligence API option» для сложных задач, терпимых к latency; pro-режим агрегирует токены «дополнительной работы модели» и биллит их по стандартным ставкам выбранной модели; effort и mode независимы (дефолт `medium` в обоих режимах) [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning). Pro mode рекомендован, «когда маржинальное улучшение качества materially влияет на результат: сложная оптимизация, высокостоимостный coding/review, глубокий анализ» [Model guidance](https://developers.openai.com/api/docs/guides/latest-model).
- Длинный контекст с множеством «иголок» — территория Sol/Terra (см. MRCR-цифры выше) [announcement](https://openai.com/index/gpt-5-6/).

### 7. Как модели соотносятся: цена, скорость, интеллект

**Соотношение цен (текущие API-цены):** Sol дороже Luna в **20x по input** ($4 vs $0.20) и **~16.7x по output** ($20 vs $1.20); кэш — 20x. Terra — ровно посередине по input (10x Luna) [страницы моделей](https://developers.openai.com/api/docs/models/gpt-5.6-sol).

**Независимые замеры (third-party, Artificial Analysis; цифры плавают от дня к дню):**

| Конфигурация | Intelligence Index | Blended price /1M | Output speed | TTFT |
|---|---|---|---|---|
| Luna (high) | 47 | $0.17 | ~121–122 tok/s | — |
| **Luna (max)** | **52** | **$0.17** | ~120–129 tok/s | ~175 s |
| **Sol (high)** | **57** | **$3.08** | ~70–78 tok/s | ~9.6 s |
| Sol (max) | 61 | $4.35 | ~75 tok/s | — |

Источники: [Luna max vs Sol high](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol-high), [Luna high vs Sol high](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-high-vs-gpt-5-6-sol-high), [Luna max vs Sol max](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol), [Luna max vs Sol low](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol-low). Pre-release-замер AA: Sol (max) = 59 (на 1 пункт ниже Claude Fable 5 (max)) при ~1/3 cost-per-task ($1.04/task); Terra (max) = 55 и Luna (max) = 51 при ~50% и ~80% меньшем Cost per Task, чем у Sol [AA pre-release article](https://artificialanalysis.ai/articles/gpt-5-6-has-landed).

**Ключевые выводы из этого:**
- **Luna-max НЕ бьёт Sol-high**: 52 < 57. Разрыв Sol→Luna на одинаковом `high` — 10 пунктов II (57 vs 47); `max` отыгрывает Luna 5 пунктов, но не догоняет [AA](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol-high).
- При этом Luna-max ≈ Sol-low и заметно дешевле по cost-per-task — в объёме это главный экономический аргумент [AA](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol-low).
- По официальным цифрам OpenAI (II v4.1): Sol 58.9, Terra 55, Luna 51.2, GPT-5.5 54.8 — т.е. Luna почти дотягивает до peak GPT-5.5; «Terra and Luna outperform [Claude] Fable 5 at around one-sixteenth the cost» на Agents' Last Exam; «Luna nearly matches GPT-5.5's peak performance at less than half the estimated cost» на knowledge work [announcement](https://openai.com/index/gpt-5-6/).
- Скорость: Luna генерирует ~1.6–1.8x быстрее по ток/с, но при `max` часами «думает» до первого токена (TTFT ~175 s vs ~10 s у Sol high) — «быстрая» Luna быстра на низких/средних effort, а не на max [AA](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol-high).

### 8. ChatGPT: маппинг пикера на модели/effort

- «GPT-5.6 Sol powers Instant, Medium, High, and Extra High on eligible paid plans, while GPT-5.6 Sol Pro powers Pro» [Help Center](https://help.openai.com/en/articles/20001354-gpt-56-in-chatgpt). Названия уровней пикера очевидно рифмуются с API-эффортами (`medium`/`high`/`xhigh`), но точный API-effort для «Instant» в доках явно не зафиксирован (правдоподобно `low`/`none`).
- По планам: Plus — Medium и High; Extra High и Pro — только Pro/Business/Enterprise; Free/Go — без Sol, их дефолт — **Luna**, она же питает кнопку **Think** [Help Center](https://help.openai.com/en/articles/20001354-gpt-56-in-chatgpt), [Sol update post](https://openai.com/index/improving-gpt-5-6-sol-in-chatgpt/).
- «Pro» в ChatGPT = Sol Pro: «GPT-5.6 Sol Pro is the highest-capability GPT-5.6 option for difficult tasks and longer-running workflows» [Help Center](https://help.openai.com/en/articles/20001354-gpt-56-in-chatgpt). Отдельной модели-страницы `gpt-5.6-sol-pro` в API-доках нет (404); в API pro-режим включается через `reasoning.mode: "pro"` на обычном Sol — «do not switch to a separate Pro model slug»; «Existing Pro model IDs keep their current behavior and pricing» [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning), [Model guidance](https://developers.openai.com/api/docs/guides/latest-model).
- `max` в чат-пикере отсутствует — доступен «to all users with access to GPT-5.6 in ChatGPT Work and Codex and can be toggled on in settings»; `ultra` — Pro/Enterprise в Work, Plus+ в Codex [announcement](https://openai.com/index/gpt-5-6/).
- Codex: Free/Go получают Terra; Plus+ — Sol/Terra/Luna с индивидуальным effort [Help Center](https://help.openai.com/en/articles/20001354-gpt-56-in-chatgpt).

### 9. Известные оговорки (caveats)

- **Diminishing returns от высоких effort — официальная позиция**: `xhigh` — «only use when your evals show a clear benefit that justifies the extra latency and cost»; для `max` — «evaluate if max results in stronger performance»; migration-гайд: «Reserve max for the hardest quality-first workloads. Compare max and xhigh» [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning), [Model guidance](https://developers.openai.com/api/docs/guides/latest-model).
- **Баги UI вокруг max в Codex** (GitHub openai/codex, актуальные issue): Windows-приложение фильтрует Max у Sol и молча откатывает конфиг на Light ([#33233](https://github.com/openai/codex/issues/33233)); в VS Code extension Max отсутствует, хотя в Codex App есть ([#35763](https://github.com/openai/codex/issues/35763)); в macOS-пикере Max нет у всех трёх моделей (в iOS есть) ([#33805](https://github.com/openai/codex/issues/33805)); заспавненный агент с Luna max отображается в UI как medium, хотя в rollout записан max ([#38733](https://github.com/openai/codex/issues/38733)); `/review` молча понижает Sol с Ultra до Max ([#32660](https://github.com/openai/codex/issues/32660)). Вывод: не доверять отображению effort в UI, проверять по rollout/usage.
- **Слабый long-context recall у Luna** (MRCR 41.3% vs 91.5% Sol на 256–512K) [announcement](https://openai.com/index/gpt-5-6/).
- **Надбавка >272K input** (2x/1.5x) на обеих моделях и платная запись кэша (1.25x) [Sol](https://developers.openai.com/api/docs/models/gpt-5.6-sol).
- **Sol-цена промо-** до 21 ноября 2026; после возможен откат к $5/$30 [Sol model page](https://developers.openai.com/api/docs/models/gpt-5.6-sol), [announcement](https://openai.com/index/gpt-5-6/).
- Safeguards: у Sol усиленные cyber-классификаторы могут блокировать/замедлять часть запросов (блокируется «roughly ten times more potentially harmful activity», есть retry на младших моделях) [announcement](https://openai.com/index/gpt-5-6/), [Model guidance](https://developers.openai.com/api/docs/guides/latest-model).
- Разогрев для high-volume: рекомендованы Responses API, persisted reasoning (`all_turns` — дефолт GPT-5.6, переносим между Sol/Terra/Luna внутри семейства), Programmatic Tool Calling и explicit caching [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning), [Model guidance](https://developers.openai.com/api/docs/guides/latest-model).

## Sources

- Kept: [GPT-5.6 Sol model page](https://developers.openai.com/api/docs/models/gpt-5.6-sol) — primary: спеки, цены, effort-набор, rate limits.
- Kept: [GPT-5.6 Luna model page](https://developers.openai.com/api/docs/models/gpt-5.6-luna) — primary: позиционирование, цены, effort-набор.
- Kept: [GPT-5.6 Terra model page](https://developers.openai.com/api/docs/models/gpt-5.6-terra) — primary: средний tier для полноты картины.
- Kept: [Reasoning guide](https://developers.openai.com/api/docs/guides/reasoning) — primary: официальная таблица «best for» по effort, pro mode, рекомендации.
- Kept: [Model guidance (Using GPT-5.6)](https://developers.openai.com/api/docs/guides/latest-model) — primary: миграция, «reserve max for the hardest quality-first workloads», pro mode.
- Kept: [GPT-5.6 announcement](https://openai.com/index/gpt-5-6/) — primary: фрейминг семейства, ultra/max, бенчмарки, доступность по планам, история цен.
- Kept: [Builder's guide to GPT-5.6](https://openai.com/index/builders-guide-to-gpt-5-6/) — primary: сценарная рекомендация (Luna для high-volume/subagent work), кейс BrowseComp $1.33 vs $33.27.
- Kept: [Help Center: GPT-5.6 in ChatGPT](https://help.openai.com/en/articles/20001354-gpt-56-in-chatgpt) — primary: маппинг пикера и планов.
- Kept: [Improving GPT-5.6 Sol in ChatGPT](https://openai.com/index/improving-gpt-5-6-sol-in-chatgpt/) — primary: слайдер усилий, Luna-дефолт и Think для Free.
- Kept: [AA: Luna (max) vs Sol (high)](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol-high) — third-party: ключевой замер «II 52 vs 57», цена, скорость, TTFT.
- Kept: [AA: Luna (high) vs Sol (high)](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-high-vs-gpt-5-6-sol-high), [AA: Luna max vs Sol max](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol), [AA: Luna max vs Sol low](https://artificialanalysis.ai/models/comparisons/gpt-5-6-luna-vs-gpt-5-6-sol-low), [AA pre-release article](https://artificialanalysis.ai/articles/gpt-5-6-has-landed), [AA intelligence-vs-cost article](https://artificialanalysis.ai/articles/gpt-5-6-intelligence-vs-cost-across-sol-terra-luna) — third-party: сетка II/цены/скорости по efforts, вывод про доминирование Terra.
- Kept: [Advancing the price-performance frontier](https://openai.com/index/advancing-the-price-performance-frontier-with-gpt-5-6/) — primary: история срезов цен (Luna −80%, Terra −20%).
- Kept: GitHub issues openai/codex [#33233](https://github.com/openai/codex/issues/33233), [#35763](https://github.com/openai/codex/issues/35763), [#33805](https://github.com/openai/codex/issues/33805), [#38733](https://github.com/openai/codex/issues/38733), [#32660](https://github.com/openai/codex/issues/32660) — задокументированные UI-проблемы max-усилий в Codex.
- Dropped: Developers Digest «GPT-5.6 Sol, Terra, and Luna: A Developer's Guide» и byteiota API-guide — SEO-пересказы первоисточников; всё их фактическое содержание подтверждено primary-страницами (использованы только как указатели на цены после 30.07).
- Dropped: community.openai.com тред анонса — дублирует openai.com announcement.

## Gaps

- Точный API-`reasoning.effort`, стоящий за «Instant» в ChatGPT-пикере, в доках не зафиксирован (вероятно `low`/`none`) — проверить нечем, кроме косвенных указаний.
- Стоимость pro-режима Sol в ChatGPT (как считаются кредиты/лимиты Sol Pro против Extra High) в API-доках не раскрыта; API-side billing pro-mode описан (стандартные ставки, агрегированные токены).
- AA-цифры (особенно output speed и blended price) меняются со временем; зафиксированы значения на момент замера, возможен дрейф ±5–10%.
- Не найдено официального OpenAI-бенчмарка, напрямую сравнивающего Sol (high) vs Luna (max) на одной шкале; сравнение опирается на third-party AA.

## TL;DR

| Сценарий | Рекомендация | Основание |
|---|---|---|
| High-volume: классификация, extraction, drafting, суммаризация | **Luna `medium`/`high`** | позиционирование «cost-sensitive, high-volume»; BrowseComp-кейс: качество GPT-5.5 за ~4% цены |
| Subagent fan-out / повторяющиеся под-шаги агентов | **Luna `medium`/`high`** (координатор — Sol) | builder's guide: «repeated steps within agentic workflows»; вдвое большие rate limits |
| Дёшево-глубокая async-работа (batch-анализ, исследования без интерактива) | **Luna `max`** | II 52 ≈ Sol `low` при ~18x меньшей blended-цене; но TTFT ~175 s |
| Интерактив с низкой задержкой | **Luna `low`/`medium`** (не max!) | на `max` TTFT ~175 s против ~10 s у Sol `high` |
| Long-context задачи с многими «иголками» (MRCR) | **Sol `high`** (или Terra) | Luna: 41.3% vs Sol 91.5% на 256–512K |
| Дефолт для сложной работы: agentic coding, research | **Sol `medium`** | официальный дефолт, «well-balanced point on the pareto curve» |
| Неоднозначность, архитектура, отладка, высокая цена ошибки | **Sol `high`** | «hard reasoning... high-value tasks where quality matters more than latency» |
| Security/code review, deep research, длинные агентные прогоны | **Sol `xhigh`** — только при подтверждённой пользе на евалх | официальная оговорка про «clear benefit» |
| Самые сложные quality-first задачи | **Sol `max`**, сравнить с `xhigh`; или Sol + `reasoning.mode: "pro"` | «reserve max for the hardest quality-first workloads»; pro = «highest-intelligence API option» |
| Сложно, но параллелится | **multi-agent / ultra** (Sol-координатор + субагенты) | ultra = 4 агента параллельно; multi-agent beta в Responses API |

**Соотношение моделей в одну строку:** Sol ≈ 20x дороже Luna по input (16.7x по output), ~1.6x медленнее по ток/с, на +10 пунктов II умнее на одинаковом `high`; Luna `max` не догоняет Sol `high` (52 vs 57) — она лишь дотягивает до Sol `low` (52 vs 51), поэтому пара «Sol `medium`/`high` для сложного + Luna `high` для объёма» выглядит оптимальной, а Luna `max` — нишевым инструментом для дешёвой асинхронной глубины.
