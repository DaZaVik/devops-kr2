# Контрольная работа №2 — вариант 3

Заботкин Данила Викторович, ЭФБО-17-24, номер по списку 11.
Ресурс: `https://stats.devops.local`, порт приложения на VM: `127.0.0.1:8003`.

Исходный шаблон: https://gitverse.ru/dgimatdinov/kr2-template,
коммит `3ee6c4bfda24df8cb4d61f8a27031813b3238338`. Исходная main сохранена;
исправления находятся в `kr2/variant-3`. Логика app.py и requirements.txt сохранены.
Таблица девяти дефектов: [docs/defects.md](docs/defects.md).

## Развёртывание на подготовленной VM

С хоста выполнить сначала пробную доставку, затем фактическую:

```bash
rsync -avz --delete --exclude=.env --exclude=.git --chmod=D755,F644 --dry-run ./ devops:kr2/
rsync -avz --delete --exclude=.env --exclude=.git --chmod=D755,F644 ./ devops:kr2/
```

На VM однократно создать секрет, не выводя его на экран:

```bash
cd ~/kr2
umask 077
test -f .env || printf 'REDIS_PASSWORD=%s\n' "$(openssl rand -hex 16)" > .env
chmod 600 .env
docker compose up -d --build --wait
docker compose ps
```

При недоступности CDN Docker Hub допустим публичный кэш Docker Hub от Google:

```bash
docker pull mirror.gcr.io/library/redis:7-alpine
docker tag mirror.gcr.io/library/redis:7-alpine redis:7-alpine
```

Сертификат SAN `stats.devops.local` создаётся на VM. Ключ остаётся в
`/etc/ssl/private/kr2.key`, владелец root, права 600. Конфигурация
[config/kr2.conf](config/kr2.conf) устанавливается в sites-available/kr2
и включается ссылкой sites-enabled/kr2. Перед reload выполнить `sudo nginx -t`.
Правила UFW и ресурс devops-site не менять.

## Проверено 29.10.2026

- UID app и Redis — 999; exec-CMD; остановка тестового контейнера 0,558 с.
- `.env` отсутствует в итоговом образе и текущей версии Git; пароль существует только на VM.
- Redis healthy, без пароля отвечает NOAUTH; 6379 не опубликован.
- Счётчик 4 → 5 после down/up без `-v`.
- HTTP 301, HTTPS с `--cacert`, `/whoami` со scheme=https; прямой 8003 с хоста недоступен.
- UFW и SHA256 devops-site не изменились; аудит 25/25, отрицательный тест публикации дал 1.

PR **не сливать до защиты**. Локальные адреса не являются публикацией в Интернете.
Самоподписанный сертификат требует явного доверия клиента. Очная защита ещё не выполнялась.
