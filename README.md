# DynamoDB

# Task 1 - Setup and Basic CLI Operations with DynamoDB Local

## Start your DynamoDB local container
Run your Docker Compose file to launch DynamoDB locally:

```bash
docker-compose up -d
```

## install AWS CLI

AWS CLI lets you interact with DynamoDB and other AWS services via terminal.


```bash

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

```

### Confirm installation 

````
aws --version
````

## Configure AWS CLI for local DynamoDB

You don’t need real AWS creds here, just configure creds to avoid errors:

```bash
aws configure
```
You will be prompted for the following:

```

AWS Access Key ID [None]: fakeMyKeyId
AWS Secret Access Key [None]: fakeSecretAccessKey
Default region name [None]: us-west-2
Default output format [None]: json

```

## Create your DynamoDB table: Movies

add permission to the folder:
```bash
sudo chmod -R 777 ./docker/dynamodb
```
Create the table with a composite primary key (year as partition key, title as sort key):
````
aws dynamodb create-table \
    --table-name Movies \
    --attribute-definitions AttributeName=year,AttributeType=N AttributeName=title,AttributeType=S \
    --key-schema AttributeName=year,KeyType=HASH AttributeName=title,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
    --endpoint-url http://localhost:8000
````

## Insert data into your table from the JSON file
````
aws dynamodb batch-write-item \
    --request-items file://movies.json \
    --endpoint-url http://localhost:8000
````
## Select 

Get all movies released in 2020:
````
aws dynamodb query \
    --table-name Movies \
    --key-condition-expression "#yr = :y" \
    --expression-attribute-names '{"#yr":"year"}' \
    --expression-attribute-values '{":y":{"N":"2020"}}' \
    --endpoint-url http://localhost:8000 \
    --no-paginate
````
# Task 2 - CRUD Operations on Movies using AWS CLI

## Retrieve one movie
````
aws dynamodb get-item \
  --table-name Movies \
  --key '{"year": {"N": "2020"}, "title": {"S": "The Last Stand"}}' \
  --endpoint-url http://localhost:8000 \
  --no-paginate | jq
````

## Update a movie’s info attribute
````
aws dynamodb update-item \
  --table-name Movies \
  --key '{"year": {"N": "2020"}, "title": {"S": "The Last Stand"}}' \
  --update-expression "SET info = :i" \
  --expression-attribute-values '{":i": {"S": "Action thriller movie"}}' \
  --endpoint-url http://localhost:8000 \
  --return-values ALL_NEW | jq
````

## Check the updated movie

## Delete a movie
````
aws dynamodb delete-item \
  --table-name Movies \
  --key '{"year": {"N": "2019"}, "title": {"S": "Comedy Nights"}}' \
  --endpoint-url http://localhost:8000 \
  --no-paginate
````
## Verify deletion

````
aws dynamodb get-item \
    --table-name Movies \
    --key '{"year": {"N": "2019"}, "title": {"S": "Comedy Nights"}}' \
    --endpoint-url http://localhost:8000 \
    --no-paginate | jq

````

## Advanced Query
 Get all movies from year 2020 whose title starts with "The" AND whose info contains "Action" (case-sensitive)


# Task 3 - DynamoDB with Python

## Install boto3

pip install boto3


## Python script to create Actors table 

The Actors table uses actor_id as partition key and movie_title as sort key.

````python
import boto3

# Connect to local DynamoDB
dynamodb = boto3.resource('dynamodb', endpoint_url='http://localhost:8000', region_name='us-west-2')

# Create Actors table
table = dynamodb.create_table(
    TableName='Actors',
    KeySchema=[
        {'AttributeName': 'actor_id', 'KeyType': 'HASH'},  # Partition key
        {'AttributeName': 'movie_title', 'KeyType': 'RANGE'}  # Sort key
    ],
    AttributeDefinitions=[
        {'AttributeName': 'actor_id', 'AttributeType': 'S'},
        {'AttributeName': 'movie_title', 'AttributeType': 'S'}
    ],
    ProvisionedThroughput={'ReadCapacityUnits': 5, 'WriteCapacityUnits': 5}
)

table.wait_until_exists()
print(f"Table status: {table.table_status}")

`````

##  Insert sample actors data

Create a new Python file insert_actors.py:

````python
actors = [
    {'actor_id': 'a1', 'movie_title': 'The Last Stand', 'name': 'Arnold Schwarzenegger', 'role': 'Lead'},
    {'actor_id': 'a2', 'movie_title': 'The Last Stand', 'name': 'Forest Whitaker', 'role': 'Support'},
    {'actor_id': 'a3', 'movie_title': 'Comedy Nights', 'name': 'Comedian X', 'role': 'Lead'},
]

table = dynamodb.Table('Actors')
with table.batch_writer() as batch:
    for actor in actors:
        batch.put_item(Item=actor)

print("Actors inserted.")

````

## Select actors from the table who played in "The Last Stand"

````python
import boto3
from boto3.dynamodb.conditions import Attr

dynamodb = boto3.resource('dynamodb', endpoint_url='http://localhost:8000', region_name='us-west-2')
table = dynamodb.Table('Actors')

response = table.scan(
    FilterExpression=Attr('movie_title').eq('The Last Stand')
)

for actor in response['Items']:
    print(actor)
````  

## Delete an actor from the Actors table 

Delete actor by actor_id and movie_title (primary key):
'actor_id': 'a1',
'movie_title': 'The Last Stand'

````python
import boto3
from boto3.dynamodb.conditions import Key

dynamodb = boto3.resource('dynamodb', endpoint_url='http://localhost:8000', region_name='us-west-2')
table = dynamodb.Table('Actors')
response = table.delete_item(
    Key={
        'actor_id': 'a1',
        'movie_title': 'The Last Stand'
    }
)
print("Actor deleted.")
````
## Select all actors from the Actors table


## Scan movies released after a 2016 and whose title starts with "The"



## Scan movies released after a 2016 and whose title starts with "The" and info contains "Action"


