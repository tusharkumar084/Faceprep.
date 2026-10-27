# MongoDB — 5000-Document Generator Notes

## 1. Purpose of the Script

This `mongosh` JavaScript program generates **5,000 randomized e-commerce documents** and stores them in the MongoDB collection:

```javascript
db.aggex
```

The dataset contains product, seller, customer, payment, order, stock, rating, tag, and specification information. It is useful for practising:

- MongoDB queries
- Filtering and sorting
- Update and delete operations
- Aggregation pipelines
- Grouping and calculations
- Working with arrays and embedded objects
- Indexing and performance testing

## 2. Master Data Arrays

The first part defines arrays containing the values from which random data will be selected.

### Categories

```javascript
const categories = [
  "Electronics", "Mobiles", "Laptops", "Home Appliances",
  "Fashion", "Books", "Sports", "Beauty", "Grocery", "Furniture"
];
```

There are 10 possible product categories.

### Subcategories

```javascript
const subCategories = {
  Electronics: ["Headphones", "Smartwatch", "Camera", "Speaker"],
  Mobiles: ["Android", "iPhone", "Feature Phone"]
};
```

`subCategories` is an object in which:

- The key is a category.
- The value is an array of valid subcategories.

This ensures that an Electronics product gets a relevant subcategory such as Camera or Speaker.

### Other value lists

The script also defines possible values for:

| Array | Example values |
|---|---|
| `brands` | Samsung, Apple, Sony, Dell |
| `cities` | Chennai, Bangalore, Mumbai |
| `states` | Tamil Nadu, Karnataka, Maharashtra |
| `paymentMethods` | UPI, Credit Card, Cash on Delivery |
| `orderStatuses` | Delivered, Shipped, Cancelled |
| `sellers` | RetailHub, TechWorld, SmartShop |
| `tags` | new, popular, premium, discount |

## 3. Temporary Batch Array

```javascript
let documents = [];
```

This array temporarily stores generated documents before inserting them into MongoDB.

The program does not immediately run `insertOne()` 5,000 times. Instead, it collects documents and inserts them in groups of 500 for better efficiency.

## 4. Main Generation Loop

```javascript
for (let i = 1; i <= 5000; i++) {
  // Generate one document
}
```

The loop runs exactly 5,000 times. Each iteration creates one product document.

The variable `i` is also used to create a unique product ID.

## 5. Selecting Random Values

The general formula for selecting a random array element is:

```javascript
array[Math.floor(Math.random() * array.length)]
```

How it works:

1. `Math.random()` produces a decimal from 0 up to, but not including, 1.
2. Multiplication by `array.length` produces a value within the array's range.
3. `Math.floor()` converts it into a valid integer index.
4. `array[index]` returns the selected value.

Example:

```javascript
const category =
  categories[Math.floor(Math.random() * categories.length)];
```

## 6. Selecting a Matching Subcategory

```javascript
const subCategory =
  subCategories[category][
    Math.floor(Math.random() * subCategories[category].length)
  ];
```

The selected `category` is used as the key. A random value is then selected from that category's subcategory array.

Example:

```text
category:    Laptops
subCategory: Gaming
```

## 7. Generating Random Numeric Data

### Quantity from 1 to 5

```javascript
const quantity = Math.floor(Math.random() * 5) + 1;
```

### Price from 500 to 95,499

```javascript
const price = Math.floor(Math.random() * 95000) + 500;
```

### Discount from 0% to 50%

```javascript
const discount = Math.floor(Math.random() * 51);
```

### Discounted price

```javascript
const discountedPrice = Math.round(
  price - (price * discount / 100)
);
```

Formula:

```text
discounted price = price − (price × discount ÷ 100)
```

### Rating from 1.0 to 5.0

```javascript
const rating = Number((Math.random() * 4 + 1).toFixed(1));
```

