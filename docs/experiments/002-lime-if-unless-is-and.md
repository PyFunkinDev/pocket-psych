# Експеримент 002 — `if`/`unless` у lime: пробіл = AND, а не OR

**Статус:** 🟢 підтверджено
**Дата:** 2026-10-24
**Коміт:** `5ceabd1`

---

## Що сталося

Перший прогін CI падав на:
```
Error: Could not find haxelib "hxdiscord_rpc", does it need to be installed?
```

Ланцюжок причин:
1. `setup/mobile.sh` клонив `hxdiscord_rpc` через **SSH** (`git@github.com:...`) → у CI немає SSH-ключа → `Could not clone Git repository.`
2. Репо `FunkinPortingMobile/hxdiscord_rpc-android-support` віддає **404** навіть з токеном → публічно недоступне.
3. `Project.xml:133` вимагає цей haxelib, бо `DISCORD_ALLOWED` був увімкнений на Android.

## Головна знахідка ⚠️

Перша спроба виправити — змінити на `unless="ios mobile"`. **Не спрацювало.**

У lime умови в атрибутах `if`/`unless` розділені **пробілом означають AND**, не OR:

```xml
unless="ios mobile"   →   unless (ios AND mobile)   →   на Android це false
                      →   дефайн ЛИШАЄТЬСЯ увімкненим
```

Тому правильний варіант — явний `if` з одним атомарним дефайном:

```xml
<define name="DISCORD_ALLOWED" if="desktop" />
```

Це підтверджується й іншими рядками того ж `Project.xml`:
`if="desktop officialBuild"` (рядок 108) і `if="VIDEOS_ALLOWED debug"` (144) — скрізь AND-семантика.

## Висновок

- Discord Rich Presence на Android неможливий за визначенням: немає локального клієнта Discord,
  з яким говорити по IPC. Тому `if="desktop"` — не «заглушка», а правильна семантика.
- Побічний ефект: з мобільної збірки зникає залежність і кілька сотень KB коду.
- **Правило на майбутнє:** для lime-проєктів ніколи не покладайся на пробіл як на OR.
  Або `||`, або один атомарний дефайн.

## Як відтворити

```bash
grep -n 'DISCORD_ALLOWED' Project.xml      # має бути if="desktop"
git log --oneline -- Project.xml | head -3
```

---

_Пов'язано з: [`000-template.md`](000-template.md)_
