# JPA

## N+1 selects problem

The **N+1 selects problem** is a common performance issue in database access, especially when using ORMs (Object-Relational Mappers) like Hibernate with JPA. It occurs when retrieving a collection of entities, and for each entity, a separate query is executed to load its related entities, leading to inefficient multiple queries.

### How it happens:
Let's break it down:

1. **N** refers to the number of queries executed to fetch related entities.
2. The **+1** refers to the initial query to fetch the main entities.

For example, suppose you have two entities: `Author` and `Book`, where an `Author` has many `Books`. Now, consider the scenario where you want to fetch a list of authors along with their books.

#### Code Example (Lazy loading, default in JPA):
```java
// Fetch all authors
List<Author> authors = em.createQuery("SELECT a FROM Author a", Author.class).getResultList();

// Access each author's books
for (Author author : authors) {
    System.out.println(author.getBooks());  // This triggers another query for each author
}
```

- **Step 1**: One query (`+1`) fetches all authors:
  ```sql
  SELECT * FROM Author;
  ```

- **Step 2**: For each author (N authors), a separate query fetches their books:
  ```sql
  SELECT * FROM Book WHERE author_id = ?;
  ```

If you have 10 authors, you will execute **1 query to fetch authors** and then **10 additional queries** to fetch books, one for each author, resulting in **11 total queries**. For large datasets, this becomes very inefficient and can cause significant performance degradation.

### Solving the N+1 Problem

To solve the N+1 selects problem, the goal is to reduce the number of queries by using **eager fetching** or **batch fetching** strategies.

1. **Eager Fetching** (`JOIN FETCH` or `@FetchType.EAGER`):
   - This approach fetches the related entities (e.g., books) in the same query as the main entity (e.g., authors) by using a join.
   
   Example:
   ```java
   List<Author> authors = em.createQuery("SELECT a FROM Author a JOIN FETCH a.books", Author.class).getResultList();
   ```
   This executes a **single query** to fetch both authors and their books in one go:
   ```sql
   SELECT a.*, b.* FROM Author a LEFT JOIN Book b ON a.id = b.author_id;
   ```

2. **Use of FetchType.LAZY with a Batch Size**:
   - You can keep lazy loading but optimize the fetching by using Hibernate’s `@BatchSize` annotation to load related entities in batches rather than one by one.
   
   Example:
   ```java
   @OneToMany(fetch = FetchType.LAZY)
   @BatchSize(size = 10)
   private List<Book> books;
   ```
   This allows Hibernate to fetch books for multiple authors in a batch, reducing the number of queries from N to fewer, larger queries.

### Summary:
The N+1 selects problem occurs when for each entity fetched (N entities), additional queries are executed to retrieve related data, leading to excessive database queries and poor performance. The problem can be solved by using techniques like eager fetching (`JOIN FETCH`) or batching.