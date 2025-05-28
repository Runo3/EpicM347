Hier ist eine vollständige Markdown-Zusammenfassung mit Erklärungen und Lösungen zu allen Lernzielen für die 2. Prüfung im **Modul 347**.

---

````markdown
# Prüfungsvorbereitung Modul 347

**Themen:**  
Dockerfile, YAML, Docker Compose, Docker Secrets, Microservices

---

## 🐳 Dockerfile

### ✅ Multistage-Container verstehen
Multistage-Container helfen, kleinere und sicherere Images zu bauen, indem sie den Build-Prozess in mehreren Schritten aufteilen.

### ✅ Multistage-Container erstellen (Beispiel)
```Dockerfile
# Build Stage
FROM node:18 as build
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Production Stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
````

### ✅ Fehler im Dockerfile finden (Typische Beispiele)

* `COPY` oder `ADD` mit falschem Pfad
* `RUN` Befehle, die in einer falschen Reihenfolge ausgeführt werden
* kein `EXPOSE` bei Webservern
* fehlendes `CMD` oder `ENTRYPOINT`

---

## 📄 YAML

### ✅ Aufbau eines YAML-Files verstehen

YAML = "YAML Ain’t Markup Language"
Wichtig: Einrückungen mit Leerzeichen, keine Tabs!

**Beispiel:**

```yaml
version: "3.9"
services:
  web:
    image: nginx
    ports:
      - "80:80"
```

---

## 🧩 Docker Compose

### ✅ Wichtige Schlüsselwörter in `docker-compose.yml`

* `image`, `build`, `container_name`, `restart`, `environment`, `volumes`, `ports`, `expose`, `networks`, `secrets`

### ✅ Multi-Container-Anwendung orchestrieren

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
  db:
    image: mongo
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

### ✅ .NET-Projekt containerisieren (Beispiel)

```Dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app .
ENTRYPOINT ["dotnet", "DeinProjekt.dll"]
```

```yaml
version: "3.9"
services:
  app:
    build: .
    ports:
      - "5000:80"
```

### ✅ Fehler in `docker-compose.yml` finden

* Falsche Einrückung
* Ports nicht als Strings angegeben: `"80:80"` statt `80:80`
* fehlendes `version` oder `services`-Block

### ✅ Netzwerk konfigurieren

```yaml
services:
  frontend:
    image: react-app
    networks:
      - mynet
  backend:
    image: api-app
    networks:
      - mynet

networks:
  mynet:
```

### ✅ `docker compose up/down`, Optionen `-d`, `--build`

```bash
docker compose up -d        # startet Container im Hintergrund
docker compose up --build   # baut Images neu vor dem Start
docker compose down         # stoppt und entfernt Container
```

---

## 🔐 Docker Secrets

### ✅ Verwendung von Secrets

Nur mit Docker Swarm nutzbar (nicht in einfachem Docker Compose ohne Swarm):

**Secret erstellen:**

```bash
echo "geheimes-passwort" | docker secret create db_password -
```

**docker-compose.yml Beispiel:**

```yaml
services:
  db:
    image: mysql
    secrets:
      - db_password

secrets:
  db_password:
    external: true
```

---

## 🧱 Microservices

### ✅ Begriff Microservice + Vor- und Nachteile

**Definition:**
Ein Microservice ist ein kleiner, unabhängig deploybarer Dienst, der eine spezifische Funktion erfüllt.

**Vorteile:**

* Skalierbarkeit
* Einfachere Wartung
* Unabhängige Entwicklung

**Nachteile:**

* Komplexere Infrastruktur (Netzwerk, Logging, etc.)
* Datenkonsistenz schwerer
* mehr DevOps-Wissen nötig

---

## 🧪 Ablauf der Prüfung

* **Dauer:** ca. 90 Minuten
* **Form:**

  * **Teil 1:** Schriftlich
  * **Teil 2:** Praktisch (Open Book, Internet, ohne Kommunikation/KI)
* **Hilfsmittel:**

  * Teil 1: Keine
  * Teil 2: Internet & eigene Geräte erlaubt (ChatGPT verboten)

---

```

Wenn du möchtest, kann ich daraus ein PDF generieren oder dir auch eine interaktive Lernkarte/Quiz-Version erstellen. Sag einfach Bescheid!
```
