# 3. Queries, Search, and Performance

**Duration:** 2 hours

This lab gives participants enough MongoDB query practice to become productive, then builds into search and performance troubleshooting.

## Learning Goals

- Run CRUD operations with `mongosh`.
- Use `find()` for filters and projections.
- Use `aggregate()` for summaries.
- Compare keyword search, full text search, and vector search.
- Read a simple query plan and identify indexing problems.

## Dataset

This section uses the movie dataset in [../sample-data](../sample-data).

If the dataset is not already loaded, import it into a database called `cosmicworks`.

```bash
mongoimport \
  --uri "<target-connection-string>" \
  --db cosmicworks \
  --collection movies \
  --file sample-data/movies_with_vectors.json \
  --jsonArray
```

If `mongoimport` is not installed, you can load the same file from `mongosh`:

```javascript
const docs = JSON.parse(fs.readFileSync("sample-data/movies_with_vectors.json", "utf8"))
db.movies.insertMany(docs)
```

## Part A: Warm-Up With Basic Queries

```javascript
use cosmicworks

show collections

db.movies.findOne()

db.movies.find(
  { genre: "Sci-Fi" },
  { _id: 0, title: 1, genre: 1, year: 1 }
).limit(5)
```

### Add a Practice Document

```javascript
db.movies.insertOne({
  title: "Workshop Test Movie",
  genre: "Drama",
  year: 2026,
  rating: 7.5,
  description: "Temporary document for CRUD practice"
})
```

### Update It

```javascript
db.movies.updateOne(
  { title: "Workshop Test Movie" },
  { $set: { rating: 8.1 } }
)
```

### Retrieve and Verify It

> 🔍 **Exercise:** Write a `find()` query to retrieve the document you just updated. Verify that the rating changed to 8.1.

### Remove It

```javascript
db.movies.deleteOne({ title: "Workshop Test Movie" })
```

## Part B: Filtering and Projection

```javascript
db.movies.find(
  {
    genre: "Action",
    year: { $gt: 2000 }
  },
  {
    _id: 0,
    title: 1,
    genre: 1,
    year: 1,
    rating: 1
  }
).sort({ rating: -1 })
```

> 🔍 **Exercise:** Write a query to find all "Drama" movies released in the 2000s (year >= 2000 and year < 2010). Return only title, year, and rating, sorted by rating descending.

## Part C: Aggregation Pipelines

### Count Movies by Genre

```javascript
db.movies.aggregate([
  { $group: { _id: "$genre", count: { $sum: 1 } } },
  { $sort: { count: -1 } }
])
```

### Top Rated Movies

```javascript
db.movies.aggregate([
  { $match: { rating: { $gte: 8.5 } } },
  { $project: { _id: 0, title: 1, rating: 1, year: 1, genre: 1 } },
  { $sort: { rating: -1 } }
])
```

### Average Rating by Decade

```javascript
db.movies.aggregate([
  {
    $bucket: {
      groupBy: "$year",
      boundaries: [1970, 1980, 1990, 2000, 2010, 2020, 2030],
      default: "Other",
      output: {
        count: { $sum: 1 },
        avgRating: { $avg: "$rating" }
      }
    }
  }
])
```

> 🔍 **Exercise:** Modify this aggregation to sort the results by decade (ascending) and limit to only decades with 5 or more movies. (Hint: Add `$match` before `$bucket` and `$sort` after.)

## Part D: Search Options

### 1. Keyword Search

```javascript
db.movies.find(
  { title: /matrix/i },
  { _id: 0, title: 1, genre: 1, year: 1 }
)
```

### 2. Full Text Search

Create a text index:

```javascript
db.runCommand({
  createIndexes: "movies",
  indexes: [{
    name: "movies_text_idx",
    key: {
      title: "text",
      description: "text"
    }
  }]
})
```

Run a text query:

```javascript
db.movies.find(
  { $text: { $search: "dream" } },
  {
    _id: 0,
    title: 1,
    genre: 1,
    score: { $meta: "textScore" }
  }
).sort({ score: { $meta: "textScore" } }).limit(5)
```

This dataset reliably returns titles such as `Inception` for that search. Score ordering can vary.

> 🔍 **Exercise:** Try searching for "time" in the title and description. How many results do you get? Do the results make sense?

### 3. Vector Search

Create a vector index:

```javascript
db.runCommand({
  createIndexes: "movies",
  indexes: [{
    name: "vector_index",
    key: { contentVector: "cosmosSearch" },
    cosmosSearchOptions: {
      kind: "vector-diskann",
      dimensions: 256,
      similarity: "COS"
    }
  }]
})
```

Then run a vector search using one of the pre-generated query embeddings:

```javascript
const queries = JSON.parse(fs.readFileSync("sample-data/query_embeddings.json", "utf8"))
const queryVector = queries[0].embedding

db.movies.aggregate([
  {
    $search: {
      cosmosSearch: {
        vector: queryVector,
        path: "contentVector",
        k: 5
      }
    }
  },
  {
    $project: {
      _id: 0,
      title: 1,
      genre: 1,
      year: 1,
      rating: 1,
      score: { $meta: "searchScore" }
    }
  }
])
```

## Part E: Performance Troubleshooting

Start with a query that works but may not be efficient:

```javascript
db.movies.find({ genre: "Action", year: { $gt: 2000 } }).explain("executionStats")
```

Look for these fields:

- `stage`
- `nReturned`
- `totalDocsExamined`
- `executionTimeMillis`

If you see `COLLSCAN`, the database scanned the full collection.

> 🔍 **Exercise:** Run the explain() query above. Write down the `executionStage` and the ratio of `totalDocsExamined` to `nReturned`. What does this tell you about efficiency?

Create an index:

```javascript
db.movies.createIndex({ genre: 1, year: 1 })
```

Re-run the query and force the index for the lab:

```javascript
db.movies.find({ genre: "Action", year: { $gt: 2000 } })
  .hint({ genre: 1, year: 1 })
  .explain("executionStats")
```

Now compare:

- `COLLSCAN` vs `IXSCAN`
- all documents examined vs only matching documents examined

> 🔍 **Exercise:** Compare the two explain() results. What changed? How many fewer documents had to be examined with the index?

## Suggested Teaching Sequence

For beginners, keep this order:

1. `findOne()` and `find()`
2. Insert, update, delete
3. `aggregate()` with `$match`, `$group`, and `$sort`
4. Keyword search
5. Full text search
6. Vector search
7. `explain()` and indexing

## Success Check

- [ ] You ran a few `find()` queries.
- [ ] You practiced insert, update, and delete.
- [ ] You ran at least two aggregation pipelines.
- [ ] You compared keyword, full text, and vector search.
- [ ] You used `explain("executionStats")`.
- [ ] You created an index and saw why it matters.