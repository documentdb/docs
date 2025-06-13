# Node.js Setup Guide

Learn how to set up and use DocumentDB with Node.js using the official MongoDB Node.js driver.

## Prerequisites

- Node.js 14.x or later
- npm or yarn package manager
- DocumentDB installed and running
- Basic Node.js knowledge

## Setting up DocumentDB with Docker

Before connecting from Node.js, make sure you have a running DocumentDB instance using Docker:

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
   - Insert operations
   - Find operations
   - Update operations
   - Delete operations

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