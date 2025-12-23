# Звіт з практичної роботи №11
**Тема:** Неперервна інтеграція (Continuous Integration)  
**Виконав:** Осипенко Роман ІПЗ-3.01

---

## 1. Виконання курсів GitHub Skills

Було пройдено навчальні курси від GitHub для ознайомлення з основами Actions та Packages.

- **Hello GitHub Actions**: (https://github.com/Soms01/skills-hello-github-actions)
- **Publish Packages**: (https://github.com/Soms01/skills-publish-packages)

---

## 2. Налаштування власного CI/CD Workflow

### Мета
Автоматизувати збірку Docker-образу фронтенд-додатку та його публікацію в реєстр пакетів GitHub Container Registry (GHCR) при кожному пуші в гілку `main`.

### Репозиторій з виконаним завданням
Усі налаштування та код знаходяться за посиланням:  
👉 **(https://github.com/Soms01/Projeck02)**

---

### Конфігурація CI/CD Workflow

**1. Вміст файлу `Dockerfile`:**
Для контейнеризації застосунку створено файл, який описує збірку на базі веб-сервера Nginx:

```dockerfile
FROM nginx:stable-alpine
WORKDIR /app
COPY . .
RUN cp -r /app/dist/* /usr/share/nginx/html
EXPOSE 80
```

Для реалізації автоматизації у директорії `.github/workflows/` було створено файл **`docker-image.yml`**.

**Вміст файлу `.github/workflows/docker-image.yml`:**

```yaml
name: Front-end Build and Publish

on:
  workflow_dispatch:
  push:
    branches:
      - 'main'
      - 'feature/*'

permissions:
  contents: read
  packages: write

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install pnpm, dependencies and build
        run: |
          npm install -g pnpm
          pnpm install
          pnpm run build

      - name: Log in to the Container registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          # Використовуємо ваш тег
          tags: ghcr.io/soms01/projeck02:latest


```

### Результати
В результаті виконання роботи було успішно налаштовано pipeline. Після коміту в репозиторій GitHub Actions автоматично зібрав проект і опублікував Docker-образ.
