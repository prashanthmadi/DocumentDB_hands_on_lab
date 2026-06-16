# Data Relationships Quick Reference

Patterns for modeling one-to-one, one-to-many, and many-to-many relationships in NoSQL.

---

## One-to-One Relationships

### When You Have
User → User Profile | Employee → Employment Details | Product → Specifications

### Option 1: Embed (Recommended for co-accessed data)

```json
{
  "_id": "user-123",
  "username": "johndoe",
  "email": "john@example.com",
  "profile": {
    "firstName": "John",
    "lastName": "Doe",
    "phone": "+1-555-0100",
    "address": {
      "street": "123 Main St",
      "city": "Seattle",
      "zip": "98101"
    }
  }
}
```

**Pros:** Single query, atomic updates  
**Cons:** None for one-to-one

---

### Option 2: Separate Collections (For sensitive or large data)

```json
// Users collection
{ "_id": "user-123", "username": "johndoe", "profileId": "prof-456" }

// Profiles collection
{ "_id": "prof-456", "userId": "user-123", "ssn": "***-**-1234", "salary": 125000 }
```

**Pros:** Separate access control, smaller documents  
**Cons:** Requires two queries

---

## One-to-Many Relationships

### Scenario: User → Orders

### Option 1: Embed Array (For < 100 items, bounded growth)

```json
{
  "_id": "order-123",
  "customerId": "cust-456",
  "items": [
    { "productId": "prod-789", "name": "Laptop", "price": 1299.99, "qty": 1 },
    { "productId": "prod-101", "name": "Mouse", "price": 29.99, "qty": 2 }
  ],
  "total": 1359.97
}
```

**Use when:** Items always accessed together with parent  
**Avoid when:** Array can grow unbounded

---

### Option 2: Child References Parent (For many items, independent queries)

**Most common approach for one-to-many**

```javascript
// Blog collection
{
  "_id": "blog-123",
  "title": "Tech Insights",
  "stats": { "posts": 247 }
}

// Posts collection (each has reference back)
{
  "_id": "post-456",
  "blogId": "blog-123",
  "title": "Understanding NoSQL",
  "content": "..."
}
{
  "_id": "post-457",
  "blogId": "blog-123",
  "title": "MongoDB Patterns"
}

// Query: Find all posts for a blog
db.posts.find({ blogId: "blog-123" }).sort({ publishDate: -1 })

// Index for performance
db.posts.createIndex({ blogId: 1, publishDate: -1 })
```

**Pros:** Handles unlimited children, independent queries, easy pagination  
**Cons:** Requires separate query

---

### Option 3: Parent References Children Array (Rarely used)

```json
{
  "_id": "project-123",
  "name": "Website Redesign",
  "memberIds": ["user-456", "user-789", "user-101"]
}
```

**Use only for:** Bounded lists (team members, roles, tags)  
**Avoid for:** Lists that grow unbounded

---

## Many-to-Many Relationships

### Scenario: Students ↔ Courses | Users ↔ Groups

### Option 1: Simple Array (For one-direction primary, no metadata)

```json
{
  "_id": "user-123",
  "name": "John Doe",
  "skills": ["JavaScript", "Python", "Docker"]
}

// Query: Find users with Python skill
db.users.find({ skills: "Python" })

// Query: Users with BOTH Python AND Docker
db.users.find({ skills: { $all: ["Python", "Docker"] } })
```

**Pros:** Simple, efficient  
**Cons:** Can't query reverse direction, no relationship metadata

---

### Option 2: Two-Way Denormalization (For bidirectional querying)

```javascript
// Students collection
{
  "_id": "student-123",
  "name": "John Doe",
  "enrolledCourses": [
    {
      "courseId": "course-456",
      "courseName": "Data Science 101",
      "enrolledDate": "2026-01-15",
      "grade": "A"
    }
  ]
}

// Courses collection
{
  "_id": "course-456",
  "name": "Data Science 101",
  "instructor": "Dr. Smith",
  "students": [
    {
      "studentId": "student-123",
      "studentName": "John Doe",
      "enrolledDate": "2026-01-15"
    }
  ]
}

// Query: All courses for student
db.students.findOne({ _id: "student-123" })

// Query: All students in course
db.courses.findOne({ _id: "course-456" })
```

