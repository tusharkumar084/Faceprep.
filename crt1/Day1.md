# JSON

JavaScript Object Notation (JSON) is a structured format used to represent data using **key-value pairs**, similar to a HashMap/Dictionary.

```json
{
  "Name": "Devang Saini",
  "Age": 20
}
```

JSON supports different types of data such as **strings, numbers, booleans, arrays, objects, and null**.

Unlike traditional relational databases such as MySQL, MongoDB uses a **flexible schema**, so documents in the same collection do not necessarily need to contain exactly the same fields.

## 1. Array Object

An array can store multiple values inside a document.

```json
{
  "Name": "Devang Saini",
  "Skills": ["Python", "MySQL"]
}
```

## 2. Embedded Object

An embedded object is an object defined inside another object.

```json
{
  "Name": "Devang Saini",
  "Address": {
    "city": "Jaipur",
    "pincode": 302021,
    "address": "Example Address"
  }
}
```

> **NOTE:** MongoDB internally stores documents in **BSON (Binary JSON)**. BSON extends JSON by supporting additional data types such as `ObjectId`, dates, and binary data. MongoDB automatically generates a unique `_id` for a document if one is not provided.

---

# How to Run MongoDB Queries?

MongoDB commands can be executed using **mongosh**.

## 1. Create / Switch Database

```javascript
use("Poornima")
```

This switches to the `Poornima` database. The database is created when data is first stored in it.

## 2. Create Collection

A collection is similar to a **table** in MySQL.

```javascript
db.createCollection("Student")
```

## 3. Insert One Document

A document is roughly similar to a **row** in MySQL.

```javascript
db.Student.insertOne({
  name: "Devang Saini",
  age: 20
})
```

MongoDB automatically generates an `_id` for the document if one is not provided.

---

# Example

## Insert One Product

```javascript
use("E-Commerce")

db.createCollection("Products")

db.Products.insertOne({
  Name: "Iphone 10 S",
  ProductionYr: 2024,
  ProcessorDetails: {
    Processor: "Gen6",
    CPU: 4,
    Threads: 16
  },
  CameraSpec: {
    Camera: "48px",
    Sensor: "IR Grade",
    MotionCapture: true
  },
  Available: false
})
```

## Insert Multiple Products

```javascript
db.Products.insertMany([
  {
    Name: "Iphone 10 S",
    ProductionYr: 2024,
    ProcessorDetails: {
      Processor: "Gen6",
      CPU: 4,
      Threads: 16
    },
    CameraSpec: {
      Camera: "48px",
      Sensor: "IR Grade",
      MotionCapture: true
    },
    Available: false
  },
  {
    Name: "Iphone 11",
    ProductionYr: 2024,
    ProcessorDetails: {
      Processor: "Gen7",
      CPU: 6,
      Threads: 12
    },
    CameraSpec: {
      Camera: "50px",
      Sensor: "CMOS",
      MotionCapture: true
    },
    Available: true
  },
  {
    Name: "Iphone 12",
    ProductionYr: 2023,
    ProcessorDetails: {
      Processor: "Gen6",
      CPU: 4,
      Threads: 8
    },
    CameraSpec: {
      Camera: "48px",
      Sensor: "IR Grade",
      MotionCapture: false
    },
    Available: true
  },
  {
    Name: "Iphone 13",
    ProductionYr: 2023,
    ProcessorDetails: {
      Processor: "Gen8",
      CPU: 8,
      Threads: 16
    },
    CameraSpec: {
      Camera: "64px",
      Sensor: "CMOS",
      MotionCapture: true
    },
    Available: true
  },
  {
    Name: "Iphone 14",
    ProductionYr: 2022,
    ProcessorDetails: {
      Processor: "Gen8",
      CPU: 6,
      Threads: 12
    },
    CameraSpec: {
      Camera: "48px",
      Sensor: "IR Grade",
      MotionCapture: true
    },
    Available: false
  },
  {
    Name: "Iphone 15",
    ProductionYr: 2022,
    ProcessorDetails: {
      Processor: "Gen9",
      CPU: 8,
      Threads: 16
    },
    CameraSpec: {
      Camera: "50px",
      Sensor: "CMOS",
      MotionCapture: true
    },
    Available: true
  },
  {
    Name: "Iphone 16",
    ProductionYr: 2021,
    ProcessorDetails: {
      Processor: "Gen7",
      CPU: 4,
      Threads: 8
    },
    CameraSpec: {
      Camera: "48px",
      Sensor: "IR Grade",
      MotionCapture: false
    },
    Available: true
  },
  {
    Name: "Iphone 17",
    ProductionYr: 2021,
    ProcessorDetails: {
      Processor: "Gen9",
      CPU: 6,
      Threads: 12
    },
    CameraSpec: {
      Camera: "64px",
      Sensor: "CMOS",
      MotionCapture: true
    },
    Available: false
  },
  {
    Name: "Iphone 18",
    ProductionYr: 2020,
    ProcessorDetails: {
      Processor: "Gen8",
      CPU: 4,
      Threads: 8
    },
    CameraSpec: {
      Camera: "48px",
      Sensor: "IR Grade",
      MotionCapture: true
    },
    Available: true
  },
  {
    Name: "Iphone 19",
    ProductionYr: 2020,
    ProcessorDetails: {
      Processor: "Gen10",
      CPU: 8,
      Threads: 16
    },
    CameraSpec: {
      Camera: "50px",
      Sensor: "CMOS",
      MotionCapture: true
    },
    Available: true
  }
])
```

## Example Output

```javascript
{
  acknowledged: true,
  insertedIds: {
    "0": ObjectId("..."),
    "1": ObjectId("..."),
    "2": ObjectId("..."),
    "3": ObjectId("..."),
    "4": ObjectId("..."),
    "5": ObjectId("..."),
    "6": ObjectId("..."),
    "7": ObjectId("..."),
    "8": ObjectId("..."),
    "9": ObjectId("...")
  }
}
```

MongoDB generates a different `ObjectId` for each inserted document.

---

# MySQL vs MongoDB

| MySQL | MongoDB |
|---|---|
| Database | Database |
| Table | Collection |
| Row | Document |
| Column | Field |
| SQL | MongoDB Query Language |
| Structured Schema | Flexible Schema |

---

# Quick Comparison

## MySQL

```sql
INSERT INTO Student (name, age)
VALUES ('Devang Saini', 20);
```

## MongoDB

```javascript
db.Student.insertOne({
  name: "Devang Saini",
  age: 20
})
```