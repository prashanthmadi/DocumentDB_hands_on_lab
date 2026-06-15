# Sample Data

This folder contains the movie dataset used for the query, search, and performance lab.

## Files

- `movies_with_vectors.json`: 50 movies with title, genre, year, rating, description, and a 256-dimensional embedding field.
- `query_embeddings.json`: pre-generated query vectors for the vector search exercises.

## Why This Dataset Is Used

- It is small enough for a workshop.
- It supports both standard MongoDB queries and search scenarios.
- It includes pre-generated vectors, so participants do not need Azure OpenAI access.

## Notes

- Import this dataset into the `cosmicworks.movies` collection for Module 3.
- Vector search requires an appropriate cluster tier.