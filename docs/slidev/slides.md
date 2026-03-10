---
theme: default
colorSchema: light
aspectRatio: 17/10
title: Dev Containers + AI Agents + Cloudflare Tunnel
info: |
  Slidev deck based on docs/CONTAINERS-DEMO.md
mdc: true
drawings:
  persist: false
transition: fade
---

<style src="./styles.css"></style>
<div class="hero-shell">
  <div class="eyebrow">📦 Dev Containers • 🤖 AI Agents • 🌐 Cloudflare Tunnel</div>
  <h1><code>Dev Container + AI + Tunnel</code></h1>
  <p class="text-lg">на прикладі NestJS + NextJS + Postgres + Adminer + Mailcatcher</p>
  <p class="hero-author">Ігор Дубій</p>
  <p class="hero-link"><a href="https://github.com/unibrix/dev-container-boilerplate">https://github.com/unibrix/dev-container-boilerplate</a></p>
</div>

---

## 🚨 Problem Statement

<div class="soft-grid">
  <div class="soft-card">
    <h3>🔀 Drift середовищ</h3>
    <p>різні версії Node, package manager, system libs, shell-tools. Нестабільний onboarding і непередбачуваний time-to-first-run.</p>
  </div>
  <div class="soft-card">
    <h3>🔓 Security debt</h3>
    <p>запуск незнайомих <code>npm</code> скриптів і CLI-команд напряму на host OS. Наслідок: ризик втрати локальних секретів, credential leak, компрометація персонального середовища.</p>
  </div>
  <div class="soft-card">
    <h3>🔗 Integration friction:</h3>
    <p>без стабільного HTTPS URL важко тестувати OAuth callbacks і webhooks. Наслідок: тимчасові костилі, ручні обхідні рішення, повільні інтеграційні ітерації.</p>
  </div>
  <div class="soft-card">
    <h3>🤖 AI execution risk</h3>
    <p>агент з full-access (yolo) може виконати руйнівну або небезпечну команду. Наслідок: небажані зміни у файлах, конфігах, git-стані та потенційний security incident на хості.</p>
  </div>
</div>

---

## 🧱 Why This Stack

<div class="soft-grid">
  <div class="soft-card"><p>Це практичний стек, який я використовую в роботі і пропоную як шаблон для ознайомлення.</p></div>
  <div class="soft-card"><p>Швидкий старт як для нового, так і для існуючого проєкту: <code>Reopen in Container</code> замість ручного сетапу.</p></div>
  <div class="soft-card"><p>Краща безпека: незнайомий код/пакети/CLI виконуються в контейнері, а не напряму на host OS.</p></div>
  <div class="soft-card"><p>Проєктний набір інструментів і плагінів фіксується в репозиторії: runtime + VS Code settings/extensions.</p></div>
  <div class="soft-card span-2"><p>AI-асистент з широкими правами працює в керованому середовищі, де простіше застосувати guardrails.</p></div>
</div>

<img src="/devcontainer.png" alt="devcontainer" />

---

```yaml
layout: two-cols-header
class: compact-code
```

## ⚙️ `devcontainer.json` за 60 секунд

::left::

```jsonc
{
  "name": "unibrix-dev (...)", // Людська назва середовища у VS Code
  "dockerComposeFile": ["docker-compose.yml"], // Звідки беруться сервіси
  "service": "backend", // Контейнер, в який під'єднується VS Code
  "workspaceFolder": "/workspaces/app", // Root робочої директорії в контейнері
  "shutdownAction": "stopCompose", // Що робити з compose-стеком при закритті
  "remoteUser": "node", // Non-root user для щоденної роботи
  "mounts": [
    "type=bind,source=${localEnv:HOME}/.codex,target=/home/node/.codex",
    "type=bind,source=${localEnv:HOME}/.gitconfig,target=/home/node/.gitconfig,readonly",
  ], // Межа між host і контейнером
  "forwardPorts": [4242, 3003, 1080, 8080, 9229], // Які порти доступні на хості
  "portsAttributes": {
    // опціонально, опис портів для VSCode
    "4242": { "label": "Backend API", "onAutoForward": "notify" },
  },
  "customizations": {
    "vscode": {
      "extensions": ["..."], // Проєктні extensions
      "settings": { "...": "..." }, // Проєктні VS Code settings
    },
  },
  "postCreateCommand": "node -v && npm -v", // Команда після створення контейнера
}
```