- `Math.random() * 4 + 1` generates a value from 1 up to 5.
- `toFixed(1)` keeps one decimal place but returns a string.
- `Number()` converts the result back into a number.

### Stock from 0 to 499

```javascript
const stock = Math.floor(Math.random() * 500);
```

## 8. Generating Tags

```javascript
const selectedTags = [];

for (let j = 0; j < 2; j++) {
  selectedTags.push(
    tags[Math.floor(Math.random() * tags.length)]
  );
}
```

This loop selects two tags and adds them to the `selectedTags` array.

> The same tag can be selected twice because the code does not check for duplicates.

To guarantee two unique tags:

```javascript
const selectedTags = [];

while (selectedTags.length < 2) {
  const tag = tags[Math.floor(Math.random() * tags.length)];

  if (!selectedTags.includes(tag)) {
    selectedTags.push(tag);
  }
}
```

## 9. Generating a Random Date

```javascript
const startDate = new Date("2024-01-01").getTime();
const endDate = new Date("2026-08-25").getTime();

const randomDate = new Date(
  startDate + Math.random() * (endDate - startDate)
);
```

`getTime()` converts a date into milliseconds. The program selects a random millisecond value between the start and end dates and converts it back into a Date object.

Therefore, every `orderDate` falls between 1 January 2024 and 25 August 2026.

## 10. Product Document Structure

Each generated document has approximately the following structure:

```javascript
{
  productId: "PROD00001",
  productName: "Samsung Android 1",
  category: "Mobiles",
  subCategory: "Android",
  brand: "Samsung",
  price: 25000,
  discountPercentage: 10,
  discountedPrice: 22500,
  quantity: 2,
  revenue: 45000,
  rating: 4.2,
  reviewCount: 2500,
  stock: 100,
  inStock: true,
  seller: {
    name: "TechWorld",
    sellerRating: 4.5
  },
  customer: {
    customerId: "CUS0025",
    city: "Chennai",
    state: "Tamil Nadu",
    age: 24
  },
  payment: {
    method: "UPI",
    transactionId: "TXNabc123xyz"
  },
  orderStatus: "Delivered",
  orderDate: ISODate("2025-04-10T00:00:00.000Z"),
  tags: ["popular", "discount"],
  specifications: {
    warranty: "2 years",
    color: "Black",
    weight: 2.35
  },
  isFeatured: false,
  createdAt: ISODate("...")
}
```

## 11. Important Generated Fields

### Product ID

```javascript
productId: "PROD" + String(i).padStart(5, "0")
```

`padStart(5, "0")` makes the numeric part five digits long.

Examples:

```text
PROD00001
PROD00025
PROD05000
```

### Product name

```javascript
productName: brand + " " + subCategory + " " + i
```

It combines the brand, subcategory, and loop number.

### Revenue

```javascript
revenue: discountedPrice * quantity
```

### Stock status

```javascript
inStock: stock > 0
```

- Stock greater than 0 produces `true`.
- Stock equal to 0 produces `false`.

### Customer ID

```javascript
"CUS" + String(
  Math.floor(Math.random() * 1000) + 1
).padStart(4, "0")
```

This generates customer IDs from `CUS0001` to `CUS1000`. Multiple orders can have the same customer ID.

### Transaction ID

```javascript
"TXN" + Math.random().toString(36).substring(2, 12)
```

The random value is converted to a base-36 string containing letters and numbers.

### Featured status

```javascript
isFeatured: Math.random() > 0.7
```

This makes approximately 30% of documents featured.

## 12. Embedded Objects and Arrays

The document contains embedded objects:

- `seller`
- `customer`
- `payment`
- `specifications`

It also contains an array:

- `tags`

Access an embedded field using dot notation:

```javascript
db.aggex.find({ "customer.city": "Chennai" })
db.aggex.find({ "seller.sellerRating": { $gte: 4 } })
db.aggex.find({ "payment.method": "UPI" })
```

Search an array value:

```javascript
db.aggex.find({ tags: "popular" })
```

