The PPC stores most of its state in three DynamoDB tables. Though this is currently our only real use of DynamoDB, we expect that we will lean on Dynamo for more scenarios in the future. As such, it behooves us to document common tips, tricks, and pitfalls encountered when using DynamoDB.

# Tips & Tricks
_fill this in as we learn_

# Pitfalls
Although DynamoDB is a schemaless key-value store, it has a few attributes that distinguish it from other key-value stores/document databases. This is especially true for databases that need to retain compatibility between upgrades. Although it is possible to solve many/all of these issues by migrating table items on upgrade paths, this approach may affect service availability. As a result, we should strive to follow these simple rules when changing the contents of the items that exist in a given table to ensure that we do not break compatibility between builds/releases/etc.:

1. DO NOT remove existing fields.
2. DO NOT change the type of existing fields.
3. DO NOT add new fields with complex (i.e. map/struct) types if the fields of these types are to be set using update expressions. DynamoDB (at least) does not support updating the fields of a nested map attribute if the map 
attribute itself does not exist (https://forums.aws.amazon.com/thread.jspa?threadID=162907).

In any of the above circumstances, add a new field with the desired type to an existing type in the item.