**Pros:** Fast queries in both directions, relationship metadata  
**Cons:** Data duplication, must update both collections

---

### Option 3: Join Collection (For complex relationships with metadata)

**Most flexible for many-to-many**

```javascript
// Users collection
{ "_id": "user-123", "name": "John Doe", "role": "Developer" }

// Projects collection
{ "_id": "project-456", "name": "Website Redesign", "budget": 50000 }

// UserProjects collection (the join)
{
  "_id": "up-789",
  "userId": "user-123",
  "projectId": "project-456",
  "role": "Lead Developer",
  "hoursPerWeek": 20,
  "startDate": "2026-01-15",
  "active": true
}

// Query: All projects for a user
db.userProjects.find({ userId: "user-123", active: true })

// Query: All users on a project
db.userProjects.find({ projectId: "project-456" })

// Query: All lead developers on any project
db.userProjects.find({ role: "Lead Developer", active: true })

// Indexes
db.userProjects.createIndex({ userId: 1, active: 1 })
db.userProjects.createIndex({ projectId: 1, active: 1 })
```

**Pros:** Most flexible, rich metadata, easy relationship management  
**Cons:** Extra collection to maintain

---

## Decision Matrix

| Relationship Type | Best Approach | Example |
|---|---|---|
| One-to-One | Embed | User + Profile |
| One-to-Few (< 100) | Embed array | Order + Items |
| One-to-Many | Child references parent | Blog + Posts |
| Many-to-Many, simple | Array of IDs | User + Skills |
| Many-to-Many, bidirectional | Two-way denormalize | Students ↔ Courses |
| Many-to-Many, complex metadata | Join collection | Users ↔ Projects |

---

## Real-World Examples

### E-Commerce Platform

```javascript
// Products ↔ Categories (many-to-many, simple)
{
  "_id": "product-123",
  "name": "Laptop",
  "categories": ["Electronics", "Computers"]  // Simple array
}

// User → Orders (one-to-many)
{
  "_id": "order-456",
  "userId": "user-789",  // Reference to parent
  "items": [ /* embedded array */ ],
  "total": 1299.99
}

// Product ↔ Reviews (one-to-many, paginated)
// Products collection: summary in product doc
{
  "_id": "product-123",
  "name": "Laptop",
  "reviewStats": { "avgRating": 4.7, "total": 523 },
  "recentReviews": [ /* subset */ ]
}
// Reviews collection: paginated queries
{
  "_id": "review-101",
  "productId": "product-123",
  "rating": 5,
  "comment": "Great!"
}
```

### Social Network

```javascript
// Users ↔ Followers (many-to-many, complex)
// Use join collection
{
  "_id": "follow-001",
  "followerId": "user-123",
  "followingId": "user-456",
  "followDate": "2026-01-15"
}

// Query: Who does user-123 follow?
db.follows.find({ followerId: "user-123" })

// Query: Who follows user-456?
db.follows.find({ followingId: "user-456" })
```

### Document Management

```javascript
// Folders ↔ Documents (one-to-many)
{
  "_id": "folder-123",
  "name": "Contracts",
  "ownerId": "user-456"
}

{
  "_id": "doc-789",
  "folderId": "folder-123",
  "title": "Employment Agreement",
  "content": "...",
  "createdDate": "2026-01-15"
}

// Query: All documents in folder
db.documents.find({ folderId: "folder-123" })
```

---

## Key Takeaways

1. **Default to embedding** for small, bounded relationships
2. **Use child references** for one-to-many relationships
3. **Use arrays** for many-to-many with no metadata
4. **Use join collections** for complex many-to-many relationships
5. **Denormalize strategically** to optimize for common queries
6. **Always index** foreign key fields for performance

For deeper exploration, participants can reference this guide during and after the workshop!