## 13. Batch Insertion

Every generated document is first added to the temporary array:

```javascript
documents.push(product);
```

When the array contains 500 documents:

```javascript
if (documents.length === 500) {
  db.aggex.insertMany(documents);
  documents = [];
}
```

This process:

1. Inserts 500 documents together.
2. Clears the temporary array.
3. Continues generating the next batch.

For 5,000 documents, `insertMany()` runs 10 times.

Batch insertion is generally faster and more efficient than performing 5,000 separate `insertOne()` operations.

## 14. Remaining Documents Check

```javascript
if (documents.length > 0) {
  db.aggex.insertMany(documents);
}
```

This inserts any documents remaining after the loop. With exactly 5,000 documents and batches of 500, no documents remain, but this check makes the program safe if the total changes later.

## 15. Completion Message

```javascript
print("5000 documents inserted successfully into aggex!");
```

This message appears after the generation and insertion code completes.

## 16. Verify the Data

```javascript

db.aggex.countDocuments()

db.aggex.findOne()

db.aggex.find().limit(5).pretty()

db.aggex.find({}, { productId: 1, _id: 0 }).sort({ productId: 1 }).limit(1)
db.aggex.find({}, { productId: 1, _id: 0 }).sort({ productId: -1 }).limit(1)
```

Expected document count:

```text
5000
```

## 17. Important Data-Quality Observation

The script selects `city` and `state` independently:

```javascript
const city = cities[Math.floor(Math.random() * cities.length)];
const state = states[Math.floor(Math.random() * states.length)];
```

This can create incorrect pairs such as:

```javascript
{ city: "Chennai", state: "Gujarat" }
```

A better approach is to store valid location pairs:

```javascript
const locations = [
  { city: "Chennai", state: "Tamil Nadu" },
  { city: "Coimbatore", state: "Tamil Nadu" },
  { city: "Bangalore", state: "Karnataka" },
  { city: "Hyderabad", state: "Telangana" },
  { city: "Mumbai", state: "Maharashtra" },
  { city: "Delhi", state: "Delhi" },
  { city: "Pune", state: "Maharashtra" },
  { city: "Kochi", state: "Kerala" },
  { city: "Kolkata", state: "West Bengal" },
  { city: "Ahmedabad", state: "Gujarat" }
];

const location =
  locations[Math.floor(Math.random() * locations.length)];
```

Then use:

```javascript
city: location.city,
state: location.state
```

## 18. Running the Script Again

Running the complete script again inserts another 5,000 documents. MongoDB does not automatically prevent repeated `productId` values unless a unique index exists.

To start again with an empty collection:

```javascript
db.aggex.deleteMany({})
```

Use that command carefully because it deletes every document in the collection.

To enforce unique product IDs:

```javascript
db.aggex.createIndex(
  { productId: 1 },
  { unique: true }
)
```

## 19. Example Practice Queries

```javascript

db.aggex.find({ category: "Laptops" })

db.aggex.find({ price: { $gt: 50000 } })


db.aggex.find({ rating: { $gte: 4.5 } })


db.aggex.find({
  orderStatus: "Delivered",
  "payment.method": "UPI"
})

db.aggex.find({
  isFeatured: true,
  inStock: true
})

db.aggex.find({ tags: "popular" })

db.aggex.find().sort({ revenue: -1 }).limit(10)
```

## 20. Quick Revision

- The script generates 5,000 randomized e-commerce documents.
- Data is stored in the `aggex` collection.
- Category and subcategory values remain logically connected.
- Products contain embedded objects and an array of tags.
- Calculated fields include `discountedPrice`, `revenue`, and `inStock`.
- Documents are inserted in 10 batches of 500.
- Random dates fall between 2024-01-01 and 2026-08-25.
- Duplicate tags and mismatched city/state combinations are possible in the original code.
- Running the script twice may create duplicate `productId` values unless a unique index is created.