# 🧪 pocket-psych

**Testing & editing Psych Engine for mobile** — 32-bit parity, low-end performance, CI-built APKs.

Тут не «ще один форк Psych Engine», а журнал експериментів: кожна зміна має гіпотезу,
метод перевірки й результат — навіть якщо результат «не запрацювало».

> Оригінальний README рушія збережено в [`docs/UPSTREAM-README.md`](docs/UPSTREAM-README.md).

---

## База

Повна копія [`FunkinPortingMobile/Psych-Engine-1.0.4-Mobile`](https://github.com/FunkinPortingMobile/Psych-Engine-1.0.4-Mobile)
(станом на 2026-10-24). Верхівка — Psych Engine 1.0.4: HaxeFlixel → lime/OpenFL → hxcpp → NDK.

`gitVersion.txt` = `1.0.4` · мова: Haxe · збірка: `lime build android -release`

---

## Статус експериментів

| № | Експеримент | Статус | Результат |
|---|---|---|---|
| 001 | Відео на 32-біт (hxvlc + armeabi-v7a) | 🟡 гіпотеза | — |
| 002 | CI-матриця: split APK на ABI | ⬜ черга | — |
| 003 | Storage без `MANAGE_EXTERNAL_STORAGE` | ⬜ черга | — |
| 004 | Low-end perf: frame-time, ASTC vs PNG | ⬜ черга | — |

---

## Що вже відомо (перевірено по сорсах)

**1. На 32-біт відео вимкнене прапорцем у проєкті.**
`Project.xml:29`
```xml
<define name="VIDEOS_ALLOWED" unless="32bits"/>
```
`hxvlc` підключається лише за цим дефайном (`Project.xml:132`), тому в 32-бітній збірці
катсцени й відео в модах недоступні.

**2. Але сама бібліотека 32-біт підтримує.**
`hxvlc/project/Build.xml:41-45` лінкує `libvlc-v7.so` для `HXCPP_ARMV7` — 32-бітний
libvlc існує. Отже обмеження найімовірніше історичне, а не технічне.

**3. lime за замовчуванням збирає ОБИДВА ARM ABI.**
`lime/tools/platforms/AndroidPlatform.hx:91`
```haxe
defaults.architectures = [Architecture.ARMV7, Architecture.ARM64];
```
Тобто «універсальний» APK містить і 32, і 64 біти — і тоді дефайн `32bits` може взагалі
не виставлятись. Перевіряється першим прогоном CI (див. експеримент 001, крок 0).

---

## Швидкий старт

```bash
sh ./setup/mobile.sh                       # залежності
haxelib run lime build android -release    # збірка
```

Локально потрібні haxe 4.3.7, hxcpp (форк FunkinCrew), Android SDK/NDK, JDK 17.
Простіше — GitHub Actions: **Actions → Build Android (ABI report) → Run workflow**.
Після збірки в job summary буде таблиця «скільки нативних `.so` на кожен ABI» і чи є `libvlc` в APK.

---

## Структура

```
.
├─ docs/
│  ├─ UPSTREAM-README.md        # оригінальний README рушія
│  ├─ experiments/              # гіпотеза → метод → заміри → висновок
│  └─ benchmarks/               # сирі цифри: ms, MB, розмір APK
├─ patches/                     # готові .patch (застосовні через git apply)
└─ .github/workflows/           # збірка + звіт про ABI
```

## Правила

- Один експеримент = одна гілка = один PR = один файл у `docs/experiments/`.
- Негативний результат теж публікується. «Не працює, бо X» — це цінність.
- Ніяких «магічних» правок без числа: якщо це про продуктивність — потрібен замір до/після.

## Подяки й ліцензія

Psych Engine — Shadow Mario, Riveren та контриб'ютори; мобільний порт 1.0.4 —
FunkinPortingMobile. Ліцензія успадковується від upstream (Apache-2.0, див. `LICENSE`).
Експерименти й документація в цьому репо — під тією ж ліцензією, якщо не вказано інше.
