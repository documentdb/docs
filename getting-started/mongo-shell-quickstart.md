# Node.js Setup Guide

Learn how to set up and use DocumentDB with Node.js using the official MongoDB Node.js driver.

## Prerequisites

- Node.js 14.x or later
- npm or yarn package manager
- DocumentDB installed and running
- Docker installed (if set up is not completed yet)
- Basic Node.js knowledge

## Project Setup (skip if already done)

Before connecting from Node.js, make sure you have a running DocumentDB instance using Docker:

   ```bash
   # Pull the latest DocumentDB Docker image
   docker pull ghcr.io/documentdb/documentdb/documentdb-local:latest

   # Tag the image for convenience
   docker tag ghcr.io/documentdb/documentdb/documentdb-local:latest documentdb

   # Run the container with your chosen username and password
   docker run -dt -p 10260:10260 --name documentdb-container documentdb --username <YOUR_USERNAME> --password <YOUR_PASSWORD>

   docker image rm -f ghcr.io/documentdb/documentdb/documentdb-local:latest
  ```
> **Note:** Replace `<YOUR_USERNAME>` and `<YOUR_PASSWORD>` with your desired credentials. You must set these when creating the container for authentication to work.
>
> **Port Note:** Port `10260` is used by default in these instructions to avoid conflicts with other local database services. You can use port `27017` (the standard MongoDB port) or any other available port if you prefer. If you do, be sure to update the port number in both your `docker run` command and your connection string accordingly.

## Installation

1. Creating a new Node.js project
   ```bash
   mkdir my-documentdb-app
   cd my-documentdb-app
   npm init -y
   ```

2. Installing the MongoDB driver
   ```bash
   npm install mongodb
   ```

## Connecting to DocumentDB

```javascript
const { MongoClient } = require('mongodb');

const uri = 'mongodb://localhost:27017';
const client = new MongoClient(uri);

async function connect() {
  try {
    await client.connect();
    const db = client.db('your_database');
    return db;
  } catch (error) {
    console.error('Connection error:', error);
    throw error;
  }
}
```

## Basic Operations

1. Creating collections
   ```javascript
   const collection = db.collection('your_collection');
   ```

2. Document operations

   **Insert operations**
   ```javascript
   // Insert a single document
   const result = await collection.insertOne({ 
     name: "John Doe", 
     email: "john@example.com", 
     created_at: new Date() 
   });
   console.log('Inserted document ID:', result.insertedId);

   // Insert multiple documents
   const documents = [
     { name: "Jane Smith", email: "jane@example.com" },
     { name: "Bob Johnson", email: "bob@example.com" }
   ];
   const result = await collection.insertMany(documents);
   console.log('Inserted document IDs:', result.insertedIds);
   ```

   **Find operations**
   ```javascript
   // Find all documents
   const allUsers = await collection.find({}).toArray();
   console.log('All users:', allUsers);

   // Find with criteria
   const user = await collection.findOne({ name: "John Doe" });
   console.log('Found user:', user);

   // Find with projection
   const users = await collection.find({}, { 
     projection: { name: 1, email: 1, _id: 0 } 
   }).toArray();
   console.log('Users with projection:', users);

   // Complex queries
   const recentUsers = await collection.find({
     $and: [
       { created_at: { $gte: new Date("2025-01-01") } },
       { email: { $regex: "@example.com$" } }
     ]
   }).toArray();
   console.log('Recent users:', recentUsers);
   ```

   **Update operations**
   ```javascript
   // Update a single document
   const result = await collection.updateOne(
     { name: "John Doe" },
     { $set: { status: "active" } }
   );
   console.log('Modified count:', result.modifiedCount);

   // Update multiple documents
   const result = await collection.updateMany(
     { email: { $regex: "@example.com$" } },
     { $set: { domain: "example.com" } }
   );
   console.log('Modified count:', result.modifiedCount);
   ```

   **Delete operations**
   ```javascript
   // Delete a single document
   const result = await collection.deleteOne({ name: "John Doe" });
   console.log('Deleted count:', result.deletedCount);

   // Delete multiple documents
   const result = await collection.deleteMany({ status: "inactive" });
   console.log('Deleted count:', result.deletedCount);
   ```

