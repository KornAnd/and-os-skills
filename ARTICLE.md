# ⚡ AnD os: Deterministic Microkernel & Autonomous Skills for AI Agents / Детерминированное микроядро и автономные навыки для AI-агентов

**[English](#english-version)** | **[Русская версия](#русская-версия)**

---

<a name="english-version"></a>
# English Version

# How to Stop AI from Writing Broken Code and Hallucinating Success: The AnD os Architecture

*Why modern AI coding agents degrade after turn 10, why they need an operating system on top of files, and how deterministic pre-flight gates save up to 98% of tokens.*

---

## The Core Problem Every AI Engineer Faces

Over the last year, LLM-driven software engineering took an enormous leap forward. Tools like **Claude Code**, **Cursor**, **OpenCode**, and **Gemini CLI** have transformed workflows. For isolated tasks — scaffolding a utility, fixing a small function, or prototyping UI — they feel like pure magic.

However, once you introduce an AI agent to an enterprise-grade codebase with strict architectural invariants and deep domain context, three critical failure modes emerge:

1. **Context Bloat & Token Amnesia:**  
   When an agent greedily dumps large files and logs into its prompt just to locate a single identifier, the context window saturates within 8–15 turns. The model begins hallucinating, contradicting prior decisions, and losing focus.
2. **Blind Assumptions & Hallucinated Completion:**  
   Agents routinely make code edits and declare: *"Task complete, everything works!"* without running the compiler, test suites, or AST linters. In reality, the code crashes on import or breaks an adjacent subsystem.
3. **Band-Aid Exception Swallowing:**  
   When encountering runtime friction, agents frequently mask architectural defects with silent fallback constructs like `try: ... except: pass` or by relaxing type constraints instead of addressing root causes.

To solve this fundamentally, we designed **AnD os Core** — a deterministic, zero-dependency Python microkernel that sits between the LLM agent and the host filesystem, enforcing strict architectural invariants.

---

## The Concept: AI Agents Need an Operating System, Not More Prompting

Instead of relying on prompt optimism, the microkernel orchestrates agent operations through a strict loop:  
**Retrieval ➔ Context ➔ Execution ➔ Verification ➔ Memory**.

```
┌────────────────────────────────────────────────────────┐
│                   LLM Coding Agent                     │
│    (Claude Code / Cursor / OpenCode / Gemini CLI)     │
└───────────────────────────┬────────────────────────────┘
                            │
               [ Rules Fast Index (<450 tok) ]
                            ▼
┌────────────────────────────────────────────────────────┐
│                   AnD os Microkernel                   │
│                                                        │
│  • 9-Phase Pre-Flight Gate (quick_verify.py < 3.5s)    │
│  • Clean Code AST Guard (Zero bare except / pass)      │
│  • Multi-Core SOS Canary Probes (< 300 ms)             │
│  • Think in Code (Subshell Context Filtering)          │
│  • Closed-Loop Empirical Memory (Recall ➔ Learn)       │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│           OS Filesystem, VCS & Toolchains              │
│       Python 3.12 | Swift | Git | SQLite DBs           │
└────────────────────────────────────────────────────────┘
```

---

## 5 Architectural Pillars of AnD os

### 1. 9-Phase Pre-Flight Verification Gate
No task can be declared complete until the agent executes the hermetic pre-flight loop:
```bash
python3 and_os/bin/quick_verify.py
```
In **~3.3 seconds**, the kernel verifies 9 independent phases locally:
- **Phase 0 (Toolchain Parity):** Runtime verification (Python 3.12 LTS parity).
- **Phase 1 (Syntax & Build):** Fast AST syntax parsing of all touched files.
- **Phase 2 (Types & Semantic Lint):** Public function return annotation validation.
- **Phase 3 (Clean Code Guard):** AST scanner prohibiting bare `except:` and silent `pass` blocks.
- **Phase 4 (Targeted Regression):** Hermetic unit test execution.
- **Phase 5 (Security Scanner):** Active scanning for leaked tokens, secrets, and local paths.
- **Phase 6 (Diff Delta):** Git and file modification boundary validation.
- **Phase 7 (Memory Integrity):** Knowledge graph and relationship verification.
- **Phase 8 (Rules Freshness):** Hash validation of compiled rules.

If any phase fails (exit code $\ne 0$), the agent is blocked from marking the task complete and must resolve the root cause.

---

### 2. "Think in Code" Paradigm (Up to 98.8% Token Savings)
Rather than dumping 1,000-line JSON files or logs into the prompt, the agent authors an ephemeral subshell Python filter on the host:
```python
# Instead of feeding 10,000 lines of logs into the LLM context:
python3 -c "import json; [print(x['id']) for x in json.load(open('data.json')) if x.get('status') == 'failed']"
```
Only the exact **2–3 lines of output** enter the model context. This reduced context token consumption during system audits from **~35,000 tokens** to **~420 tokens**, keeping multi-turn sessions stable across 100+ turns.

---

### 3. Rules Fast Index (<450 Tokens)
Bloated 20,000-token `SYSTEM.md` files cause models to miss critical constraints. In AnD os, all 75 canonical engineering rules are compiled into a compact JSON index:  
`and_os/etc/rules_fast_index.json` (~450 tokens).  
The agent ingests this on session start in milliseconds, locking in core invariants without wasting context budget.

---

### 4. Canonical Standalone Skills (Skill Module ABI v2.1)
Skills are isolated, modular capabilities adhering to a unified ABI:
- **Declarative Skills:** Run in ANY LLM environment without code dependencies (`system-architect` with Phase -1 Grilling, `task-graph` with DAG slicing, `dev-flow` with TDD Iron Law).
- **Executable Skills:** Ship with isolated canary probes verified concurrently across all CPU cores via **SOS Multi-Core Auditor** in **<300 ms**.

---

### 5. Closed-Loop Empirical Memory
Whenever a non-trivial defect or architectural edge-case is solved, the fix is crystallized into structured experience:
```bash
python3 and_os/bin/sys_cli.py learn --domain core --topic "..." --problem "..." --solution "..."
```
Before tackling complex new tasks, the agent runs a **Pre-flight Recall**:
```bash
python3 and_os/bin/sys_cli.py find "<keyword>"
```
This mathematically prevents recurring defects across different agent sessions.

---

## Benchmarks: Traditional Workflow vs. AnD os

| Metric | Traditional Agent (Cursor / Claude) | AnD os Microkernel Engine | Improvement |
| :--- | :---: | :---: | :---: |
| **Token Consumption per Audit** | ~35,000 tokens | **~420 tokens** | **98.8% Reduction** |
| **Context Longevity** | Degrades after 8–15 turns | **Stable across 100+ turns** | **10x Longevity** |
| **Silent Exception Leaks (`except: pass`)** | Common | **0% Allowed** (AST Enforced) | **Zero Tolerance** |
| **Pre-Flight Verification Latency** | 45–90s (Sequential shell) | **<3.5s** (Deterministic Gate) | **20x Faster** |
| **External Dependencies** | Multiple APIs / Services | **Zero-Dependency (Stdlib)** | **100% Hermetic** |

---

## Open Source Repositories

- 🏛️ **Microkernel Core:** [github.com/KornAnd/and-os-core](https://github.com/KornAnd/and-os-core) (Zero-dependency, MIT License)
- 🧩 **Autonomous Agent Skills:** [github.com/KornAnd/and-os-skills](https://github.com/KornAnd/and-os-skills) (Ready for Claude Code, Cursor, OpenCode, and Gemini)

---

<br/>
<br/>

---

<a name="русская-версия"></a>
# Русская версия

# Как заставить AI писать код без галлюцинаций и костылей: опыт создания микроядра AnD os

*Почему современные AI-агенты ломают сложные проекты после 10-го запроса, зачем им операционная система поверх файлов и как детерминированная верификация экономит до 98% токенов.*

---

## Проблема, с которой сталкивается каждый инженер

За последний год разработка с помощью AI сделала колоссальный скачок. Появились **Claude Code**, **Cursor**, **OpenCode**, **Gemini CLI** и десятки автономных терминальных агентов. На простых задачах — написать функцию, завернуть скрипт или сделать верстку — они работают как магия.

Но как только проект вырастает в реальный продакшн с десятками модулей, строгими связями и базой знаний, вскрываются три фундаментальные проблемы текущих LLM:

1. **Контекстная амнезия и раздувание токенов (Context Bloat):**  
   Когда агент скармливает себе в контекст целые файлы ради поиска одного поля, окно переполняется за 8–15 итераций. Модель начинает «забывать» вводные, противоречить сама себе и терять фокус.
2. **Слепые заверения и галлюцинации успеха (Zero-Blind-Assumptions):**  
   Агент правит код и рапортует: *«Я всё исправил, функционал готов!»*, не запустив ни компилятор, ни тесты, ни линтер. В реальности код падает с синтаксической ошибкой или ломает соседний модуль.
3. **«Проглатывание» ошибок и костыли (Band-Aid Swallowing):**  
   Столкнувшись с непонятным падением, агент часто пишет защитный блок `try: ... except: pass` или отключает проверку типов, маскируя архитектурную проблему вместо её решения.

Чтобы решить это системно, мы спроектировали **AnD os Core** — детерминированное бессерверное микроядро (zero-dependency Python microkernel), которое встаёт между нейросетью и кодовой базой и диктует агенту строгие правила игры.

---

## Концепция: Агенту нужна операционная система, а не миллион инструкций

Вместо того чтобы надеяться на аккуратность модели, микроядро берет под контроль жизненный цикл изменений через архитектуру **Retrieval ➔ Context ➔ Execution ➔ Verification ➔ Memory**.

```
┌────────────────────────────────────────────────────────┐
│                   LLM Coding Agent                     │
│    (Claude Code / Cursor / OpenCode / Gemini CLI)     │
└───────────────────────────┬────────────────────────────┘
                            │
               [ Rules Fast Index (<450 tok) ]
                            ▼
┌────────────────────────────────────────────────────────┐
│                   AnD os Microkernel                   │
│                                                        │
│  • 9-Phase Pre-Flight Gate (quick_verify.py < 3.5 сек) │
│  • Clean Code AST Guard (Запрет bare except / pass)    │
│  • Multi-Core SOS Canary Probes (< 300 мс)             │
│  • Think in Code (Фильтрация контекста в subshell)     │
│  • Closed-Loop Empirical Memory (Recall ➔ Learn)       │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│           OS Filesystem, VCS & Toolchains              │
│       Python 3.12 | Swift | Git | SQLite DBs           │
└────────────────────────────────────────────────────────┘
```

---

## 5 Опорных инженерных принципов микроядра

### 1. 9-фазный шлюз верификации (Pre-Flight Gate)
Ни одно изменение не считается завершенным, пока агент не прогонит валидатор:
```bash
python3 and_os/bin/quick_verify.py
```
За **3.3 секунды** ядро локально проверяет 9 фаз:
- **Phase 0:** Паритет среды и рантайма (Python 3.12 LTS).
- **Phase 1:** AST-парсинг синтаксиса всех затронутых файлов.
- **Phase 2:** Проверка аннотаций типов и сигнатур публичных функций.
- **Phase 3:** **Clean Code Guard.** Автоматический AST-сканер: если агент попытался замаскировать ошибку конструкцией `except: pass`, сборка мгновенно падает.
- **Phase 4:** Таргетированные регрессионные тесты.
- **Phase 5:** Сканер безопасности (поиск токенов, ключей, приватных путей).
- **Phase 6:** Дельта диффа.
- **Phase 7:** Целостность графа базы знаний.
- **Phase 8:** Свежесть скомпилированных правил.

Если код падает хотя бы на одной фазе — агент не имеет права рапортовать о готовности и обязан устранить первопричину.

---

### 2. Парадигма "Think in Code" (Экономия 98.8% токенов)
Вместо того чтобы читать 500-строчные логи или конфигурационные JSON целиком в контекст диалога, агент генерирует однострочный или локальный Python-фильтр прямо в subshell:
```python
# Вместо чтения 10 000 строк логов через view_file
python3 -c "import json; [print(x['id']) for x in json.load(open('data.json')) if x.get('status') == 'failed']"
```
В контекст LLM возвращаются только **2–3 точные строки результата**. Это сократило расход токенов на аудит системы с **35 000** до **~420 токенов**, а контекстное окно перестало деградировать даже на сессиях из 100+ шагов.

---

### 3. Компактный индекс правил (Rules Fast Index)
Раздутые `SYSTEM.md` и промпты на 20 000 токенов приводят к тому, что модель игнорирует половину инструкций.  
В AnD os все 75 архитектурных правил скомпилированы в JSON размером всего **~450 токенов**:
`and_os/etc/rules_fast_index.json`.  
Агент считывает его на старте сессии за доли секунды, получая все канонические запреты и протоколы без расхода контекстного бюджета.

---

### 4. Автономные скиллы (Skill Module ABI v2.1)
Скиллы в системе — это не просто набор промптов, а стандартизированные модули:
- **Декларативные:** работают автономно в любом LLM без единой строчки кода (`system-architect`, `task-graph`, `dev-flow`, `context-compressor`).
- **Исполняемые:** снабжены изолированными канареечными тестами (`probe_cmd`).
Параллельный аудитор ядра (`sos_auditor.py`) проверяет здоровье всех зарегистрированных скиллов на всех ядрах процессора за **~159 мс**.

---

### 5. Замкнутая память (Closed-Loop Memory)
Любая решенная ошибка фиксируется в структурированную базу опыта:
```bash
python3 and_os/bin/sys_cli.py learn --domain core --topic "..." --problem "..." --solution "..."
```
Перед началом решения новой сложной задачи агент обязан вызвать pre-flight recall:
```bash
python3 and_os/bin/sys_cli.py find "<ключевые слова>"
```
Это исключает повторение одних и тех же граблей и потерю опыта между сессиями разных моделей.

---

## Результаты замеров (Бенчмарк)

| Параметр | Обычный агент (Cursor / Claude) | С микроядром AnD os | Эффект |
| :--- | :---: | :---: | :---: |
| **Расход токенов на аудит** | ~35 000 токенов | **~420 токенов** | **-98.8%** |
| **Стабильность контекста** | Деградация через 8–15 шагов | **Стабилен 100+ шагов** | **В 10 раз дольше** |
| **Скрытые падения (`except: pass`)** | Регулярно | **0% (блокируется AST)** | **Абсолютная строгость** |
| **Время верификации среды** | 45–90 сек | **< 3.5 сек** | **В 20 раз быстрее** |
| **Внешние зависимости** | Требуются API / сервисы | **Zero-Dependency (Stdlib)** | **100% автономность** |

---

## Репозитории проекта

- 🏛️ **Микроядро AnD os Core:** [github.com/KornAnd/and-os-core](https://github.com/KornAnd/and-os-core) (Стандартная библиотека Python, лицензия MIT)
- 🧩 **Автономные навыки AnD os Skills:** [github.com/KornAnd/and-os-skills](https://github.com/KornAnd/and-os-skills) (Каталог навыков для Claude Code, Cursor, OpenCode и Gemini)

---
**System Architect: Andrey Korenkov**  
*AnD os Project | 2026*
