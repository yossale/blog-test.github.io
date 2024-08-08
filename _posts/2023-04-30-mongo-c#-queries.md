---
title: 'Querying Mongo in C#'
description: A few example of how to use MongodoDB drivers in C# and LINQ
date: '2023-04-30 10:00:00'
author: [Yossale]
---

In this blog post, I'll show so example on how to use C# MongoDB drive to querying a collection called "movies", available in the sample data provided by MongoDB when you create a new cluster. 

First, you'll need to install the C# MongoDB driver. 
```
dotnet add package MongoDB.Driver
```

Then, set your environment variable ATLAS_URI param, it should look something like this: 
```sh
ATLAS_URI="mongodb+srv://${MFLIX_DEMO_USER}:${MFLIX_DEMO_PASS}@<YOUR_CLUSTER_URI>.mongodb.net/?retryWrites=true&w=majority"`
```

Our collection conatains movie documents from the last 30 40 years, and each record looks something like this:
```json
{
  "_id": {"573a1398f29313caabce9682"
  },
  "plot": "A young man is accidentally sent 30 years into the past in a time-traveling DeLorean invented by his friend, Dr. Emmett Brown, and must make sure his high-school-age parents unite in order to save his own existence.",
  "genres": [
    "Adventure",
    "Comedy",
    "Sci-Fi"
  ],
  "runtime": 116,
  "metacritic": 86,
  "rated": "PG",
  "cast": [
    "Michael J. Fox",
    "Christopher Lloyd",
    "Lea Thompson",
    "Crispin Glover"
  ],
  "poster": "https://m.media-amazon.com/images/M/MV5BZmU0M2Y1OGUtZjIxNi00ZjBkLTg1MjgtOWIyNThiZWIwYjRiXkEyXkFqcGdeQXVyMTQxNzMzNDI@._V1_SY1000_SX677_AL_.jpg",
  "title": "Back to the Future",
  "fullplot": "Marty McFly, a typical American teenager of the Eighties, is accidentally sent back to 1955 in a plutonium-powered DeLorean \"time machine\" invented by slightly mad scientist. During his often hysterical, always amazing trip back in time, Marty must make certain his teenage parents-to-be meet and fall in love - so he can get back to the future.",
  "languages": [
    "English"
  ],
  "released": {
    "$date": "1985-07-03T00:00:00Z"
  },
  "directors": [
    "Robert Zemeckis"
  ],
  "writers": [
    "Robert Zemeckis",
    "Bob Gale"
  ],
  "awards": {
    "wins": 19,
    "nominations": 24,
    "text": "Won 1 Oscar. Another 18 wins & 24 nominations."
  },
  "lastupdated": "2015-09-12 00:29:36.890000000",
  "year": 1985,
  "imdb": {
    "rating": 8.5,
    "votes": 636511,
    "id": 88763
  },
  "countries": [
    "USA"
  ],
  "type": "movie",
  "num_mflix_comments": 0
}
```

## We're going to run 3 queries:
1. Find the info of the classic "Back to the future" movie
2. Find all the movies Michael Keaton played in between 1985-1990
3. Sum all the runtime of movies played by Ryan Reynolds in our collection


### Plain MongoDB Query Language (MQL)
```mql
use('sample_mflix');

db.movies.findOne({title: "Back to the Future"})

db.movies.find({
    year: {$gte: 1985, $lt: 1990},
    cast: {$in: ["Michael Keaton"]}
})

db.movies.aggregate([
    {$match: {cast: {$in: ["Ryan Reynolds"]}}},
    {$set: {cast: "Ryan Reynolds"}},
    {$group: {_id: "cast", sum: {$sum: "$runtime"}}}
])
```

### C# using a regular filters
`Program.cs`
```c#
using System;
using MongoDB.Driver;
using MongoDB.Bson;

var connectionString = Environment.GetEnvironmentVariable("ATLAS_URI");

var client = new MongoClient(connectionString);

var collection = client.GetDatabase("sample_mflix").GetCollection<BsonDocument>("movies");

// Find movie by title

var filter = Builders<BsonDocument>.Filter.Eq("title", "Back to the Future");

var document = collection.Find(filter).First();

Console.WriteLine(document);

// Michael Keaton movies in the 80's:

filter = Builders<BsonDocument>.Filter.And(
    Builders<BsonDocument>.Filter.Gte("year", 1985),
    Builders<BsonDocument>.Filter.Lte("year", 1990),
    Builders<BsonDocument>.Filter.AnyIn("cast", new BsonArray { "Michael Keaton" })
);

