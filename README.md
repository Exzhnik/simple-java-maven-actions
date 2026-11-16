# Simple Java Maven App with GitHub Actions

Учебный кроссплатформенный Java-проект для демонстрации автоматической сборки,
тестирования и сохранения артефакта с помощью GitHub Actions.

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

## GitHub Actions

Workflow `.github/workflows/ci-cd.yml` запускается при каждом `push` и при
создании или обновлении `pull request`. Джоба `build-and-test`:

1. получает исходный код репозитория;
2. устанавливает Temurin JDK 21;
3. восстанавливает и обновляет кэш зависимостей Maven;
4. запускает unit-тесты командой `mvn test`;
5. собирает JAR командой `mvn package -DskipTests` без повторного запуска тестов;
6. сохраняет JAR как скачиваемый артефакт на 14 дней.

## Лицензия и источник

Исходный проект распространяется по лицензии MIT. Текст лицензии сохранён в
файле `LICENSE.txt`.

