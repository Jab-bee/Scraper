Aquí tienes un **README.md** para documentar tu proyecto Django Scraper con todos los pasos que has descrito, listo para que alguien pueda seguirlo desde cero:

---

# 🕷️ Django Scraper

Este proyecto es un ejemplo de cómo crear un **scraper web en Django**, organizado como un servicio y activado mediante un comando de `manage.py`.

---

## 🚀 Requisitos previos

* Tener instalado **Python 3**
* Tener instalado **pip**
* Opcional: [DB Browser for SQLite](https://sqlitebrowser.org/) para explorar la base de datos

---

## 📦 Instalación paso a paso

```bash
# Crear carpeta del proyecto
mkdir webscraper
cd webscraper

# Crear .gitignore (usa https://www.toptal.com/developers/gitignore -> Windows, Django)
touch .gitignore

# Crear entorno virtual
python3 -m venv venv
# o en Windows
python -m venv venv

# Activar entorno virtual
# macOS / Linux (bash/zsh)
source venv/bin/activate
# Windows (bash)
source venv/Scripts/activate

# Instalar dependencias principales
pip install django
pip install selenium
pip install webdriver-manager

# Congelar dependencias
pip freeze > requirements.txt
cat requirements.txt
```

---

## ⚙️ Crear proyecto Django

```bash
django-admin startproject webscraper_project
cd webscraper_project

# Crear la app de scraping
python3 manage.py startapp scraper
```

Editar `webscraper_project/settings.py` y añadir la app:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'scraper',  # nueva app
]
```

---

## 🗄️ Modelo para guardar datos

En `scraper/models.py`:

```python
from django.db import models

class ScrapedData(models.Model):
    title = models.CharField(max_length=200)
    url = models.URLField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

Aplicar migraciones:

```bash
python3 manage.py makemigrations
python3 manage.py migrate
```

---

## 🛠️ Crear servicio de scraping

```bash
cd scraper
mkdir services
touch services/__init__.py
touch services/scrape.py
```

En `scraper/services/scrape.py`:

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager

def scrape_website():
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--no-sandbox')
    options.add_argument('--disable-dev-shm-usage')

    service = Service(ChromeDriverManager().install())
    driver = webdriver.Chrome(service=service, options=options)

    url = "https://jorgebenitezlopez.com"
    driver.get(url)

    try:
        WebDriverWait(driver, 10).until(
            EC.presence_of_all_elements_located((By.CSS_SELECTOR, "h1"))
        )
        titles = driver.find_elements(By.CSS_SELECTOR, "h1")
        urls = driver.find_elements(By.CSS_SELECTOR, "a")
    except Exception as e:
        print("Error al encontrar los elementos:", e)
        driver.quit()
        return []

    scraped_data = []
    for title, link in zip(titles, urls):
        scraped_data.append({
            "title": title.text,
            "url": link.get_attribute("href"),
        })

    driver.quit()
    return scraped_data
```

---

## ⚡ Crear comando de scraping

```bash
mkdir -p scraper/management/commands
touch scraper/management/__init__.py
touch scraper/management/commands/__init__.py
touch scraper/management/commands/scraper.py
```

En `scraper/management/commands/scraper.py`:

```python
from django.core.management.base import BaseCommand
from scraper.services.scrape import scrape_website
from scraper.models import ScrapedData

class Command(BaseCommand):
    help = "Run the web scraper"

    def handle(self, *args, **kwargs):
        data = scrape_website()
        print("Scraped Data:", data)
        for item in data:
            ScrapedData.objects.create(title=item["title"], url=item["url"])
        self.stdout.write(self.style.SUCCESS("Scraping completed!"))
```

---

## ▶️ Ejecutar el scraper

```bash
python3 manage.py scraper
```

Los datos quedarán guardados en **SQLite** (`db.sqlite3`). Puedes inspeccionarlos con DB Browser.

---

## 🔧 Personalización

* Cambia el **selector CSS** en `scrape.py` para extraer otros datos.
* Modifica el **modelo** en `models.py` para guardar más campos.
* Agrega **cronjobs o Celery** si quieres programar el scraper automáticamente.

---

## 📂 Estructura del proyecto

```
webscraper/
├── venv/
├── requirements.txt
├── webscraper_project/
│   ├── manage.py
│   ├── webscraper_project/
│   └── scraper/
│       ├── models.py
│       ├── services/
│       │   └── scrape.py
│       └── management/
│           └── commands/
│               └── scraper.py
└── db.sqlite3
```