::right::

<div class="soft-card bullet-clean code-notes">
  <ul>
    <li><code>.devcontainer/devcontainer.json</code>: service, mounts, extensions, settings, ports.</li>
    <li><code>.devcontainer/docker-compose.yml</code>: app/data services.</li>
    <li><code>.devcontainer/Dockerfile</code>: базовий image + toolchain (<code>@openai/codex</code>).</li>
    <li><code>.devcontainer/.env</code> і <code>.env.example</code>: конфіг і секрети.</li>
  </ul>
</div>

---


## 🛡️ Threat Model

<div class="risk-grid">
  <div class="soft-card bullet-clean">
    <h3>📦 Supply chain через <code>postinstall</code>/<code>prepare</code> у залежностях.</h3>
    <ul>
      <li><strong>Сценарій:</strong> пакет виконує shell, читає файли/змінні середовища, робить outbound exfiltration.</li>
      <li><strong>Impact:</strong> витік токенів, сесійних ключів, конфігураційних секретів.</li>
    </ul>
  </div>
  <div class="soft-card bullet-clean">
    <h3>🧨 Шкідливий код у самому репозиторії (<code>scripts</code>, hooks, helper CLI).</h3>
    <ul>
      <li><strong>Сценарій:</strong> <code>npm run dev</code> або <code>npm run setup</code> запускає приховані side-effects.</li>
      <li><strong>Impact:</strong> несанкціоновані зміни, бекдори в локальному середовищі, data leakage.</li>
    </ul>
  </div>
  <div class="soft-card bullet-clean">
    <h3>🤖 AI agent з широкими правами в терміналі/FS.</h3>
    <ul>
      <li><strong>Сценарій:</strong> виконання небезпечних команд, не контрольовані масові зміни.</li>
      <li><strong>Impact:</strong> втрата даних, небажані зміни в хост системі, потенційний security incident.</li>
    </ul>
  </div>
</div>

---

## 📦 Dev Container: Scope та Межі

<div class="soft-grid">
  <div class="soft-card bullet-clean">
    <h3>✅ Це:</h3>
    <ul>
      <li>Відтворюване dev-середовище як код (IaC): runtime, tooling, editor setup.</li>
      <li>Швидший onboarding і менше environment drift.</li>
      <li>Кращий контроль ризиків при запуску незнайомого коду.</li>
    </ul>
  </div>
  <div class="soft-card bullet-clean">
    <h3>🚫 Не дає:</h3>
    <ul>
      <li>Оркестрацію для production.</li>
      <li>Повну ізоляцію VM-рівня .</li>
      <li>Гарантію безпеки без додаткових командних правил.</li>
      <li>I/O performance не такий як на хості.</li>
    </ul>
  </div>
  <div class="soft-card bullet-clean span-2">
    <h3>🧭 У цьому репозиторії це реалізовано так:</h3>
    <ul>
      <li><code>.devcontainer/devcontainer.json</code> фіксує <code>service</code>, <code>remoteUser</code>, <code>mounts</code>, <code>forwardPorts</code>, VS Code policy.</li>
      <li><code>.devcontainer/docker-compose.yml</code> фіксує склад сервісів (<code>backend</code>, <code>frontend</code>, <code>db</code>, <code>mailcatcher</code>, <code>adminer</code>, <code>cloudflared</code>).</li>
      <li>Guardrails і anti-patterns винесені в окремі слайди як best practices.</li>
    </ul>
  </div>
</div>

---

## ⚖️ Порівняння підходів

<div class="comparison-shell comparison-tight">

