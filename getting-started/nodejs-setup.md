# Node.js Setup Guide

Learn how to set up and use DocumentDB with Node.js using the official MongoDB Node.js driver.

## Prerequisites

- Node.js 14.x or later
- npm or yarn package manager
- DocumentDB installed and running
- Basic Node.js knowledge

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