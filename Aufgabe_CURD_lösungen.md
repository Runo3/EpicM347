# M165 – MongoDB CRUD mit C#

## Lernziele
- Sie erstellen eine Verbindung zwischen einer .NET 8 Anwendung und einer MongoDB-Datenbank.
- Sie führen CRUD-Operationen (Create, Read, Update, Delete) mit dem MongoDB C# Driver durch.

## Umgebung
- **Ubuntu 22.04 VM**
- **Visual Studio Code**
- **Docker mit Container `frm1971/m165-15`**
- **.NET 8 oder höher**
- **MongoDB.Driver NuGet-Paket**



## Inhalt

1. [MongoDB-Container starten](#1-mongodb-container-starten)
2. [.NET 8-Projekt erstellen](#2-net-8-projekt-erstellen)
3. [Verbindung zur MongoDB](#3-verbindung-zur-mongodb)
4. [CRUD-Operationen](#4-crud-operationen)
   - [A) Datenbanken anzeigen](#a-datenbanken-anzeigen)
   - [B) Collections anzeigen](#b-collections-anzeigen)
   - [C) Erster Film von 2012](#c-erster-film-von-2012)
   - [D) Filme mit Pierce Brosnan](#d-filme-mit-pierce-brosnan)
   - [E) Einfügen eines Films](#e-einfügen-eines-films)
   - [F) Einfügen mehrerer Filme](#f-einfügen-mehrerer-filme)
   - [G) Titel-Update](#g-titel-update)
   - [H) Filme löschen](#h-filme-löschen)
   - [I) Anzahl Filme pro Jahr ab 2000](#i-anzahl-filme-pro-jahr-ab-2000)
   - [J) Alle Filme als JSON ausgeben](#j-alle-filme-als-json-ausgeben)

---

## 1. MongoDB-Container starten

```bash
docker run --name m165 -p 27017:27017 -d frm1971/m165-15
````

---

## 2. .NET 8-Projekt erstellen

```bash
dotnet new console --name m165-15
cd m165-15
dotnet add package MongoDB.Driver
dotnet run
```

---

## 3. Verbindung zur MongoDB

```csharp
var client = new MongoClient("mongodb://localhost:27017");
var database = client.GetDatabase("M165");
var collection = database.GetCollection<Movie>("Movies");
```

```csharp
public class Movie
{
    public ObjectId Id { get; set; }
    public string Title { get; set; }
    public int Year { get; set; }
    public string Summary { get; set; }
    public List<string> Actors { get; set; }
}
```

---

## 4. CRUD-Operationen

### A) Datenbanken anzeigen

```csharp
var dbs = await client.ListDatabaseNamesAsync();
await foreach (var name in dbs.ToAsyncEnumerable())
    Console.WriteLine(name);
```

---

### B) Collections anzeigen

```csharp
var collections = await database.ListCollectionNamesAsync();
await foreach (var col in collections.ToAsyncEnumerable())
    Console.WriteLine(col);
```

---

### C) Erster Film von 2012

```csharp
var result = await collection.Find(m => m.Year == 2012).FirstOrDefaultAsync();
Console.WriteLine(result?.Title ?? "Kein Film gefunden.");
```

---

### D) Filme mit Pierce Brosnan

```csharp
var brosnanMovies = await collection.Find(m => m.Actors.Contains("Pierce Brosnan")).ToListAsync();
foreach (var movie in brosnanMovies)
    Console.WriteLine(movie.Title);
```

---

### Aufgabe: Fügen Sie folgenden Film ein:
▪ Title: The Da Vinci Code  
▪ Summary: So dunkel ist der Betrug an der Menschheit  
▪ Actors: Tom Hanks, Audrey Tautou  
▪ Year: 2006  

### E) Einfügen eines Films

```csharp
var newMovie = new Movie {
    Title = "The Da Vinci Code",
    Year = 2006,
    Summary = "So dunkel ist der Betrug an der Menschheit",
    Actors = new List<string> { "Tom Hanks", "Audrey Tautou" }
};

await collection.InsertOneAsync(newMovie);
```

---
### Aufgabe: Fügen Sie folgende beiden Film mit einem Befehl ein:

▪ Title: Ocean’s Eleven  
▪ Summary: Bist du drin oder draussen?  
▪ Actors: George Clooney, Brad Pitt, Julia Roberts  
▪ Year: 2001  
  
▪ Title: The Da Vinci Code  
▪ Summary: Die Elf sind jetzt Zwölf.  
▪ Actors: George Clooney, Brad Pitt, Julia Roberts, Andy Garcia  
▪ Year: 2004    

### F) Einfügen mehrerer Filme

```csharp
var movies = new List<Movie>
{
    new Movie {
        Title = "Ocean’s Eleven",
        Year = 2001,
        Summary = "Bist du drin oder draussen?",
        Actors = new List<string> { "George Clooney", "Brad Pitt", "Julia Roberts" }
    },
    new Movie {
        Title = "The Da Vinci Code",
        Year = 2004,
        Summary = "Die Elf sind jetzt Zwölf.",
        Actors = new List<string> { "George Clooney", "Brad Pitt", "Julia Roberts", "Andy Garcia" }
    }
};

await collection.InsertManyAsync(movies);
```

---

### G) Titel-Update: "Skyfall - 007" zu "Skyfall"

```csharp
var filter = Builders<Movie>.Filter.Eq(m => m.Title, "Skyfall - 007");
var update = Builders<Movie>.Update.Set(m => m.Title, "Skyfall");
await collection.UpdateManyAsync(filter, update);
```

---

### H) Löschen aller Filme mit Jahr ≤ 1995

```csharp
var deleteFilter = Builders<Movie>.Filter.Lte(m => m.Year, 1995);
var deleteResult = await collection.DeleteManyAsync(deleteFilter);
Console.WriteLine($"{deleteResult.DeletedCount} Filme gelöscht.");
```

---

### I) Anzahl Filme pro Jahr ab 2000 (Aggregation)

```csharp
var pipeline = collection.Aggregate()
    .Match(m => m.Year >= 2000)
    .Group(m => m.Year, g => new { Year = g.Key, Count = g.Count() });

await pipeline.ForEachAsync(result =>
    Console.WriteLine($"Jahr: {result.Year}, Anzahl: {result.Count}")
);
```

---

### J) Alle Filme als JSON anzeigen

```csharp
var allMovies = await collection.Find(_ => true).ToListAsync();
foreach (var movie in allMovies)
    Console.WriteLine(movie.ToJson());
```

## Total sollte es schlussendlich so aussehen:  
  
```csharp

using MongoDB.Bson;
using MongoDB.Driver;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace M165App
{
    public class Movie
    {
        public ObjectId Id { get; set; }
        public string Title { get; set; }
        public int Year { get; set; }
        public string Summary { get; set; }
        public List<string> Actors { get; set; }
    }

    class Program
    {
        static async Task Main(string[] args)
        {
            // Verbindung zu MongoDB
            var client = new MongoClient("mongodb://localhost:27017");
            var database = client.GetDatabase("M165");
            var collection = database.GetCollection<Movie>("Movies");

            // Ab hier folgen deine CRUD-Operationen, z. B.:
            var dbs = await client.ListDatabaseNamesAsync();
            await foreach (var name in dbs.ToAsyncEnumerable())
                Console.WriteLine(name);

            // usw.
        }
    }
}

```

> Tipp: Füge `using MongoDB.Bson;` hinzu, um `ToJson()` verwenden zu können.

---

## Weiterführende Links

* [MongoDB C# Driver Quick Start](https://www.mongodb.com/docs/drivers/csharp/current/quick-start/)
* [CRUD Usage Examples](https://www.mongodb.com/docs/drivers/csharp/current/usage-examples/)


