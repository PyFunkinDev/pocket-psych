# Експеримент 001 — відео на 32-біт (hxvlc + armeabi-v7a)

**Статус:** 🟡 гіпотеза
**Гілка:** `exp/001-videos-on-32bit`
**Дата:** 2026-10-24

---

## Гіпотеза

`VIDEOS_ALLOWED` навмисно вимкнений для 32-бітних збірок, хоча `hxvlc` має 32-бітний
libvlc. Якщо прибрати обмеження, відео працюватиме на `armeabi-v7a` без критичних
проблем — ціною, можливо, +X MB до APK і +Y MB RAM.

## Навіщо

На дешевих і старих пристроях (а це саме 32-бітна аудиторія) катсцени в модах
зараз просто не показуються. Модерам доводиться збирати окрему «без відео» версію.

## Точка відліку

- `Project.xml:29` — `<define name="VIDEOS_ALLOWED" unless="32bits"/>`
- `Project.xml:132` — `<haxelib name="hxvlc" if="VIDEOS_ALLOWED" version="2.0.1"/>`
- `Project.xml:86` — `<assets path="assets/videos" if="VIDEOS_ALLOWED"/>`
- `hxvlc/project/Build.xml:43` — `<lib name=".../libvlc-v7.so" if="HXCPP_ARMV7"/>` ← 32-біт існує
- `lime/tools/platforms/AndroidPlatform.hx:91` — дефолтні архітектури `[ARMV7, ARM64]`

## Метод

**Крок 0 — з'ясувати, коли взагалі виставляється `32bits`.**
Це ключове. Якщо lime збирає обидва ABI одним APK, дефайн може не виставлятись узагалі,
і тоді відео вже працює на 32-біт, а проблема лише в single-ABI збірках.
Як перевірити: зібрати APK і подивитись, чи є `assets/videos`, чи є `libvlc-v7.so`.

**Крок 1 — білд з патчем.**
Застосувати `patches/0001-enable-videos-on-32bit.patch`, зібрати:
```bash
haxelib run lime build android -release
```

**Крок 2 — перевірити, що 32-бітний libvlc реально в APK:**
```bash
unzip -l export/release/android/bin/app/build/outputs/apk/release/*.apk \
  | grep -E 'libvlc|armeabi-v7a|arm64-v8a'
```
Очікування: `lib/armeabi-v7a/libvlc-v7.so` + `lib/arm64-v8a/libvlc-64.so`.

**Крок 3 — перевірити на пристрої/емуляторі (32-біт).**
Сценарій: MP4 H.264 у катсцені → відтворення → seek → вихід із катсцени → повторний вхід.

**Крок 4 — заміряти:**

| Метрика | Як міряти |
|---|---|
| Розмір APK | `ls -lh *.apk` до/після |
| RSS у катсцені | `adb shell dumpsys meminfo <pkg>` |
| Падіння | `adb logcat` — шукати SIGSEGV / libvlc |
| Час старту катсцени | логи рушія / таймер від початку до першого кадру |

## Ризики

- libvlc для v7 може не мати частини кодеків/плагінів → частина модів не відтвориться
- 32-бітний процес має ~3 GB адресного простору: великі відео + текстури → OOM
- Можливі падіння в JNI при одночасному аудіо-виводі та відео-виводі

## Фолбек (якщо відео на 32-біт не тягне)

Замість чорного екрана показувати перший кадр відео або прев'ю-зображення з мода.
Це вже покращення UX навіть без відтворення.

## Результат

_Заповнити після прогону._

## Як відтворити

```bash
git checkout exp/001-videos-on-32bit
git apply patches/0001-enable-videos-on-32bit.patch
haxelib run lime build android -release
```