3. Working with indexes
   ```javascript
   // Create a single field index
   await collection.createIndex({ email: 1 });
   console.log('Created index on email field');

   // Create a compound index
   await collection.createIndex({ name: 1, email: 1 });
   console.log('Created compound index on name and email');

   // Create a unique index
   await collection.createIndex({ email: 1 }, { unique: true });
   console.log('Created unique index on email');

   // Create a text index
   await collection.createIndex({ content: "text" });
   console.log('Created text index on content field');

   // Create a geospatial index
   await collection.createIndex({ location: "2dsphere" });
   console.log('Created geospatial index on location field');

   // Create a vector index
   await collection.createIndex({ 
     embedding: "vector" 
   }, { 
     vectorOptions: { dimensions: 384 } 
   });
   console.log('Created vector index on embedding field');

   // List all indexes
   const indexes = await collection.indexes();
   console.log('Collection indexes:', indexes);
   ```

4. Advanced operations
   ```javascript
   // Aggregation pipeline
   const pipeline = [
     { $match: { status: "completed" } },
     { $group: {
         _id: "$customer",
         total: { $sum: "$amount" },
         count: { $sum: 1 }
     }},
     { $sort: { total: -1 } }
   ];
   const results = await collection.aggregate(pipeline).toArray();
   console.log('Aggregation results:', results);

   // Vector search operations
   const vectorQuery = {
     $vectorSearch: {
       queryVector: [0.1, 0.2, 0.3],
       path: "embedding",
       numCandidates: 100,
       limit: 10
     }
   };
   const vectorResults = await collection.find(vectorQuery).toArray();
   console.log('Vector search results:', vectorResults);

   // Geospatial queries
   const geoQuery = {
     location: {
       $near: {
         $geometry: {
           type: "Point",
           coordinates: [-73.9667, 40.78]
         },
         $maxDistance: 1000
       }
     }
   };
   const geoResults = await collection.find(geoQuery).toArray();
   console.log('Geospatial results:', geoResults);

   // Text search
   const textResults = await collection.find({
     $text: { $search: "search term" }
   }).toArray();
   console.log('Text search results:', textResults);
   ```

5. Complete example
   ```javascript
   const { MongoClient } = require('mongodb');

   async function main() {
     const uri = 'mongodb://localhost:10260';
     const client = new MongoClient(uri);

     try {
       await client.connect();
       console.log('Connected to DocumentDB');

       const db = client.db('myapp');
       const collection = db.collection('users');

       // Create an index
       await collection.createIndex({ email: 1 }, { unique: true });

       // Insert documents
       const insertResult = await collection.insertMany([
         { name: "John Doe", email: "john@example.com", age: 30 },
         { name: "Jane Smith", email: "jane@example.com", age: 25 },
         { name: "Bob Johnson", email: "bob@example.com", age: 35 }
       ]);
       console.log('Inserted documents:', insertResult.insertedIds);

       // Find documents
       const users = await collection.find({ age: { $gte: 30 } }).toArray();
       console.log('Users 30 and older:', users);

       // Update a document
       const updateResult = await collection.updateOne(
         { email: "john@example.com" },
         { $set: { status: "active" } }
       );
       console.log('Updated document:', updateResult.modifiedCount);

       // Aggregate data
       const stats = await collection.aggregate([
         { $group: { _id: null, avgAge: { $avg: "$age" }, count: { $sum: 1 } } }
       ]).toArray();
       console.log('User statistics:', stats[0]);

     } catch (error) {
       console.error('Error:', error);
     } finally {
       await client.close();
       console.log('Disconnected from DocumentDB');
     }
   }

   main().catch(console.error);
   ```

## Working with Promises and Async/Await

1. Promise-based operations
2. Async/await patterns
3. Error handling
4. Connection management

## Advanced Features

1. Bulk operations
2. Aggregation framework
3. Vector search
4. Geospatial queries
5. Change streams
6. Transactions

## Error Handling

1. Connection errors
2. Operation errors
3. Timeout handling
4. Retry strategies

## Best Practices

1. Connection pooling
2. Query optimization
3. Bulk operations
4. Error handling
5. Security considerations

## Sample Applications

1. Basic CRUD application
2. REST API with Express
3. Vector search example
4. Real-time applications with change streams

## Testing

1. Setting up test environment
2. Unit testing with Jest/Mocha
3. Integration testing
4. Mock testing

## Deployment

1. Development setup
2. Production considerations
3. Monitoring and logging
4. Performance optimization

## Next Steps

- Explore advanced features
- Learn about indexing strategies
- Build your first application 
