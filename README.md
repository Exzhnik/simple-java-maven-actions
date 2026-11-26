# Simple Java Maven App with GitHub Actions

Учебный кроссплатформенный Java-проект для демонстрации автоматической сборки,
тестирования, сохранения артефакта и публикации Docker-образа с помощью
GitHub Actions.

За основу взят открытый проект
[`jenkins-docs/simple-java-maven-app`](https://github.com/jenkins-docs/simple-java-maven-app)
под лицензией MIT. Исходный пример адаптирован для GitHub Actions.

Приложение выводит строку `Hello World!` и содержит два unit-теста.

## Требования

- JDK 21 или новее
- Apache Maven 3.9.9 или новее

## Сборка и тестирование

```bash
mvn clean verify
```

После успешной сборки исполняемый файл появится по пути
`target/my-app-1.0-SNAPSHOT.jar`.

## Запуск

```bash
java -jar target/my-app-1.0-SNAPSHOT.jar
```

Ожидаемый результат:

```text
Hello World!
```

## Docker

Образ собирается многоэтапной сборкой (`Dockerfile`): этап `build` — Maven и
JDK 21 собирают JAR, финальный этап `runtime` — только JRE 21 и JAR, без
исходного кода и инструментов сборки. Приложение запускается от
непривилегированного пользователя.

Локальная сборка и запуск:

```bash
docker build -t my-app .
docker run --rm my-app
```

Опубликованный образ доступен в GitHub Container Registry (GHCR):

```bash
docker pull ghcr.io/exzhnik/simple-java-maven-actions:latest
docker run --rm ghcr.io/exzhnik/simple-java-maven-actions:latest
```

Ожидаемый результат — `Hello World!`.

Теги образа:

- `1.0.<номер запуска>` — версия сборки, например `1.0.42`;
- `sha-<короткий SHA коммита>` — привязка к коммиту;
- `latest` — последняя успешная сборка из `main`.

Тегирование и публикация объединены в один шаг: теги формирует
`docker/metadata-action`, а `docker/build-push-action` собирает образ и
сразу отправляет его в GHCR.

Пакет в GHCR по умолчанию приватный. Чтобы образ скачивался без токена,
сделайте пакет публичным: GitHub → Packages → пакет → Package settings →
Change package visibility. Либо перед скачиванием выполните
`docker login ghcr.io -u <ваш GitHub-логин>` с PAT, имеющим scope
`read:packages`.

## GitHub Actions

Workflow `.github/workflows/ci-cd.yml` запускается при каждом `push` и при
создании или обновлении `pull request`. Пайплайн разделён на три джобы:

- `test` — получает исходный код, устанавливает Temurin JDK 21 с кэшем
  зависимостей Maven и запускает unit-тесты командой `mvn test`;
- `build` — запускается только после успешного прохождения тестов
  (`needs: test`), собирает JAR командой `mvn package -DskipTests` без
  повторного запуска тестов и сохраняет его как скачиваемый артефакт на 14 дней;
- `docker` — запускается после успешной сборки (`needs: build`), собирает
  Docker-образ из `Dockerfile` и публикует его в GHCR. Использует
  `docker/setup-buildx-action`, `docker/login-action` (вход через встроенный
  `GITHUB_TOKEN`, ручные секреты не нужны), `docker/metadata-action` (теги) и
  `docker/build-push-action`. Образ публикуется только при `push` в `main`;
  для pull request и остальных веток образ только собирается — Dockerfile
  проверяется без публикации.

Если тесты падают, джобы сборки и публикации образа не выполняются.

## Лицензия и источник

Исходный проект распространяется по лицензии MIT. Текст лицензии сохранён в
файле `LICENSE.txt`.

