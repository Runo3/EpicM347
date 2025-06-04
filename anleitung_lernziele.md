Anleitung gemäss den lernzielen
---

# .NET 8 Minimal API mit MongoDB (Docker) auf Ubuntu

Dieses Projekt zeigt, wie du mit .NET 8 eine Minimal API erstellst, MongoDB via Docker containerisierst und CRUD-Endpunkte anbietest. Alle Schritte funktionieren vollständig über das Terminal unter Ubuntu.

---

## 🧰 Voraussetzungen

- Ubuntu 22.04+ (getestet)
- [.NET 8 SDK](https://dotnet.microsoft.com/en-us/download)
- [Docker Engine](https://docs.docker.com/engine/install/ubuntu/)
- (optional) VS Code mit REST Client Extension

---

## 🔧 Schritt 1: Projekt erstellen

```bash
dotnet new webapi -n MongoMinimalApi --no-https
cd MongoMinimalApi
````

Lösche in `Program.cs` den Standardcode und ersetze ihn mit:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/", () => "MongoDB Minimal API läuft!");

app.Run();
```

---

## 🐳 Schritt 2: MongoDB via Docker starten

Erstelle im Projektverzeichnis eine Datei `docker-compose.yml`:

```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:6
    container_name: mongodb
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
```

MongoDB starten:

```bash
docker compose up -d
```

Die Datenbank läuft nun auf:
`mongodb://admin:admin@localhost:27017`

---

## ⚙️ Schritt 3: MongoDB-Anbindung vorbereiten

### 3.1 NuGet-Paket installieren:

```bash
dotnet add package MongoDB.Driver
```

### 3.2 Konfiguration in `appsettings.json` anlegen:

```json
{
  "MongoDb": {
    "ConnectionString": "mongodb://admin:admin@localhost:27017",
    "Database": "MyDatabase",
    "Collection": "Items"
  }
}
```

---

## 📦 Schritt 4: Model-Klasse erstellen

Datei: `Models/Item.cs`

```csharp
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;

namespace MongoMinimalApi.Models;

public class Item
{
    [BsonId]
    [BsonRepresentation(BsonType.ObjectId)]
    public string? Id { get; set; }

    [BsonElement("name")]
    public string Name { get; set; } = "";

    [BsonElement("price")]
    public double Price { get; set; }
}
```

---

## 🛠 Schritt 5: MongoDB-Service

Datei: `Services/MongoService.cs`

```csharp
using MongoDB.Driver;
using MongoMinimalApi.Models;

namespace MongoMinimalApi.Services;

public class MongoService
{
    private readonly IMongoCollection<Item> _items;

    public MongoService(IConfiguration configuration)
    {
        var client = new MongoClient(configuration["MongoDb:ConnectionString"]);
        var database = client.GetDatabase(configuration["MongoDb:Database"]);
        _items = database.GetCollection<Item>(configuration["MongoDb:Collection"]);
    }

    public async Task<List<Item>> GetAllAsync() => await _items.Find(_ => true).ToListAsync();
    public async Task<Item?> GetByIdAsync(string id) => await _items.Find(x => x.Id == id).FirstOrDefaultAsync();
    public async Task CreateAsync(Item item) => await _items.InsertOneAsync(item);
    public async Task UpdateAsync(string id, Item item) => await _items.ReplaceOneAsync(x => x.Id == id, item);
    public async Task DeleteAsync(string id) => await _items.DeleteOneAsync(x => x.Id == id);
}
```

---

## 🚀 Schritt 6: CRUD-Endpunkte definieren

Datei: `Program.cs`

```csharp
using MongoMinimalApi.Models;
using MongoMinimalApi.Services;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSingleton<MongoService>();

var app = builder.Build();

app.MapGet("/", () => "MongoDB Minimal API läuft!");

// CRUD-Endpunkte
app.MapGet("/items", async (MongoService db) => await db.GetAllAsync());

app.MapGet("/items/{id}", async (string id, MongoService db) =>
    await db.GetByIdAsync(id) is Item item ? Results.Ok(item) : Results.NotFound());

app.MapPost("/items", async (Item item, MongoService db) =>
{
    await db.CreateAsync(item);
    return Results.Created($"/items/{item.Id}", item);
});

app.MapPut("/items/{id}", async (string id, Item item, MongoService db) =>
{
    var existing = await db.GetByIdAsync(id);
    if (existing is null) return Results.NotFound();

    item.Id = id;
    await db.UpdateAsync(id, item);
    return Results.NoContent();
});

app.MapDelete("/items/{id}", async (string id, MongoService db) =>
{
    var existing = await db.GetByIdAsync(id);
    if (existing is null) return Results.NotFound();

    await db.DeleteAsync(id);
    return Results.NoContent();
});

app.Run();
```

---

## 🧪 Schritt 7: API testen mit REST Client

Erstelle eine Datei `requests.http` im Projekt:

```http
@baseUrl = http://localhost:5000

### Alle Items abrufen
GET {{baseUrl}}/items

### Neues Item erstellen
POST {{baseUrl}}/items
Content-Type: application/json

{
  "name": "Beispielprodukt",
  "price": 14.99
}

### Einzelnes Item abrufen
GET {{baseUrl}}/items/{id}

### Item aktualisieren
PUT {{baseUrl}}/items/{id}
Content-Type: application/json

{
  "name": "Geändert",
  "price": 19.99
}

### Item löschen
DELETE {{baseUrl}}/items/{id}
```

In VS Code kannst du dann direkt auf `Send Request` klicken.

---

## ▶️ API starten

```bash
dotnet run --urls http://localhost:5000
```

---

## ✅ Zusammenfassung

| Schritt                    | Status |
| -------------------------- | ------ |
| .NET 8 Minimal API         | ✅      |
| MongoDB via Docker         | ✅      |
| Datenmodell erstellt       | ✅      |
| MongoService implementiert | ✅      |
| CRUD-Endpunkte definiert   | ✅      |
| REST getestet              | ✅      |

---

## 🧼 Container stoppen (optional)

```bash
docker compose down
```

---

## 📁 Projektstruktur (vereinfacht)

```text
MongoMinimalApi/
├── Models/
│   └── Item.cs
├── Services/
│   └── MongoService.cs
├── Program.cs
├── appsettings.json
├── docker-compose.yml
├── requests.http
```

---

## ℹ️ Hinweis

Die MongoDB läuft mit Benutzer `admin` und Passwort `admin` in einer nicht-produktiven Umgebung. In Produktion solltest du sichere Zugangsdaten und Authentifizierungsmechanismen nutzen.


---

Wenn du willst, kann ich dir daraus auch ein GitHub-Repository mit `.gitignore`, `.editorconfig`, und Docker-Setup generieren. Sag einfach Bescheid.
```
