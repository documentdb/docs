---
title: Python Setup Guide
description: Learn how to set up and use DocumentDB with Python using the official MongoDB Python driver (PyMongo).
---

# Python Setup Guide

Learn how to set up and use DocumentDB with Python using the official MongoDB Python driver (PyMongo).

## Prerequisites

- Python 3.9 or later (required by PyMongo 4.x)
- pip package manager
- DocumentDB installed and running (see [Pre-built Packages](https://documentdb.io/docs/getting-started/packages/))
- Docker (if DocumentDB is not set up yet)
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
   docker pull ghcr.io/documentdb/documentdb/documentdb-local:latest

   # Tag the image for convenience
   docker tag ghcr.io/documentdb/documentdb/documentdb-local:latest documentdb

   printf 'DocumentDB username: '; IFS= read -r DOCUMENTDB_USERNAME
   printf 'DocumentDB password: '; IFS= read -r -s DOCUMENTDB_PASSWORD; echo
   export DOCUMENTDB_USERNAME DOCUMENTDB_PASSWORD

   # Run the container with your chosen username and password
   if docker run -dt -p 127.0.0.1:10260:10260 --name documentdb-container \
     -e USERNAME="${DOCUMENTDB_USERNAME:?DocumentDB username cannot be empty}" \
     -e PASSWORD="${DOCUMENTDB_PASSWORD:?DocumentDB password cannot be empty}" \
     documentdb; then
     docker image rm -f ghcr.io/documentdb/documentdb/documentdb-local:latest
   fi
   ```
   > **Note:** During the transition to the Linux Foundation, Docker images may still be hosted on Microsoft's container registry. These will be migrated to the new DocumentDB organization as the transition completes.
   > **Note:** The prompts export the credentials for the Python examples below. The guards reject empty values so the container cannot fall through to the public `default_user` / `Admin100` defaults. If you skip this Docker setup or open a new shell, set both environment variables before running Python.
   >
   > **Readiness Note:** `docker ps` reports the container as `Up` before DocumentDB can accept connections. Wait for the ready banner first: `until docker logs documentdb-container 2>&1 | grep -q "=== DocumentDB is ready ==="; do sleep 2; done`
   > 
   > **Telemetry note:** The 1.0 image sends usage data (its version, platform and CPU architecture) by default. To turn it off, add `-e DOCUMENTDB_USAGE_TELEMETRY=false` before the image name. See [Usage telemetry](../documentdb-local/index.md#usage-telemetry).
   >
   > **Network Note:** The example binds the gateway only to the local host. Expose it to other machines only after adding firewall rules and a certificate those clients can validate.
   >
   > **Port Note:** To use host port `27017` while leaving the gateway on its default container port, publish `-p 127.0.0.1:27017:10260` and connect to `localhost:27017`. To change the gateway's internal port too, add `--documentdb-port 27017` after the image name and publish that container port.

## Connecting to DocumentDB

DocumentDB Local accepts TLS connections on the gateway port and requires authentication. Connect with the username and password you set when starting the container, and because the container uses a self-signed certificate, the simplest local setup skips certificate validation with `tlsAllowInvalidCertificates=true` (in production, provide the gateway certificate instead).

The code reads the same `DOCUMENTDB_USERNAME` and `DOCUMENTDB_PASSWORD` values exported during Docker setup and raises a clear error if either is missing.

1. Basic Connection
   ```python
   import os
   import pymongo

   username = os.environ.get('DOCUMENTDB_USERNAME')
   password = os.environ.get('DOCUMENTDB_PASSWORD')
   if not username or not password:
       raise RuntimeError(
           'Set DOCUMENTDB_USERNAME and DOCUMENTDB_PASSWORD before connecting'
       )

   client = pymongo.MongoClient(
       'mongodb://localhost:10260/',
       username=username,
       password=password,
       authSource='admin',
       tls=True,
       tlsAllowInvalidCertificates=True
   )

   # Specify the database to be used
   db = client.sample_database

   # Specify the collection
   collection = db.sample_collection
   ```

2. Connection with certificate validation
   ```python
   client = pymongo.MongoClient(
       'mongodb://localhost:10260/',
       username=username,
       password=password,
       authSource='admin',
       tls=True,
       tlsCAFile='/path/to/documentdb-cert.pem'
   )
   ```

3. Connection with Options
   ```python
   # With additional options
   client = pymongo.MongoClient(
       'mongodb://localhost:10260/',
       username=username,
       password=password,
       authSource='admin',
       tls=True,
       tlsAllowInvalidCertificates=True,
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
   from datetime import datetime, timezone

   # Insert a single document
   collection.insert_one({
       'name': 'John Doe',
       'email': 'john@example.com',
       'created_at': datetime.now(timezone.utc)
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
   from datetime import datetime, timezone

   # Insert with timestamp
   collection.insert_one({
       'name': 'Event',
       'timestamp': datetime.now(timezone.utc)
   })
   ```

## Advanced Features

1. Bulk operations
   ```python
   from pymongo import UpdateMany, DeleteMany

   result = collection.bulk_write([
       UpdateMany({'status': 'pending'}, {'$set': {'status': 'processed'}}),
       DeleteMany({'age': {'$lt': 18}}),
   ])
   print(result.modified_count, result.deleted_count)
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

   `$vectorSearch` is an aggregation stage and must be the first stage in the
   pipeline — it does not work inside `find()`. It also requires a vector index
   on the field, or the query fails with
   `Similarity index was not found for a vector similarity search query`.

   Create the index once:

   ```python
   db.command({
       'createIndexes': 'sample_collection',
       'indexes': [
           {
               'name': 'embeddings_idx',
               'key': {'embeddings': 'cosmosSearch'},
               'cosmosSearchOptions': {
                   'kind': 'vector-ivf',
                   'numLists': 100,
                   'similarity': 'COS',
                   'dimensions': 3
               }
           }
       ]
   })
   ```

   `dimensions` must match the length of the vectors you store and query. Three keeps
   the example short; real embeddings are typically 384, 768, or 1536 wide.

   ```python
   results = collection.aggregate([
       {
           '$vectorSearch': {
               'queryVector': [0.1, 0.2, 0.3],
               'path': 'embeddings',
               'numCandidates': 100,
               'limit': 10
           }
       }
   ])
   ```

## Error Handling

1. Connection errors
   ```python
   from pymongo.errors import ConnectionFailure

   try:
       client.admin.command('ping')
   except ConnectionFailure as e:
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
       'mongodb://localhost:10260/',
       username=username,
       password=password,
       authSource='admin',
       tls=True,
       tlsAllowInvalidCertificates=True,
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
   try:
       collection.insert_one({'name': 'Example'})
   finally:
       client.close()
   ```

## Sample Application

```python
import os
from flask import Flask, jsonify
from pymongo import MongoClient

app = Flask(__name__)
username = os.environ.get('DOCUMENTDB_USERNAME')
password = os.environ.get('DOCUMENTDB_PASSWORD')
if not username or not password:
    raise RuntimeError(
        'Set DOCUMENTDB_USERNAME and DOCUMENTDB_PASSWORD before starting the app'
    )

client = MongoClient(
    'mongodb://localhost:10260/',
    username=username,
    password=password,
    authSource='admin',
    tls=True,
    tlsAllowInvalidCertificates=True
)
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

- Explore advanced features in the [API Reference](https://documentdb.io/docs/reference/)
- Check out the [MongoDB Shell Guide](https://documentdb.io/docs/getting-started/mongo-shell-quickstart/) for additional query examples
