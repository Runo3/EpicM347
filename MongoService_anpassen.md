# Q: Wie muss man das MongoService.cs an die MongoDB Strucktur anpassen?

A:
Die `MongoService.cs` muss an die **Datenstruktur deiner MongoDB** angepasst werden, indem du:

1. **Den Namen der Datenbank**
2. **Den Namen der Collection**
3. **Das Datenmodell (Klasse)**

korrekt definierst und sicherstellst, dass sie zur realen MongoDB-Datenstruktur passen.

---

## 🔧 Anpassungen im Detail

### 1. **Datenbank- und Collection-Namen**

Diese Werte kommen typischerweise aus `appsettings.json`:

```json
{
  "MongoDb": {
    "ConnectionString": "mongodb://admin:admin@localhost:27017",
    "Database": "MyDatabase",
    "Collection": "Items"
  }
}
```

Wenn deine Datenbank z. B. `shop` heißt und die Collection `products`, musst du `appsettings.json` anpassen:

```json
{
  "MongoDb": {
    "ConnectionString": "mongodb://admin:admin@localhost:27017",
    "Database": "shop",
    "Collection": "products"
  }
}
```

### 2. **Modell-Klasse anpassen**

Die Modell-Klasse `Item.cs` muss **den Feldern in deinen MongoDB-Dokumenten entsprechen**.

Beispiel: Dein Dokument in MongoDB sieht so aus:

```json
{
  "_id": "665d75e...",
  "title": "Tastatur",
  "price": 49.99,
  "inStock": true
}
```

Dann muss `Item.cs` wie folgt aussehen:

```csharp
public class Item
{
    [BsonId]
    [BsonRepresentation(BsonType.ObjectId)]
    public string? Id { get; set; }

    [BsonElement("title")]
    public string Title { get; set; } = "";

    [BsonElement("price")]
    public double Price { get; set; }

    [BsonElement("inStock")]
    public bool InStock { get; set; }
}
```

**Wichtig:** Die Feldernamen müssen mit den Feldnamen in MongoDB übereinstimmen – oder du verwendest `[BsonElement("...")]`, um sie zu mappen.

---

### 3. **MongoService.cs bleibt fast gleich**, solange du die Collection richtig referenzierst:

```csharp
var database = client.GetDatabase(configuration["MongoDb:Database"]);
_items = database.GetCollection<Item>(configuration["MongoDb:Collection"]);
```

Wenn du mehrere Collections hast (z. B. `users`, `orders`), musst du für jede Collection eine eigene Klasse und ggf. einen eigenen Service schreiben – oder du generalisierst das Repository (z. B. mit Generics).

---

## ✅ Zusammenfassung

| Was ändern?                | Wo?                | Beispiel                                       |
| -------------------------- | ------------------ | ---------------------------------------------- |
| DB-Name & Collection       | `appsettings.json` | `"Database": "shop", "Collection": "products"` |
| Felder der Mongo-Dokumente | `Item.cs`          | `public string Title { get; set; }` etc.       |
| Datenzugriff               | `MongoService.cs`  | Bezieht sich auf `Item`-Collection             |

---