| Критерій (developer view)    | Local Host                                | Dev Container + Compose                            |
| ---------------------------- | ----------------------------------------- | -------------------------------------------------- |
| Onboarding speed             | Нестабільний, залежить від стану host     | Швидкий і повторюваний через `Reopen in Container` |
| Tooling consistency          | Часто різні версії Node/npm/CLI           | Зафіксовано в repo, передбачувано для команди      |
| Context switch між проєктами | Часті конфлікти залежностей і портів      | Ізольовані середовища per-project                  |
| Reproducibility багів        | "Works on my machine" трапляється частіше | Вища відтворюваність сценаріїв                     |
| IDE parity                   | Плагіни і settings у кожного свої         | Проєктна VS Code policy в `devcontainer.json`      |
| Debugging setup              | Ручний attach/налаштування у кожного dev  | Стандартизований сценарій debug                    |
| CI parity                    | Низька                                    | Вища (ближче до контейнеризованих CI job)          |
| Reset/Cleanup                | Складніше прибрати локальний "слід"       | Простий reset через rebuild/down                   |
| Security                     | Низька/середня                            | Середня/вища за умови guardrails                   |
| Local performance            | Краща raw-продуктивність                  | Може бути нижча на bind mounts (macOS/Windows)     |
| Cost (setup/maintenance)     | Низький старт, вищий hidden team cost     | Середній старт, нижчий командний OPEX              |

</div>

---

## 🐳 Чому не просто `docker compose up`

<div class="soft-grid">
  <div class="soft-card"><p>Compose піднімає сервіси, але не стандартизує IDE/tooling поведінку.</p></div>
  <div class="soft-card"><p>Dev Container фіксує VS Code extensions/settings разом із runtime.</p></div>
  <div class="soft-card"><p>Один вхід у середовище: термінал, плагіни, lint/format версії — консистентні.</p></div>
  <div class="soft-card"><p>Менше локального "зоопарку" на host OS.</p></div>
  <div class="soft-card span-2"><p>Типовий підхід: <code>db/mail/adminer</code> у <code>docker-compose</code>, а <code>backend/frontend</code> запускаються на host - не покращує безпеку.</p></div>
</div>

---

```yaml
layout: two-cols-header
class: compact-two-col
```

## 💻 VS Code Workflow

::left::

<div class="soft-card bullet-clean">
  <ul>
    <li>Потрібно: Docker Desktop + VS Code + Dev Containers extension.</li>
    <li>Запуск: <code>Cmd + Shift + P &gt; Dev Containers: Reopen in Container</code>.</li>
    <li>Вихід: <code>Cmd + Shift + P &gt; Dev Containers: Reopen Folder Locally</code>.</li>
    <li>Перезбірка: <code>Dev Containers: Rebuild and Reopen in Container</code>.</li>
  </ul>
</div>

::right::

<div class="soft-card bullet-clean">
  <ul>
    <li>Перший запуск зазвичай довший (pull/build image), наступні — значно швидші.</li>
    <li>Логи сервісів: Docker Desktop (агреговані по всіх контейнерах) або <code>docker compose logs -f &lt;service&gt;</code>.</li>
    <li>Візуальна індикація активного devcontainer через Peacock (<code>peacock.remoteColor</code>) зменшує помилки context-switch.</li>
  </ul>
</div>

---

```yaml
layout: two-cols-header
class: compact-two-col
```

## 🎬 Live Demo

::left::

<div class="soft-card bullet-clean">
  <h3>📦 devcontainer demo</h3>
  <ul>
    <li>Reopen in Container.</li>
    <li>показати, що VS Code settings/extensions підтягнулися автоматично.</li>
    <li>показати <code>docker compose ps</code> і <code>docker compose logs -f backend</code>.</li>
    <li>перевірити локальні endpoints (<code>3003</code>, <code>4242</code>, <code>8080</code>).</li>
  </ul>
</div>

::right::

<div class="soft-card bullet-clean">
  <h3>🌐 cloudflare</h3>
  <ul>
    <li>відкрити frontend URL у браузері.</li>
    <li>пройти Google login і callback на backend.</li>
    <li>показати backend logs з callback-запитом.</li>
    <li>зупинити контейнер і перевірити URL.</li>
  </ul>
</div>

---

## 🌐 Cloudflare Tunnel: навіщо

<div class="soft-card bullet-clean">
  <ul>
    <li>HTTPS endpoints без відкриття inbound портів на роутері.</li>
    <li>Стабільні URL для callback/webhook flows.</li>
    <li>Швидкий демонстраційний доступ для команди або клієнта.</li>
    <li>Менше тимчасових staging-деплоїв для інтеграційних перевірок.</li>
  </ul>
