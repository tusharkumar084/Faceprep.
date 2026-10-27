# MongoDB Class — Thursday Work Log

## 1. Topics Covered

This practical session covered:

- Connecting to MongoDB Atlas using `mongosh`
- Viewing databases and collections
- Switching databases
- Retrieving documents
- Generating 5,000 sample product documents
- Inserting documents in batches
- Understanding query execution with `explain()`
- Creating a single-field index
- Comparing `COLLSCAN` and `IXSCAN`
- Viewing and deleting indexes
- Creating a compound index
- Creating a multikey index on an array field

## 2. Connect to MongoDB Atlas

General connection command:

```bash
mongosh "mongodb+srv://<cluster-address>/" --apiVersion 1 --username <username>
```

After running the command, `mongosh` asks for the database user's password.

> Never write the password directly in class notes, screenshots, source code, or GitHub repositories.

A successful connection displays information similar to:

```text
Using MongoDB: 8.0.x
Using Mongosh:  2.10.x
```

The prompt also identifies the Atlas cluster, primary node, and current database:

```text
Atlas atlas-104b3m-shard-0 [primary] test>
```

## 3. View and Switch Databases

Display all accessible databases:

```javascript
show dbs
```

Switch to a database:

```javascript
use PCEA24CY020
```

Expected confirmation:

```text
switched to db PCEA24CY020
```

Display collections in the current database:

```javascript
show collections
```

Collections observed during the practical included:

```text
aggex
categorySummary
crudTest
electronicsProducts
products
sellers
students
```

## 4. Retrieve Existing Product Documents

Display documents from the `products` collection:

```javascript
db.products.find()
```

Display them in a readable format:

```javascript
db.products.find().pretty()
```

Example product structure:

```javascript
{
  _id: ObjectId("..."),
  productId: 1,
  productName: "Product 1",
  category: "Mobile",
  brand: "Samsung",
  price: 2500,
  stock: 11,
  rating: 3.5,
  inStock: true,
  tags: ["electronics", "new"],
  seller: {
    sellerId: 1001,
    sellerName: "Seller 1"
  },
  createdAt: ISODate("...")
}
```

This document demonstrates:

- Normal fields such as `price` and `brand`
- A Boolean field: `inStock`
- An array field: `tags`
- An embedded object: `seller`
- A date field: `createdAt`

## 5. Generate 5,000 Product Documents

### Source data arrays

```javascript
const categories = [
  "Electronics",
  "Mobiles",
  "Laptops",
  "Clothing",
  "Books",
  "Furniture",
  "Shoes",
  "Accessories",
  "Home Appliances",
  "Sports"
];

const brands = [
  "Apple",
  "Samsung",
  "Dell",
  "HP",
  "Lenovo",
  "Sony",
  "Nike",
  "Adidas",
  "Puma",
  "OnePlus"
];

const cities = [
  "Chennai",
  "Bangalore",
  "Hyderabad",
  "Coimbatore",
  "Salem",
  "Erode",
  "Tiruchengode",
  "Madurai",
  "Trichy",
  "Pondicherry"
];

const tags = [
  "new",
  "popular",
  "discount",
  "premium",
  "bestseller",
  "trending",
  "featured",
  "budget"
];

const paymentMethods = [
  "UPI",
  "Credit Card",
  "Debit Card",
  "Cash",
  "Net Banking"
];
```

### Complete document-generation script

```javascript
let products = [];

for (let i = 1; i <= 5000; i++) {
  const category = categories[i % categories.length];
  const brand = brands[i % brands.length];
  const city = cities[i % cities.length];

  const price = Math.floor(Math.random() * 90000) + 1000;
  const quantity = Math.floor(Math.random() * 10) + 1;
  const rating = Number((Math.random() * 4 + 1).toFixed(1));

  const productTags = [
    tags[i % tags.length],
    tags[(i + 2) % tags.length]
  ];

  const product = {
    productId: i,
    name: brand + " Product " + i,
    category: category,
    brand: brand,
    price: price,
    quantity: quantity,
    rating: rating,

    description:
      "This is a high quality " +
      category +
      " product from " +
      brand +
      " with excellent features and performance.",

    tags: productTags,

    seller: {
      sellerId: 1000 + (i % 100),
      name: "Seller " + (i % 100),
      city: city
    },

    location: {
      type: "Point",
      coordinates: [
        76.95 + (Math.random() * 0.5),
        11.00 + (Math.random() * 0.5)
      ]
    },

    paymentMethods: paymentMethods,
    isActive: i % 5 !== 0,

    createdAt: new Date(
      2024,
      i % 12,
      (i % 28) + 1
    )
  };

  // Add email only to every third document.
  // This can be used to practise sparse indexes.
  if (i % 3 === 0) {
    product.email = "customer" + i + "@example.com";
  }

  // Add discount only to every fourth document.
  // This can be used to practise partial indexes.
  if (i % 4 === 0) {
    product.discount = Math.floor(Math.random() * 40) + 5;
  }

  products.push(product);

  // Insert each batch of 500 documents.
  if (products.length === 500) {
    db.product.insertMany(products);
    products = [];
  }
}

// Insert any documents remaining after the loop.
if (products.length > 0) {
  db.product.insertMany(products);
}

print("5000 documents inserted successfully!");
```

