# Python Setup Guide

Learn how to set up and use DocumentDB with Python using the official MongoDB Python driver (PyMongo).

## Prerequisites

- Python 3.7+
- pip package manager
- DocumentDB installed and running (see [Pre-built Packages](prebuilt-packages.md))
- Basic Python knowledge
- Git installed (for cloning the repository)

## Installation

1. Installing the MongoDB Python driver
   ```bash
   pip install pymongo
   ```

2. Optional dependencies
   ```bash
   pip install dnspython  # For connection string support
   ```

## Project Setup (skip if already done)

1. Setting up DocumentDB with Docker
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

## Connecting to DocumentDB

1. Basic Connection
   ```python
   import pymongo
   import sys

   # Create a MongoDB client and open a connection to DocumentDB
   client = pymongo.MongoClient(
       'mongodb://localhost:27017'
   )

   # Specify the database to be used
   db = client.sample_database

   # Specify the collection
   collection = db.sample_collection
   ```

2. Connection with Authentication
   ```python
   # With username and password
   client = pymongo.MongoClient(
       'mongodb://username:password@localhost:27017'
   )
   ```

3. Connection with Options
   ```python
   # With additional options
   client = pymongo.MongoClient(
       'mongodb://localhost:27017',
       maxPoolSize=50,
       retryWrites=False,
       w='majority'
   )
   ```

## Basic Operations

1. Creating collections
   ```python
   # Create a new collection
   db.create_collection('users')

   # Create a collection with options
   db.create_collection('logs')
   ```

2. Document operations
   ```python
   # Insert a single document
   collection.insert_one({
       'name': 'John Doe',
       'email': 'john@example.com',
       'created_at': datetime.utcnow()
   })

   # Insert multiple documents
   collection.insert_many([
       {'name': 'Jane Smith', 'email': 'jane@example.com'},
       {'name': 'Bob Johnson', 'email': 'bob@example.com'}
   ])

   # Find documents
   result = collection.find({'name': 'John Doe'})
   
   # Find with projection
   result = collection.find(
       {'email': {'$regex': '@example.com$'}},
       {'name': 1, 'email': 1, '_id': 0}
   )

   # Update a document
   collection.update_one(
       {'name': 'John Doe'},
       {'$set': {'status': 'active'}}
   )

   # Delete documents
   collection.delete_one({'name': 'John Doe'})
   ```

## Working with BSON Types

1. ObjectId
   ```python
   from bson import ObjectId

   # Find by ObjectId
   doc = collection.find_one({'_id': ObjectId('...')})
   ```

2. DateTime
   ```python
   from datetime import datetime

   # Insert with timestamp
   collection.insert_one({
       'name': 'Event',
       'timestamp': datetime.utcnow()
   })
   ```

## Advanced Features

1. Bulk operations
   ```python
   # Initialize bulk operations
   bulk = collection.initialize_ordered_bulk_op()
   
   # Add operations
   bulk.find({'status': 'pending'}).update({'$set': {'status': 'processed'}})
   bulk.find({'age': {'$lt': 18}}).delete()
   
   # Execute
   result = bulk.execute()
   ```

2. Aggregation framework
   ```python
   pipeline = [
       {'$match': {'status': 'active'}},
       {'$group': {
           '_id': '$type',
           'count': {'$sum': 1},
           'avg_value': {'$avg': '$value'}
       }}
   ]
   results = collection.aggregate(pipeline)
   ```

3. Vector search
   ```python
   # Vector similarity search
   results = collection.find({
       '$vectorSearch': {
           'queryVector': [0.1, 0.2, 0.3],
           'path': 'embeddings',
           'numCandidates': 100,
           'limit': 10
       }
   })
   ```

4. PostgreSQL Integration
   ```python
   # Access PostgreSQL features directly
   from documentdb_api import DocumentDB
   
   # Initialize DocumentDB with PostgreSQL support
   db = DocumentDB(client)
   
   # Execute SQL queries on BSON documents
   result = db.sql_query(
       "SELECT jsonb_path_query(data, '$.name') FROM collection WHERE data @? '$.age > 21'"
   )
   ```

## Error Handling

1. Connection errors
   ```python
   try:
       client = pymongo.MongoClient(connection_string)
       client.admin.command('ping')
   except pymongo.errors.ConnectionError as e:
       print(f"Connection error: {e}")
   ```

2. Operation errors
   ```python
   from pymongo.errors import OperationFailure

   try:
       result = collection.insert_one({'_id': existing_id})
   except OperationFailure as e:
       print(f"Operation failed: {e}")
   ```

## Best Practices

1. Connection pooling
   ```python
   # Configure connection pool
   client = pymongo.MongoClient(
       connection_string,
       maxPoolSize=50,
       waitQueueTimeoutMS=2000
   )
   ```

2. Query optimization
   ```python
   # Use explain for query analysis
   explanation = collection.find({'status': 'active'}).explain()
   ```

3. Proper cleanup
   ```python
   # Always close connections when done
   try:
       # Your code here
   finally:
       client.close()
   ```

## Sample Application

```python
from flask import Flask, jsonify
from pymongo import MongoClient
from datetime import datetime

app = Flask(__name__)
client = MongoClient('mongodb://localhost:27017/')
db = client.sample_database

@app.route('/users', methods=['GET'])
def get_users():
    users = list(db.users.find({}, {'_id': 0}))
    return jsonify(users)

@app.route('/user/<name>', methods=['GET'])
def get_user(name):
    user = db.users.find_one({'name': name}, {'_id': 0})
    return jsonify(user)

if __name__ == '__main__':
    app.run(debug=True)
```

## Next Steps

- Explore advanced features in the [API Reference](../api-reference/index.md)
- Learn about indexing strategies in the [Architecture](../architecture/index.md)
- Check out the [MongoDB Shell Guide](mongo-shell-quickstart.md) for additional query examples 