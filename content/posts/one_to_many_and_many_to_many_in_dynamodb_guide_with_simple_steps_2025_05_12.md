---
title: "One-to-Many and Many-to-Many in DynamoDB: A Guide with Simple Steps"
date: 2025-05-12
categories:
  - AWS
tags:
  - AWS
thumbnailImagePosition: left
thumbnailImage: images/relationships-in-ddb-cover.png
---

## 1. Introduction

I had to design a DynamoDB schema to support both one-to-many and many-to-many relationships in my company’s invoice center. Coming from an RDBMS background, it took me about a week to really understand how to model these types of relationships in a NoSQL environment. There are several ways to approach it, and in this article, I’ll show you one of the most common patterns for handling relationships in DynamoDB.

![4 OAuth2 Grant Types](/images/relationships-in-ddb-cover.png)

---

## 2. Content

### 2.1 Before You Start Designing

DynamoDB is a NoSQL database. Unlike traditional relational databases where you normalize entities and perform joins to query data, DynamoDB requires you to design based on access patterns first.

Before modeling your tables, here are a few things you should do:

- Fully understand your application
- Create your entity-relationship diagram (ERD)
- List all your access patterns

Once you have these, you can design your tables to support those access patterns effectively. You’ll primarily use **partition keys**, **sort keys**, and optionally **secondary indexes**.

### 2.2 One-to-Many Relationship

DynamoDB allows you to model one-to-many relationships using composite primary keys. A group of related items shares the same partition key, and you differentiate them using the sort key. You can also use a Global Secondary Index (GSI) to support alternative access patterns.

Let’s say each user can have multiple orders, and each order belongs to one specific user. We want to be able to query a user’s profile, retrieve all their orders, or fetch a specific order by its order ID.

**Access Patterns**

| Access pattern | Index | Parameters |
| --- | --- | --- |
| View a user’s profile | Main table | User ID |
| View all orders belonging to a user | Main table | User ID |
| View a specific order by order ID | GSI1 | Order ID |

**Entity Design**

```
Entity| PK           | SK
------|--------------|----------------
User  | USER#<userId>| USER#<userId>
Order | USER#<userId>| ORDER#<orderId>
```

To support reverse lookup (e.g., find an order just by its ID), we create GSI1 (partition key: ‘SK’, sort key: ‘PK’).

**Query Examples**

View profile of a user:

```json
KeyConditionExpression: "PK = :pk AND SK = :sk",
ExpressionAttributeValues: {
  ":pk": "USER#123",
  ":sk": "USER#123"
}
```

View all orders of a user

```json
KeyConditionExpression: "PK = :pk AND begins_with(SK, :skPrefix)",
ExpressionAttributeValues: {
  ":pk": "USER#123",
  ":skPrefix": "ORDER#"
}
```

View an order info

```json
IndexName: "GSI1",
KeyConditionExpression: "SK = :sk AND begins_with(PK, :pkPrefix)",
ExpressionAttributeValues: {
  ":sk": "ORDER#1001",
  ":pkPrefix": "USER#"
}
```

### 2.3 Many-to-Many Relationship

Just like with one-to-many relationships in DynamoDB, we’ll use the same core concepts—composite keys and access pattern–driven design. But with many-to-many relationships, we need to create association items that represent the link between two top-level entities.

Let’s say we have student and lecture entities. A student can attend multiple lectures, and each lecture can be attended by multiple students.

**Access Patterns**

| **Access Pattern** | **Index** | **Parameters** |
| --- | --- | --- |
| View a student’s profile | Main table | Student ID |
| View a lecture’s info | Main table | Lecture ID |
| View all lectures for a student | Main table | Student ID |
| View all students for a lecture | GSI1 | Lecture ID |

**Entity Design**

We store the actual entities (students and lectures), along with attendance records that represent the many-to-many relationships. These records are written from both directions to support flexible querying.

```json
Entity    | PK                 | SK
----------|--------------------|--------------------
Student   | STUDENT#<studentId>| STUDENT#<studentId>
Attendance| STUDENT#<studentId>| LECTURE#<lectureId>
Lecture   | LECTURE#<lectureId>| LECTURE#<lectureId>
```

To support reverse lookup (e.g., all students for a lecture), we create GSI1 (partition key: ‘SK’, sort key: ‘PK’).

**Query Examples**

View all lectures a student is attending

```json
KeyConditionExpression: "PK = :pk AND begins_with(SK, :skPrefix)",
ExpressionAttributeValues: {
  ":pk": "STUDENT#S001",
  ":skPrefix": "LECTURE#"
}
```

View all students attending a lecture (using GSI1)

```json
IndexName: "GSI1",
KeyConditionExpression: "SK = :sk AND begins_with(PK, :pkPrefix)",
ExpressionAttributeValues: {
  ":sk": "LECTURE#L100",
  ":pkPrefix": "STUDENT#"
}
```


---

## 3. Conclusion

In the one-to-many example, we used composite keys to group related items under the same partition key. For many-to-many relationships, we introduced association items and a reverse GSI to enable efficient querying from both sides. It’s important to note that modeling in DynamoDB doesn’t start with designing entities or tables—it starts with understanding how your application needs to query the data.

## References

- [Amazon DynamoDB Document](https://docs.aws.amazon.com/dynamodb/)
- [How to model one-to-many relationships - DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItemCollections.html)
- [Best practices for managing many-to-many relationships - DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-adjacency-graphs.html)