## 6. Important Generation Logic

### Repeating category, brand, and city values

```javascript
categories[i % categories.length]
```

The modulo operator `%` keeps the index within the array size. After reaching the last element, selection starts again from the beginning.

### Random price

```javascript
Math.floor(Math.random() * 90000) + 1000
```

This creates an integer price from `1000` to `90999`.

### Random quantity

```javascript
Math.floor(Math.random() * 10) + 1
```

This produces a quantity from `1` to `10`.

### Random rating

```javascript
Number((Math.random() * 4 + 1).toFixed(1))
```

This produces a rating from `1.0` up to `5.0`, with one digit after the decimal point.

### Two tags

```javascript
const productTags = [
  tags[i % tags.length],
  tags[(i + 2) % tags.length]
];
```

Every product receives two tags.

### GeoJSON location

```javascript
location: {
  type: "Point",
  coordinates: [longitude, latitude]
}
```

MongoDB GeoJSON coordinates use this order:

```text
[longitude, latitude]
```

### Optional fields

- `email` exists only when `i % 3 === 0`.
- `discount` exists only when `i % 4 === 0`.

These optional fields are useful for testing sparse and partial indexes.

### Batch insertion

Documents are inserted in groups of 500:

```javascript
db.product.insertMany(products)
```

For 5,000 documents, this produces 10 batches. Batch insertion is more efficient than calling `insertOne()` 5,000 times.

## 7. Verify the Inserted Data

Count all documents:

```javascript
db.product.countDocuments()
```

Expected result:

```text
5000
```

Display sample documents:

```javascript
db.product.find().limit(5).pretty()
```

## 8. Understand `explain("executionStats")`

Use `explain()` to see how MongoDB executes a query:

```javascript
db.product.find({
  price: { $gt: 50000 }
}).explain("executionStats")
```

Important fields:

| Field | Meaning |
|---|---|
| `winningPlan` | Query plan selected by MongoDB |
| `COLLSCAN` | Entire collection was scanned |
| `IXSCAN` | An index was scanned |
| `nReturned` | Documents returned by the query |
| `totalDocsExamined` | Documents inspected |
| `totalKeysExamined` | Index entries inspected |
| `executionTimeMillis` | Approximate execution time |

## 9. Query Before Creating an Index

```javascript
db.product.find({
  price: { $gt: 50000 }
}).explain("executionStats")
```

Observed results before indexing:

```text
winning stage:        COLLSCAN
nReturned:            2234
totalKeysExamined:    0
totalDocsExamined:    5000
```

Meaning:

- MongoDB checked all 5,000 documents.
- It returned 2,234 matching documents.
- No index was available, so no index keys were examined.

## 10. Create a Single-Field Index

Create an ascending index on `price`:

```javascript
db.product.createIndex({ price: 1 })
```

MongoDB returns the generated index name:

```text
price_1
```

Here, `1` means ascending order.

## 11. Query After Creating the Index

Run the same query again:

```javascript
db.product.find({
  price: { $gt: 50000 }
}).explain("executionStats")
```

Observed results after indexing:

```text
winning input stage:  IXSCAN
indexName:            price_1
nReturned:            2234
totalKeysExamined:    2234
totalDocsExamined:    2234
```

Meaning:

- MongoDB used the `price_1` index.
- Documents examined decreased from 5,000 to 2,234.
- Only matching index keys and documents were examined.

### Comparison

| Measurement | Before index | After index |
|---|---:|---:|
| Main scan | `COLLSCAN` | `IXSCAN` |
| Documents returned | 2,234 | 2,234 |
| Documents examined | 5,000 | 2,234 |
| Index keys examined | 0 | 2,234 |

The result did not change. The method MongoDB used to find it became more efficient.

> On a small dataset, execution time can vary and may not immediately decrease. The scan type and number of examined documents are more useful for understanding whether the index helped.

## 12. View Indexes

```javascript
db.product.getIndexes()
```

Observed indexes:

```javascript
[
  { key: { _id: 1 }, name: "_id_" },
  { key: { price: 1 }, name: "price_1" }
]
```

