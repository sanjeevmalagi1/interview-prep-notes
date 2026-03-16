What is Gopuff?
Gopuff delivers goods typically found in a convenience store via rapid delivery and 500+ micro distribution centers (DCs). Similar to Blinkit, Zepto etc

## Requirements
### Functional Requirements
- Customers should be able to query availability of items, deliverable within 1 hr, by location
- Customers should be able to place orders with multiple items.

##### Out of scope
- Handling payments/purchases.
- Handling driver routing and deliveries.
- Search functionality and catalog APIs. (The system is strictly concerned with availability and ordering).
- Cancelling and returns

### Non Functional Requirements
- Availability requests should be fast 
- Ordering should be strongly consistent: two customers should not be able to purchase the same physical product.
- System should be able to support 10k DCs and 100k items in the catalog across DCs.
- Order volume will be O(1m)

## Core Entities
- Inventory: Physical item present in a location DC
- Item: Type of the item
- DistributionCenter: physical location where items are stored. We will use this to determine which items are available to a user. Inventory are stored in DCs.
- Order: A collection of inventory which has been ordered.

## Interfaces
Basic APIs will look like this:
  - Get availability of items from a user's perspective.
  ```
  GET /v1/availability?lat=xx&lng=xx&page_num=xx&page_size=xx
  Response:
  {
    items: [
      {
        name: "Cheetos",
        quantity: n
      },
      ...
    ]
  }
  ```
  
  - Place an order with items:
  ```
  POST /v1/order
  {
    lat: xx,
    lng: xx,
    items: [item1, item2, xxx]
  }

  Response: Success/Failure
  ```


## High Level Design
### Customers should be able to query availability of items
- We make a request to the `Availability Service` with the user's location X and Y and any relevant filters.
- The availability service fires a request to the `Nearby Service` with the user's location X and Y.
- The nearby service returns us a list of DCs that can deliver to our location.
- With the DCs available, the availability service query our database with those DC IDs
- We sum up the results and return them to our client.

### Customers should be able to order items.
- The user makes a request to the Orders Service to place an order for items A, B, and C.
- The Orders Service makes creates a singular transaction
  - Check availability of Inventory
  - if available updates the status of inventory items A,B and C to `ordered`
  - A new row is created in orders (And orderItems) table recording the order A, B and C.


## Deep Dives

### Make availability lookups incorporate traffic and drive time
We build on the previous solution to sync periodically (like every 5 minutes) from a DC table to memory of our service. When an input comes in, we can prune down the "candidate" DCs by taking a fixed radius (say 60 miles, the most optimistic distance we could drive over in 1 hour) and limiting ourselves to only evaluating those. We'll take these restricted candidates and then pass to the external travel time service to create our final estimate

### Make availability lookups fast and scalable
Since we only ever read inventory from a nearby collection of DC’s, we can group them together with a region ID using the first 3 digits of their zipcode. Then we can partition our inventory based on this region IDs. This means all queries will go to mostly 1 or 2 partitions rather than the entire inventory dataset.
We can also use read replicas for availability since we can tolerate a small amount of inconsistency. Our orders need to be strongly consistent, so those transactions need to be sent to the Postgres leader, but our availability queries can go to our read replicas.
