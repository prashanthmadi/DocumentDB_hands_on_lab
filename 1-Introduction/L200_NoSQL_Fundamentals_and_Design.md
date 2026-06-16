# NoSQL Fundamentals and Design Patterns

A quick-reference guide to core NoSQL concepts and common design patterns.

---

## Why NoSQL?

### The Traditional Database World

**Relational Databases (SQL):**
```
Orders Table          Customers Table
┌─────┬──────┬───┐    ┌─────┬────────┬───────┐
│ ID  │CustID│Amt│    │ ID  │  Name  │ Email │
├─────┼──────┼───┤    ├─────┼────────┼───────┤
│ 101 │  1   │500│    │  1  │  John  │ j@... │
│ 102 │  1   │300│    │  2  │  Jane  │ ja... │
│ 103 │  2   │750│    │  3  │  Bob   │ b@... │
└─────┴──────┴───┘    └─────┴────────┴───────┘
```

**Challenges:**
- ❌ **Rigid Schema** - Must define structure upfront
- ❌ **JOINs Required** - Fetch data from multiple tables
- ❌ **Vertical Scaling** - Expensive hardware upgrades
- ❌ **Fixed Schema Changes** - Migrations break applications
- ❌ **Not Cloud-Native** - Difficult to distribute globally

### Enter NoSQL (DocumentDB)

**Document Model:**
```json
{
  "_id": "101",
  "customer": {
    "name": "John Doe",
    "email": "john@example.com",
    "memberId": "GOLD-1234"
  },
  "items": [
    { "product": "Laptop", "price": 1299, "qty": 1 },
    { "product": "Mouse", "price": 25, "qty": 2 }
  ],
  "total": 1349,
  "status": "shipped",
  "orderDate": "2026-02-01"
}
```

**Advantages:**
- ✅ **Flexible Schema** - Add fields as needed
- ✅ **No JOINs** - Data stored together (embedded)
- ✅ **Horizontal Scaling** - Add more servers easily
- ✅ **Cloud-Native** - Distributed by design
- ✅ **Developer-Friendly** - JSON matches application objects

### When to Use NoSQL?

| Use NoSQL When... | Use SQL When... |
|---|---|
| Rapid prototyping & agile development | Strict transactional requirements |
| Hierarchical/nested data structures | Complex reporting across many tables |
| High read/write throughput needed | ACID guarantees are critical |
| Schema evolves frequently | Schema is stable and well-defined |
| Global distribution required | Single-region deployment |
| Working with semi-structured data | Highly structured, tabular data |

**Real-World NoSQL Use Cases:**
- 🛒 **E-commerce:** Product catalogs with varying attributes
- 👤 **User Profiles:** Social media, gaming profiles
- 📱 **Mobile Apps:** Offline-first applications
- 📊 **IoT:** Sensor data with diverse structures
- 🤖 **AI/ML:** Embeddings, vectors, unstructured data

---

## The Core Decision: Embed vs Reference

Before applying any pattern, every NoSQL schema design starts with one question: **should I embed or reference?**