</div>

<img src="/cloudflared.png" alt="cloudflared" />

---

```yaml
layout: two-cols-header
class: compact-two-col
```

## 🛠️ Cloudflare Tunnel: як налаштувати

::left::

<div class="soft-card bullet-clean">
  <ul>
    <li>Створити акаунт Cloudflare (email + пароль) + 2FA.</li>
    <li>Cloudflare Dashboard &gt; Onboard a domain / Add a site, ввести свій домен.</li>
    <li>Обрати Free план.</li>
    <li>У реєстратора домену (наприклад nic.ua з free доменом в зоні pp.ua) замінити NS на ті, що дав Cloudflare.</li>
  </ul>
</div>

::right::

<div class="soft-card bullet-clean">
  <ul>
    <li>Відкрити Cloudflare Zero Trust. Далі Networks → Connectors → Cloudflare Tunnels.</li>
    <li>Create a tunnel → тип конектора Cloudflared → задай назву → Save.</li>
    <li>Токен додай в .devcontainer/.env в CLOUDFLARED_TOKEN</li>
  </ul>
</div>

---

## ✅ Guardrails і Checklists

<div class="guard-grid">
  <div class="soft-card"><p><code>remoteUser</code> non-root за замовчуванням.</p></div>
  <div class="soft-card"><p>Не монтувати host-sensitive шляхи (browser profiles, wallets, ssh keys без потреби).</p></div>
  <div class="soft-card"><p>Не монтувати <code>docker.sock</code> без обґрунтування.</p></div>
  <div class="soft-card"><p>Секрети тільки через env/secret store.</p></div>
  <div class="soft-card"><p>Secret scanning в CI (<code>gitleaks</code>/аналог).</p></div>
  <div class="soft-card"><p>Обов'язковий code review AI-generated змін.</p></div>
  <div class="soft-card span-2"><p>Логування і аудит критичних команд.</p></div>
</div>

---

## 🚫 Anti-patterns (реально болючі)

<div class="anti-grid">
  <div class="soft-card"><p>Коміт <code>.env</code> з токенами/ключами.</p></div>
  <div class="soft-card"><p>Коміт runtime-даних БД (<code>postgres/</code>) у git.</p></div>
  <div class="soft-card"><p>Public tunnel для Adminer/internal UI.</p></div>
  <div class="soft-card"><p>Ручна зміна портів без синхронізації docs.</p></div>
  <div class="soft-card"><p>Дублювання setup кроків у чатах замість конфігу в репозиторії.</p></div>
  <div class="soft-card"><p>"Швидко на хості" замість reproducible container workflow.</p></div>
</div>

---

## 🤝 Closing

<div class="closing-shell bullet-clean">
  <ul>
    <li>Підхід не позиціонується як "єдино правильний".</li>
    <li>Це практичний baseline, який зменшує ризики й прискорює старт.</li>
    <li>Спробуйте репозиторій локально.</li>
    <li>Поверніться з критикою і покращеннями.</li>
  </ul>
</div>

---

```yaml
layout: two-cols-header
class: compact-two-col
```

## 💬 Q&A: Discussion starter

::left::

<div class="soft-card bullet-clean">
  <ul>
    <li>Який у вас зараз baseline: host-only, гібрид (compose + локальний app) чи повний Dev Container?</li>
    <li>Чи прийнятний для вас Public Tunnel для демо, і де межа між demo та internal-only?</li>
    <li>Які локальні сервіси ви б не публікували через тунель?</li>
    <li>Чого вам не вистачає тут?</li>
    <li>Які ризики або заперечення ви бачите для того щоб почати це використовувати?</li>
    <li>Які guardrails для AI agent ви вважаєте обов’язковими?</li>
  </ul>
</div>

::right::

<div class="soft-card bullet-clean">
  <h3>🔗 Посилання</h3>
  <ul>
    <li><a href="https://github.com/unibrix/dev-container-boilerplate">https://github.com/unibrix/dev-container-boilerplate</a></li>
    <li><a href="https://containers.dev">https://containers.dev</a></li>
    <li><a href="https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers">https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers</a></li>
    <li><a href="https://cloudflare.com">https://cloudflare.com</a></li>
  </ul>
</div>
