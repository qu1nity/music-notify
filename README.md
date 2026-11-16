# music-notify

Небольшой bash-скрипт, который следит за тем, что играет в браузере (Chromium-подобные
браузеры через MPRIS), и показывает всплывающее уведомление с обложкой, исполнителем
и названием трека — в стиле foobar2000.

A small bash script that watches what is playing in the browser (Chromium-based browsers
via MPRIS) and shows a pop-up notification with artwork, artist and track title.

[Русский](#русский) · [English](#english)

---

## Русский

### Что делает

- Ищет активный MPRIS-плеер с именем, начинающимся на `chromium.instance`
  (Chrome, Chromium, Brave, Vivaldi, Edge и т.п. через `playerctl`).
- Постоянно читает метаданные: `artist`, `title`, `mpris:artUrl`.
- Ждёт, пока один и тот же трек продержится 3 подряд опроса (защита от ложных
  срабатываний при рекламе, переключениях и подгрузке метаданных).
- Для обложек по `http(s)://` скачивает картинку через `curl` в `/tmp/music-notify-art`
  и кэширует её по SHA-256 от URL. Локальные `file://`-обложки используются как есть.
- Убирает старые уведомления (`makoctl dismiss --all`) и показывает новое через
  `notify-send` с фиксированным `--replace-id`, низким приоритетом и временем жизни 4 секунды.
- Не запускается второй раз: использует `flock` на `/tmp/music-notify.lock`.

### Зависимости

| Зависимость | Зачем |
| --- | --- |
| `playerctl` | чтение MPRIS-плееров и метаданных |
| `notify-send` (`libnotify`) | показ уведомлений |
| `curl` | скачивание обложек по HTTP(S) |
| `flock` (`util-linux`) | защита от повторного запуска |
| `sha256sum` (`coreutils`) | имя файла кэша обложки |
| `mako` (`makoctl`) | закрытие старых уведомлений (необязательно) |

Стандартные утилиты `grep`, `sleep`, `mkdir` и т.д. уже есть в системе.

Установка (пример для Arch Linux):

```bash
sudo pacman -S playerctl libnotify curl util-linux mako
```

### Запуск

```bash
./music-notify
```

Скрипт работает в бесконечном цикле, поэтому его удобно держать как systemd-сервис
пользователя.

Пример `~/.config/systemd/user/music-notify.service`:

```ini
[Unit]
Description=Music notification daemon
After=graphical-session.target

[Service]
ExecStart=%h/music-notify/music-notify
Restart=always

[Install]
WantedBy=default.target
```

```bash
systemctl --user enable --now music-notify.service
```

### Настройка

Основные параметры задаются в начале скрипта:

```bash
PLAYER_PREFIX="chromium.instance"   # какие плееры отслеживать
NOTIFICATION_ID=13371488            # id уведомления для --replace-id
LOCK_FILE="/tmp/music-notify.lock"  # lock-файл
ART_DIR="/tmp/music-notify-art"     # кэш обложек
```

### Примечания

- Если уведомления не появляются, проверьте, что браузер отдаёт MPRIS:
  `playerctl -l` должен показывать `chromium.instance...`.
- Кэш обложек лежит в `/tmp` и очищается системой сам.
- Имя приложения в уведомлении жёстко задано как `soundcloud` — поменяйте `--app-name`
  в скрипте, если нужно другое.

---

## English

### What it does

- Finds the active MPRIS player whose name starts with `chromium.instance`
  (Chrome, Chromium, Brave, Vivaldi, Edge, etc. via `playerctl`).
- Continuously reads metadata: `artist`, `title`, `mpris:artUrl`.
- Waits until the same track stays for 3 consecutive polls (prevents false positives
  from ads, skipping and delayed metadata).
- For `http(s)://` artwork it downloads the image with `curl` into `/tmp/music-notify-art`
  and caches it by the SHA-256 of the URL. Local `file://` artwork is used as is.
- Dismisses old notifications (`makoctl dismiss --all`) and shows a new one via
  `notify-send` with a fixed `--replace-id`, low urgency and a 4-second timeout.
- Refuses to run twice: uses `flock` on `/tmp/music-notify.lock`.

### Dependencies

| Dependency | Purpose |
| --- | --- |
| `playerctl` | read MPRIS players and metadata |
| `notify-send` (`libnotify`) | show notifications |
| `curl` | download artwork over HTTP(S) |
| `flock` (`util-linux`) | prevent duplicate instances |
| `sha256sum` (`coreutils`) | artwork cache filename |
| `mako` (`makoctl`) | dismiss old notifications (optional) |

Common tools like `grep`, `sleep`, `mkdir` are already present on the system.

Install example (Arch Linux):

```bash
sudo pacman -S playerctl libnotify curl util-linux mako
```

### Usage

```bash
./music-notify
```

The script runs in an infinite loop, so it is best kept as a user systemd service.

Example `~/.config/systemd/user/music-notify.service`:

```ini
[Unit]
Description=Music notification daemon
After=graphical-session.target

[Service]
ExecStart=%h/music-notify/music-notify
Restart=always

[Install]
WantedBy=default.target
```

```bash
systemctl --user enable --now music-notify.service
```

### Configuration

The main parameters are defined at the top of the script:

```bash
PLAYER_PREFIX="chromium.instance"   # which players to watch
NOTIFICATION_ID=13371488            # notification id for --replace-id
LOCK_FILE="/tmp/music-notify.lock"  # lock file
ART_DIR="/tmp/music-notify-art"     # artwork cache
```

### Notes

- If notifications don't appear, make sure the browser exposes MPRIS:
  `playerctl -l` should list `chromium.instance...`.
- The artwork cache lives in `/tmp` and is cleaned by the system automatically.
- The app name in notifications is hardcoded as `soundcloud` — change `--app-name`
  in the script if you need something else.

---

## Это не плеер / Not a player

Скрипт **не является заменой нормальному музыкальному плееру** и не управляет
воспроизведением. Он лишь показывает уведомление о смене трека, который в данный
момент отслеживает `soundcloud-rpc`-подобный MPRIS-источник.
Для полноценного прослушивания и управления используйте отдельный плеер.

This script is **not a replacement for a proper music player** and does not control
playback. It only shows a notification when the track watched by a soundcloud-rpc-style
MPRIS source changes. Use a dedicated player for actual listening and playback control.

## Лицензия / License

GPL-3.0, см. [LICENSE](LICENSE).
