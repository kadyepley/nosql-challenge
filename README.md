# nosql-challenge
# Eat Safe, Love
# Part 1: Database and Jupyter Notebook Set Up
Import the data provided in the `establishments.json` file from your Terminal. Name the database `uk_food` and the collection `establishments`.

Within this markdown cell, copy the line of text you used to import the data from your Terminal. This way, future analysts will be able to repeat your process.

e.g.: Import the dataset with mongoimport --type json -d uk_food -c establishments --drop --jsonArray establishments.json


### Import dependencies

from pymongo import MongoClient
from pprint import print

### Create an instance of MongoClient

mongo = MongoClient(port=27017)

### Confirm that our new database was created

print(mongo.list_database_names())

### Assign the uk_food database to a variable name

db = mongo['uk_food']

### Review the collections in our new database

collections_list = db.list_collection_names()

### Review the collections in our new database

for collection in collections_list:
    print(collection)
    
### Review a document in the establishments collection

document = db.establishments.find_one()
print(document)

### Assign the collection to a variable

establishments = db['establishments']

# Part 2: Update the Database
## 1. An exciting new halal restaurant just opened in Greenwich, but hasn't been rated yet. The magazine has asked you to include it in your analysis. Add the following restaurant "Penang Flavours" to the database.
   
### Create a dictionary for the new restaurant data

new_rest_data = {    
    "BusinessName":"Penang Flavours",
    "BusinessType":"Restaurant/Cafe/Canteen",
    "BusinessTypeID":"",
    "AddressLine1":"Penang Flavours",
    "AddressLine2":"146A Plumstead Rd",
    "AddressLine3":"London",
    "AddressLine4":"",
    "PostCode":"SE18 7DY",
    "Phone":"",
    "LocalAuthorityCode":"511",
    "LocalAuthorityName":"Greenwich",
    "LocalAuthorityWebSite":"http://www.royalgreenwich.gov.uk",
    "LocalAuthorityEmailAddress":"health@royalgreenwich.gov.uk",
    "scores":{
        "Hygiene":"",
        "Structural":"",
        "ConfidenceInManagement":""
    },
    "SchemeType":"FHRS",
    "geocode":{
        "longitude":"0.08384000",
        "latitude":"51.49014200"
    },
    "RightToReply":"",
    "Distance":4623.9723280747176,
    "NewRatingPending":True
}

### Insert the new restaurant into the collection

db_updated = establishments.insert_one(new_rest_data)

### Check that the new restaurant was inserted

establishments.find_one({'BusinessName' : 'Penang Flavours'})

## 2. Find the BusinessTypeID for "Restaurant/Cafe/Canteen" and return only the `BusinessTypeID` and `BusinessType` fields.

### Find the BusinessTypeID for "Restaurant/Cafe/Canteen" and return only the BusinessTypeID and BusinessType fields

query = {'BusinessType' : 'Restaurant/Cafe/Canteen'}
fields = ['BusinessTypeID', 'BusinessType']

establishments.find_one(query, fields)

## 3. Update the new restaurant with the `BusinessTypeID` you found.
## Update the new restaurant with the correct BusinessTypeID

establishments.update_one(
    new_rest_data,
    {'$set' :
        {'BusinessTypeID': 1}
    }
)

### Confirm that the new restaurant was updated

establishments.find_one({'BusinessName' : 'Penang Flavours'})

## 4. The magazine is not interested in any establishments in Dover, so check how many documents contain the Dover Local Authority. Then, remove any establishments within the Dover Local Authority from the database, and check the number of documents to ensure they were deleted.

### Find how many documents have LocalAuthorityName as "Dover"

establishments.count_documents({'LocalAuthorityName' : 'Dover'})

### Delete all documents where LocalAuthorityName is "Dover"

establishments.delete_many({'LocalAuthorityName' : 'Dover'})

### Check if any remaining documents include Dover

establishments.count_documents({'LocalAuthorityName' : 'Dover'})

### Check that other documents remain with 'find_one'

establishments.find_one({})

## 5. Some of the number values are stored as strings, when they should be stored as numbers.

Use `update_many` to convert `latitude` and `longitude` to decimal numbers.

### Change the data type from String to Decimal for longitude and latitude

establishments.update_many({}, [{'$set' : {'geocode.longitude' : {'$toDouble': '$geocode.longitude'},
                                           'geocode.latitude' : {'$toDouble' : '$geocode.latitude'}
                                            }
                                }
                                ]
                        )
Use `update_many` to convert `RatingValue` to integer numbers.

### Set non 1-5 Rating Values to Null