Console.WriteLine("\nMichael Keaton movies in the 80's:");
var cursor = collection.Find(filter).ToCursor();
while (cursor.MoveNext())
{
    foreach (var doc in cursor.Current)
    {
        Console.WriteLine("Title: {0}, Year: {1}", doc["title"], doc["year"]);
    }
}

// Ryan Reynolds Screen time

Console.WriteLine("\n Ryan Reynolds Screen time");

var pipeline = new BsonDocument[]
{
    new BsonDocument("$match", new BsonDocument("cast", "Ryan Reynolds")),    
    new BsonDocument("$group", new BsonDocument("_id", "Ryan Reynolds")
        .Add("totalRuntime", new BsonDocument("$sum", "$runtime")))
};

var result = collection.Aggregate<BsonDocument>(pipeline).FirstOrDefault();

if (result != null) {
    var totalRuntime = result.GetValue("totalRuntime", new BsonInt32(0));
    Console.WriteLine("Total runtime of movies with Ryan Reynolds: {0}", totalRuntime);
}
```

### C# using LINQ

When using Linq, you'll need to define the Movie object we're refering too. Here is a sample of Movie definition file:

`Movie.cs`
```cs
// Movie.cs
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;

[BsonIgnoreExtraElements] // When deserializing, ignore fields that weren't specified here
public class Movie {

    [BsonId] // "This is the ID field"
    [BsonRepresentation(BsonType.ObjectId)] // "It should be represented as Mongo object Id"
    public string Id { get; set; }

    [BsonElement("title")]
    public string Title { get; set; } = null!;

    [BsonElement("year")]
    public int Year { get; set; }

    [BsonElement("runtime")]
    public int Runtime { get; set; }

    [BsonElement("plot")]
    [BsonIgnoreIfNull]
    public string Plot { get; set; } = null!;

    [BsonElement("cast")]
    [BsonIgnoreIfNull] // "When serializing, don't output if it's null. On deserializing, it will be null"
    public List<string> Cast { get; set; } = null!;

}
```

`Program.cs`
```c#
using MongoDB.Driver;
using MongoDB.Driver.Linq;
using System;
using System.Text.Json;

MongoClientSettings settings = MongoClientSettings.FromConnectionString(
    Environment.GetEnvironmentVariable("ATLAS_URI")
);

settings.LinqProvider = LinqProvider.V3;

MongoClient client = new MongoClient(settings);

IMongoCollection<Movie> moviesCollection = client.GetDatabase("sample_mflix").GetCollection<Movie>("movies");

Console.WriteLine("Back to the future!");
IMongoQueryable<Movie> backToFuture =
            from movie in moviesCollection.AsQueryable()
            where movie.Title == "Back to the Future"
            select movie;

var entry = backToFuture.FirstOrDefault();
string strJson = JsonSerializer.Serialize<Object>(entry);
Console.WriteLine(strJson);

Console.WriteLine("\nMichael Keaton movies in the 80's:");

IMongoQueryable<Movie> michaelKeatonMovies =
            from movie in moviesCollection.AsQueryable()
            where movie.Year >= 1985 && movie.Year < 1990
            where movie.Cast.Contains("Michael Keaton")
            select movie;

foreach(Movie film in michaelKeatonMovies) {
    Console.WriteLine("{0}: {1}", film.Title, film.Year);
}


var ryanReynTimeOnScreen = 
            from movie in moviesCollection.AsQueryable()
            where movie.Cast.Contains("Ryan Reynolds") from cast in movie.Cast            
            where cast == "Ryan Reynolds"
            group movie by cast into g
            select new { Cast = g.Key, Sum = g.Sum(x => x.Runtime) };

foreach(var result in ryanReynTimeOnScreen) {
    Console.WriteLine("\n{0} appeared on screen for {1} minutes!", result.Cast, result.Sum);
}
```

## Additional Resources

Sparked you'r interest? Want to read more? Here are a few places to continue your MongoDB C# journey:

[C# On Mac - Quick Start](https://dotnet.microsoft.com/en-us/learn/dotnet/hello-world-tutorial/install)

[Mongo Driver - Quick Start C#](https://www.mongodb.com/docs/drivers/csharp/current/quick-start/)

[LINQ](https://www.mongodb.com/docs/drivers/csharp/current/fundamentals/linq/)

[How to Create a .NET Core App with MongoDB Atlas
](https://www.youtube.com/watch?v=iklJHXn8D1s&t=2s&ab_channel=MongoDB)