MongoDB automatically creates the unique `_id_` index for every normal collection.

## 13. Delete an Index

Delete the price index by name:

```javascript
db.product.dropIndex("price_1")
```

This removes only the index. It does not delete any product documents.

Check the result:

```javascript
db.product.getIndexes()
```

## 14. Compound Index

A compound index contains more than one field:

```javascript
db.product.createIndex({
  category: 1,
  price: -1
})
```

Generated index name:

```text
category_1_price_-1
```

This index is useful for queries that filter by category and sort or filter by price:

```javascript
db.product.find({
  category: "Laptops"
}).sort({ price: -1 })
```

In the index:

- `category: 1` means ascending category order.
- `price: -1` means descending price order.

Field order matters. This index primarily supports queries beginning with `category`.

View the index:

```javascript
db.product.getIndexes()
```

## 15. Multikey Index on an Array

Create an index on the `tags` array:

```javascript
db.product.createIndex({ tags: 1 })
```

MongoDB automatically treats an index on an array field as a **multikey index**.

Search for products containing the `bestseller` tag:

```javascript
db.product.find({
  tags: "bestseller"
}).explain("executionStats")
```

Observed results:

```text
stage:                IXSCAN
indexName:            tags_1
isMultiKey:           true
nReturned:            1250
totalKeysExamined:    1250
totalDocsExamined:    1250
```

`isMultiKey: true` confirms that MongoDB created a multikey index for the array.

## 16. Sparse Index Practice

The generated `email` field exists only in every third document. A sparse index includes only documents containing the indexed field.

```javascript
db.product.createIndex(
  { email: 1 },
  { sparse: true }
)
```

Useful query:

```javascript
db.product.find({
  email: "customer3@example.com"
})
```

Check the index:

```javascript
db.product.getIndexes()
```

## 17. Partial Index Practice

The `discount` field exists only in every fourth document. A partial index stores only documents matching a specified condition.

```javascript
db.product.createIndex(
  { discount: 1 },
  {
    partialFilterExpression: {
      discount: { $exists: true }
    },
    name: "discount_partial_index"
  }
)
```

Useful query:

```javascript
db.product.find({
  discount: { $gte: 20 }
})
```

## 18. Geospatial Index Practice

The `location` field follows the GeoJSON Point structure:

```javascript
{
  type: "Point",
  coordinates: [longitude, latitude]
}
```

Create a `2dsphere` index:

```javascript
db.product.createIndex({
  location: "2dsphere"
})
```

This index supports geographical queries such as `$near` and `$geoWithin`.

Example:

```javascript
db.product.find({
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [77.0, 11.1]
      },
      $maxDistance: 10000
    }
  }
})
```

`$maxDistance` is measured in metres for GeoJSON queries.

## 19. Important Commands Summary

```javascript
// Display databases
show dbs

// Switch database
use PCEA24CY020

// Display collections
show collections

// Count documents
db.product.countDocuments()

// Examine a query plan
db.product.find({ price: { $gt: 50000 } })
  .explain("executionStats")

// Create single-field index
db.product.createIndex({ price: 1 })

// Create compound index
db.product.createIndex({ category: 1, price: -1 })

// Create multikey index
db.product.createIndex({ tags: 1 })

// Display indexes
db.product.getIndexes()

// Delete one index
db.product.dropIndex("price_1")
```

## 20. Key Learning Outcomes

- `COLLSCAN` means MongoDB scanned the entire collection.
- `IXSCAN` means MongoDB used an index.
- A single-field index contains one field.
- A compound index contains multiple fields, and field order matters.
- An array field automatically produces a multikey index.
- `getIndexes()` lists all collection indexes.
- `dropIndex()` removes an index without deleting documents.
- Sparse indexes contain only documents with the indexed field.
- Partial indexes contain only documents satisfying a filter.
- `2dsphere` indexes support GeoJSON location queries.
- Indexes improve reads but require extra storage and add write overhead.

## 21. Exam-Ready Definitions

### Index

An index in MongoDB is a data structure that stores selected field values in an ordered form, allowing MongoDB to retrieve documents without scanning the entire collection.

### Collection scan

A collection scan (`COLLSCAN`) occurs when MongoDB examines collection documents one by one to find matching records.

### Index scan

An index scan (`IXSCAN`) occurs when MongoDB searches an index to locate matching documents more efficiently.

### Compound index

A compound index is an index created on two or more fields. The order of fields determines which query patterns it can support efficiently.

### Multikey index

A multikey index is an index MongoDB automatically creates when the indexed field contains an array.

### Sparse index

A sparse index includes only documents that contain the indexed field.

### Partial index

A partial index includes only documents that satisfy a specified filter condition.