# Развёртывание карты на GitHub Pages — пошагово

## Что получится
Публичный URL вида `https://<твой-логин>.github.io/realestate-spb-map/`.
Кто угодно может открыть, но без пароля увидит только форму ввода —
данные зашифрованы, расшифровываются в браузере после ввода правильного пароля.

Лимиты GitHub Pages (бесплатно): 1 ГБ репозиторий, 100 ГБ трафика/мес,
10 деплоев/час. С запасом для одного-двух пользователей.

---

## Один раз: создать репозиторий и подключить GH Pages

### 1. Настроить git (если ещё не настроен)
```
git config --global user.name "Твоё Имя"
git config --global user.email "твой_email@example.com"
```

### 2. Создать пустой публичный repo на GitHub
- Открыть https://github.com/new
- **Repository name**: `realestate-spb-map`
- **Public**
- НЕ ставить галки «Add a README», «Add .gitignore», «Add license» — должен быть полностью пустой
- Жми **Create repository**
- Скопируй URL из появившейся инструкции — выглядит как
  `https://github.com/<твой-логин>/realestate-spb-map.git`

### 3. Залить содержимое папки `deploy/` в этот repo
В терминале, из корня проекта `Офис/`:
```
cd deploy
git init -b main
git add .
git commit -m "Initial deploy"
git remote add origin https://github.com/<твой-логин>/realestate-spb-map.git
git push -u origin main
```

При первом push GitHub спросит логин/пароль или PAT (Personal Access Token).
Если попросит пароль — это не пароль от аккаунта, а PAT, получить тут:
https://github.com/settings/tokens/new (scope `repo`).
Можно один раз в Windows Credential Manager сохранить — потом не спрашивает.

### 4. Включить GitHub Pages
- На странице repo → вкладка **Settings** → раздел **Pages** (слева)
- **Source**: Deploy from a branch
- **Branch**: `main`, папка `/ (root)`
- **Save**
- Через 1–2 минуты вверху появится зелёная плашка с твоим URL.

### 5. Дать пароль маме
Пароль лежит в `Офис/config/map_password.txt`. Покажи ей URL и пароль.
Файл `map_password.txt` в `.gitignore` — в git не попадёт.

---

## Каждый раз: обновить карту новыми данными

Из корня проекта `Офис/`:
```
node scripts/deploy_map.mjs --regenerate
cd deploy
git add .
git commit -m "data update"
git push
```

Или двойной клик по `tools/deploy_map.bat` — он сделает всё это автоматом
(включая git push, если remote уже подключён).

Через ~30 секунд после push GitHub Pages обновит карту.
В URL после деплоя можно увидеть время сборки в правом нижнем углу панели
(«Данные от …»).

---

## Сменить пароль
1. Открыть `Офис/config/map_password.txt`, заменить содержимое на новый пароль
2. `node scripts/deploy_map.mjs` (без `--regenerate`, если данные те же)
3. `cd deploy && git add . && git commit -m "rotate password" && git push`
4. Сказать маме новый пароль

Старая ссылка перестанет работать со старым паролем (новый зашифрованный
файл уже на хосте).

---

## Если что-то пошло не так

### При `git push` пишет `repository not found` или `403`
- Проверь URL remote: `git remote -v`
- Если URL неверный: `git remote set-url origin <правильный>`
- Если 403 — проблема с PAT, перевыпусти токен

### GH Pages не открывается / 404
- Settings → Pages: проверь что Source = main / root
- Подожди 2 минуты после первого включения
- В корне `deploy/` должен быть `index.html` (он есть)
- `.nojekyll` тоже должен быть (есть)

### Карта открывается, пароль не подходит
- Сверь содержимое `config/map_password.txt` побайтно (особенно дефисы и регистр)
- Если только что менял пароль — подожди ~30 сек чтобы GH Pages обновился
- В DevTools (F12) → Console: посмотри ошибки

### Слишком большой `points.enc.json` (>100 МБ — лимит файла в GitHub)
Сейчас ~9 МБ, есть огромный запас. Если когда-то перерастёт — можно
сжимать через gzip перед шифрованием. Скажи мне, добавлю.