**Embedding (Denormalization)** — Store related data together
```json
{
  "_id": "order-123",
  "customer": { "name": "John", "email": "john@example.com" },
  "items": [{ "product": "Laptop", "price": 1299 }]
}
```
✅ Use for: One-to-few relationships, data accessed together  
❌ Avoid for: Unbounded growth (e.g., user's 10,000 orders)

**Referencing (Normalization)** — Store links to other documents
```json
{
  "_id": "order-123",
  "customerId": "cust-456",
  "itemIds": ["item-789"]
}
```
✅ Use for: One-to-many, many-to-many, independent access  
❌ Avoid for: Frequently accessed together data

| Factor | Embed When... | Reference When... |
|---|---|---|
| **Relationship** | One-to-few | One-to-many / Many-to-many |
| **Data Size** | Small, bounded | Large, unbounded |
| **Update Frequency** | Related data updated together | Updated independently |
| **Read Pattern** | Always read together | Sometimes read separately |
| **Data Growth** | Size is predictable | Size grows unbounded |
| **Duplication** | Acceptable | Must avoid |

---

## Key Design Patterns

### Pattern 1: Computed Pattern (Pre-Calculate)

❌ Bad: Calculate totals on every read
```json
{ "items": [{ "price": 100 }, { "price": 50 }] }
```

✅ Good: Store pre-calculated values
```json
{
  "items": [{ "price": 100 }, { "price": 50 }],
  "subtotal": 150,
  "tax": 12,
  "total": 162
}
```

**When to use:** Sums, averages, counts, complex calculations

---

### Pattern 2: Subset Pattern (Keep Recent, Archive Bulk)

❌ Bad: Store all 10,000 reviews in product document
```json
{ "name": "Laptop", "reviews": [ /* 10,000 items */ ] }
```

✅ Good: Store recent subset, keep archive separate
```json
{
  "name": "Laptop",
  "reviewStats": { "avgRating": 4.7, "total": 10000 },
  "recentReviews": [ /* 5 items */ ]
}
// Full reviews in separate collection
```

**When to use:** Large arrays, historical data, paginated content

---

### Pattern 3: Extended Reference (Duplicate Display Fields)

❌ Bad: Need separate lookup for author details
```json
{ "title": "Post Title", "authorId": "user-456" }
```

✅ Good: Duplicate frequently-accessed fields
```json
{
  "title": "Post Title",
  "author": {
    "id": "user-456",
    "name": "John Doe",
    "avatar": "url/img.jpg"
  }
}
```

**Trade-off:** Duplication, but avoids expensive lookups

---

### Pattern 4: Bucket Pattern (Group Time-Series Data)

❌ Bad: One doc per sensor reading = 1M documents
```json
{ "sensorId": "temp-01", "value": 22.5, "time": "10:00" }
{ "sensorId": "temp-01", "value": 22.7, "time": "10:01" }
// 1 million rows!
```

✅ Good: Bucket readings by hour
```json
{
  "_id": "temp-01-2026-02-07-10",
  "readings": [
    { "minute": 0, "value": 22.5 },
    { "minute": 1, "value": 22.7 }
  ],
  "stats": { "min": 22.3, "max": 23.1, "avg": 22.7 }
}
// 60x fewer documents!
```

**When to use:** IoT, logs, metrics, time-series data

---

### Pattern 5: Outlier Pattern (Handle Extreme Cases Separately)

**Problem:** Some documents have extreme values that don't fit the normal pattern

**Solution:** Handle outliers separately

**Example: Social Media Posts**

```json
// Regular post (< 100 comments)
{
  "_id": "post-123",
  "content": "Check out my photo",
  "comments": [
    { "user": "Alice", "text": "Nice!" },
    { "user": "Bob", "text": "Cool!" }
    // Embed up to 100 comments
  ],
  "hasOverflowComments": false
}

// Viral post (10,000+ comments)
{
  "_id": "post-viral",
  "content": "This went viral!",
  "commentsPreview": [
    { "user": "Alice", "text": "Amazing!" }
    // First 10 comments
  ],
  "commentCount": 10523,
  "hasOverflowComments": true,
  "commentsBucketId": "comments-post-viral"
}

// Overflow comments in separate collection
{
  "_id": "comments-post-viral-bucket-1",
  "postId": "post-viral",
  "comments": [
    // 100 comments per bucket
  ]
}
```

**When to use:**
- 95% of documents fit normal pattern
- 5% are outliers (very popular posts, products, etc.)
- Want to optimize for the common case

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Massive Arrays (Unbounded Growth)

❌ **BAD:**
```json
{
  "_id": "user-123",
  "name": "John",
  "orders": [
    // User has 10,000 orders over 10 years
    // Document size exceeds 16MB limit!
  ]
}
```

✅ **FIX:** Use references or subset pattern
```json
{
  "_id": "user-123",
  "name": "John",
  "recentOrders": [
    // Last 5 orders only
  ]
}

// Orders collection
{
  "_id": "order-456",
  "userId": "user-123",
  "items": [...]
}
```

---

### Anti-Pattern 2: One Collection Per Customer/Tenant

❌ **BAD:**
```javascript
// customer1_orders, customer2_orders, customer3_orders...
// Nightmare to maintain!
```

✅ **FIX:** Use single collection with tenant field
```json
{
  "_id": "order-123",
  "tenantId": "customer1",
  "items": [...]
}

// Index on tenantId for fast filtering
db.orders.createIndex({ tenantId: 1 })
```

---

### Anti-Pattern 3: Unnecessary Indexes

❌ **BAD:**
```javascript
// Index on every field "just in case"
db.products.createIndex({ name: 1 })
db.products.createIndex({ category: 1 })
db.products.createIndex({ brand: 1 })
db.products.createIndex({ price: 1 })
db.products.createIndex({ rating: 1 })
// Slows down writes, wastes storage
```

✅ **FIX:** Index only queried fields
```javascript
// Based on actual query patterns
db.products.createIndex({ category: 1, inStock: 1 })  // Compound index
db.products.createIndex({ rating: -1 })  // For sorting
```

---

### Anti-Pattern 4: Lack of Schema Validation

❌ **BAD:**
```json
// No consistency
{ "name": "Product 1", "price": 100 }
{ "productName": "Product 2", "cost": "50" }  // Different field names, wrong type!
{ "name": "Product 3" }  // Missing price!
```

✅ **FIX:** Use schema validation
```javascript
db.createCollection("products", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "price", "category"],
      properties: {
        name: { bsonType: "string" },
        price: { bsonType: "double", minimum: 0 },
        category: { bsonType: "string" }
      }
    }
  }
})
```

---

### Anti-Pattern 5: Case-Sensitive Fields

❌ **BAD:**
```json
{ "email": "JOHN@EXAMPLE.COM" }
{ "email": "john@example.com" }
// Same user, different emails due to case!
```

✅ **FIX:** Normalize to lowercase
```javascript
// Normalize before insert
const email = userInput.email.toLowerCase();
db.users.insertOne({ email: email });

// Case-insensitive index
db.users.createIndex(
  { email: 1 },
  { collation: { locale: "en", strength: 2 } }
)
```

---

## Quick Design Decision Tree

```
Is this data always accessed together?
├─ YES → Embed it
└─ NO → Reference it

Will this array grow unbounded?
├─ YES → Use reference or bucket pattern
└─ NO → Embed if accessed together

Do you need relationship metadata?
├─ YES → Use join collection
└─ NO → Use simple array of IDs

Is this data read frequently?
├─ YES → Consider duplication (extended reference)
└─ NO → Normalize to save space
```

---

## Key Takeaways

1. **Embed for speed** — Combine related data when accessed together
2. **Reference for flexibility** — Separate when data is independent
3. **Compute when needed** — Pre-calculate expensive operations
4. **Subset for scale** — Keep recent data, archive the rest
5. **Index strategically** — Only on queried fields
6. **Normalize carefully** — Duplication is often worth it for performance

For deeper exploration of these concepts, participants can review this guide later!
