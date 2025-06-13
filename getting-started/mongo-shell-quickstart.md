# MongoDB Shell Quick Start

Get started with DocumentDB using the MongoDB shell for a familiar MongoDB-compatible experience.

## Prerequisites

- MongoDB Shell (mongosh) installed
- Docker Desktop installed and running
- Basic MongoDB knowledge
- Git installed (for cloning the repository)

## Installation

1. Setting up DocumentDB locally
   ```bash
   # Pull the latest DocumentDB Docker image
   docker pull ghcr.io/microsoft/documentdb/documentdb-local:latest

   # Tag the image for convenience
   docker tag ghcr.io/microsoft/documentdb/documentdb-local:latest documentdb

   # Run the container with your chosen username and password
   docker run -dt -p 10260:10260 --name documentdb-container documentdb --username <YOUR_USERNAME> --password <YOUR_PASSWORD>
   docker image rm -f ghcr.io/microsoft/documentdb/documentdb-local:latest || echo "No existing documentdb image to remove"
   ```
   > **Note:** Replace `<YOUR_USERNAME>` and `<YOUR_PASSWORD>` with your desired credentials. You must set these when creating the container for authentication to work.
   > 
   > **Port Note:** Port `10260` is used by default in these instructions to avoid conflicts with other local database services. You can use port `27017` (the standard MongoDB port) or any other available port if you prefer. If you do, be sure to update the port number in both your `docker run` command and your connection string accordingly.

2. Starting the server
   ```bash
   # The server will be available at localhost:10260 (or your chosen port)
   # You can verify the server is running using:
   docker ps
   ```

## Connecting to DocumentDB

Connection string format
   ```bash
   mongosh "mongodb://<YOUR_USERNAME>:<YOUR_PASSWORD>@localhost:10260/?tls=true&tlsAllowInvalidCertificates=true"
   ```

## Basic Operations

1. Creating databases and collections
   ```javascript
   // Create/switch to a database
   use mydb

   // Create a collection
   db.createCollection("users")

   // Create another collection
   db.createCollection("logs")
   ```

2. Inserting documents
   ```javascript
   // Insert a single document
   db.users.insertOne({ name: "John Doe", email: "john@example.com", created_at: new Date() })

   // Insert multiple documents
   db.users.insertMany([{ name: "Jane Smith", email: "jane@example.com" }, { name: "Bob Johnson", email: "bob@example.com" }])
   ```

3. Querying documents
   ```javascript
   // Find all documents
   db.users.find()

   // Find with criteria
   db.users.find({ name: "John Doe" })

   // Find with projection
   db.users.find({}, { name: 1, email: 1, _id: 0 })

   // Complex queries
   db.users.find({ $and: [{ created_at: { $gte: new Date("2025-01-01") } }, { email: { $regex: "@example.com$" } }] })
   ```

4. Updating documents
   ```javascript
   // Update a single document
   db.users.updateOne({ name: "John Doe" }, { $set: { status: "active" } })

   // Update multiple documents
   db.users.updateMany({ email: { $regex: "@example.com$" } }, { $set: { domain: "example.com" } })
   ```

5. Deleting documents
   ```javascript
   // Delete a single document
   db.users.deleteOne({ name: "John Doe" })

   // Delete multiple documents
   db.users.deleteMany({ status: "inactive" })
   ```

## Working with Indexes

1. Understanding index types
   ```javascript
   // Available index types:
   // - Single field
   // - Compound
   // - Multi-key
   // - Text
   // - Geospatial
   // - Vector
   ```

2. Creating indexes
   ```javascript
   // Single field index
   db.users.createIndex({ email: 1 })

   // Compound index
   db.users.createIndex({ name: 1, email: 1 })

   // Text index
   db.articles.createIndex({ content: "text" })

   // Geospatial index
   db.places.createIndex({ location: "2dsphere" })

   // Vector index
   db.products.createIndex({ 
     embedding: "vector",
     vectorOptions: { dimensions: 384 }
   })
   ```

3. Index strategies
   ```javascript
   // Unique index
   db.users.createIndex({ email: 1 }, { unique: true })

   // Partial index
   db.orders.createIndex(
     { orderDate: 1 },
     { partialFilterExpression: { status: "active" } }
   )
   ```

## Advanced Features

1. Aggregation pipeline
   ```javascript
   db.orders.aggregate([
     { $match: { status: "completed" } },
     { $group: {
         _id: "$customer",
         total: { $sum: "$amount" },
         count: { $sum: 1 }
     }},
     { $sort: { total: -1 } }
   ])
   ```

2. Vector search operations
   ```javascript
   db.products.find({
     $vectorSearch: {
       queryVector: [0.1, 0.2, 0.3],
       path: "embedding",
       numCandidates: 100,
       limit: 10
     }
   })
   ```

3. Geospatial queries
   ```javascript
   db.places.find({
     location: {
       $near: {
         $geometry: {
           type: "Point",
           coordinates: [-73.9667, 40.78]
         },
         $maxDistance: 1000
       }
     }
   })
   ```

## Monitoring and Management

1. Server status commands
   ```javascript
   // Get server status
   db.serverStatus()

   // Get server information
   db.runCommand({ buildInfo: 1 })
   ```

2. Database statistics
   ```javascript
   // Get database stats
   db.stats()

   // Get collection stats
   db.users.stats()
   ```

3. Collection statistics
   ```javascript
   // Get collection size
   db.users.dataSize()

   // Get index sizes
   db.users.stats().indexSizes
   ```

## Best Practices

1. Connection management
   - Use connection pooling
   - Set appropriate timeouts
   - Handle reconnection logic

2. Query optimization
   - Use explain() for query analysis
   - Create appropriate indexes
   - Monitor query performance

3. Error handling
   ```javascript
   try {
     db.users.insertOne({ _id: 1, name: "Test" })
   } catch (error) {
     print("Error:", error.message)
   }
   ```

## Next Steps

- Explore the API reference for advanced features
- Learn about advanced features in the Architecture section
- Connect your application using one of our language guides 