non_ratings = ["AwaitingInspection", "Awaiting Inspection", "AwaitingPublication", "Pass", "Exempt"]
establishments.update_many({"RatingValue": {"$in": non_ratings}}, [ {'$set':{ "RatingValue" : None}} ])

### Change the data type from String to Integer for RatingValue

establishments.update_many({}, [{'$set' : {'RatingValue' : {'$toDouble' : '$RatingValue'}
                                           }
                                }
                                ]
                            )
### Check that the coordinates and rating value are now numbers
establishments.find_one()

# Eat Safe, Love

## Notebook Set Up

### Import dependencies

from pymongo import MongoClient
from pprint import print

### Create an instance of MongoClient

mongo = MongoClient(port=27017)

### Assign the uk_food database to a variable name

db = mongo['uk_food']

### Review the collections in our database

print(db.list_collection_names())

### Assign the collection to a variable

establishments = db['establishments']

# Part 3: Exploratory Analysis

Unless otherwise stated, for each question: 
* Use `count_documents` to display the number of documents contained in the result.
* Display the first document in the results using `pprint`.
* Convert the result to a Pandas DataFrame, print the number of rows in the DataFrame, and display the first 10 rows.
  
## 1. Which establishments have a hygiene score equal to 20?

### Find the establishments with a hygiene score of 20

query = {'scores.Hygiene' : 20}

### Use count_documents to display the number of documents in the result

count = establishments.count_documents(query)
print(f'Number of Documents: {count}')

### Display the first document in the results using print

pprint(establishments.find_one(query))

### Convert the result to a Pandas DataFrame

### Import the dependency

import pandas as pd

### Create the dataframe

hygiene_df = pd.DataFrame(establishments.find(query))

### Display the number of rows in the DataFrame

print(f'Number of Rows: {len(hygiene_df)}')

### Display the first 10 rows of the DataFrame

hygiene_df.head(10)

## 2. Which establishments in London have a `RatingValue` greater than or equal to 4?

### Find the establishments with London as the Local Authority and has a RatingValue greater than or equal to 4.

query2 = {'LocalAuthorityName' : {'$regex' : 'London'},
         'RatingValue' : {'$gte' : '4'}
         }

### Use count_documents to display the number of documents in the result

count2 = establishments.count_documents(query2)
print(f'Number of Documents: {count2}')

### Display the first document in the results using print

pprint(establishments.find_one(query2))

### Convert the result to a Pandas DataFrame

rating_val_df = pd.DataFrame(establishments.find(query2))

### Display the number of rows in the DataFrame

print(f'Number of Rows: {len(rating_val_df)}')

### Display the first 10 rows of the DataFrame

rating_val_df.head(10)

## 3. What are the top 5 establishments with a `RatingValue` rating value of 5, sorted by lowest hygiene score, nearest to the new restaurant added, "Penang Flavours"?

### Search within 0.01 degree on either side of the latitude and longitude.
### Rating value must equal 5
### Sort by hygiene score

### Print the longitude and latitude of Penang 

### pprint(establishments.find_one({'BusinessName' : 'Penang Flavours'}, {'geocode.latitude', 'geocode.longitude'}))

degree_search = 0.01
latitude = 51.49014200
longitude = 0.08384000

query3 = {'RatingValue': '5',
    'geocode.latitude' : {'$lte': latitude + degree_search, '$gte': latitude - degree_search},
    'geocode.longitude' : {'$lte': longitude + degree_search, '$gte': longitude - degree_search}
}

### Sort by hygiene score

sort =  [('scores.Hygiene', 1)]

### Print the results

results = establishments.find(query3).sort(sort)
pprint(list(results))

### Convert result to Pandas DataFrame

pd.DataFrame(establishments.find(query3).sort(sort).limit(5))

## 4. How many establishments in each Local Authority area have a hygiene score of 0?

### Create a pipeline that: 

### 1. Matches establishments with a hygiene score of 0

match_query = {'$match' : {'scores.Hygiene' : 0}}

### 2. Groups the matches by Local Authority

group_query = {'$group' : {'_id' : '$LocalAuthorityName',
                           'count' : {'$sum' : 1}
                            }
                }

### 3. Sorts the matches from highest to lowest

sort_values = {'$sort' : {'count' : -1}}

pipeline = [match_query, match_query, sort_values]

results2 = list(establishments.aggregate(pipeline))

### Print the number of documents in the result
print(f'Number of Documents: {len(results2)}')

### Print the first 10 results

pprint(results2[0:10])

### Convert the result to a Pandas DataFrame

aggregated_df = pd.json_normalize(results2)

### Display the number of rows in the DataFrame

print(f'Number of Rows: {len(aggregated_df)}')

### Display the first 10 rows of the DataFrame

aggregated_df.head(10)
