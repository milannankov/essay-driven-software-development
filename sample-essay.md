# Sample Essay - Implementing Data On Demand
*This essay demonstrates how you can reason about a potential feature without going into technical details. This is actually the first thing you do when there is an idea about a new feature or an improvement. First try defending why that thing should be built/implemented. Sometimes as this sample demonstrates, the idea is not worth implementing.*

## Background
The app is at a point where we must decide if downloading all data and constantly keeping it up to date is worth it. From today's perspective that might not be the case and here I will explore the idea to only download data when it is need, i.e. when bulk operations executed. 

For auto tagiggng we can easily use Tag Rules which constantly get data using webhooks. Still, if we want to trigger any rule manually, all data must be available or be retrieved.

### Initial Arguments for Downloading All Data
- **Fast bulk operations** - having the data on hand means that we can analyze it quickly and schedule tag updates almost immediately without having to download data using the Shopify API
- **Shopify API is slow** - getting data using the Shopify API is super slow for large collection and it might take hours to get all data required for some bulk operations. Image having to download 50,000 thousand items from Shopify for every operation.
- **Fast Preview** - we can show a preview with all changing items -> no other app has this feature.
- **Tag Dashboard** - we have dashboard with all tags by products / customers / etc. If data is not downloaded we cannot offer this feature.

**Current Problems With Downloading All Data**
- Some stores have huge amount of data which seemed to be only a corner case for some stores but with the addition of Customer and Order tagging the problem only got worse.
- The intial asumption that products are the largest data set is definitely not true. Customers and Order can be for stores that have been operating for a long time. For example, we had a store with 650K customers!!!
- Merchants only need to work with all data when they are doing bulk updates. For other times we can deal with partial updates either through webhooks or through bulk updates. 

**New Assumptions**
- We only need tag data in order to show the dashboard with maybe only the title and customer / order title.
- Downloading all customer data is danegerous and also does not make much sense outside of the case where all data items must be update which happens only when bulk updates are run.
- Tag Rules kind of elliminate the need to have bulk operations - whey are just needed in bulk scenarios
- It might be possible to be very smart and download data for each bulk operation (just the data that is needed) process it and then throw it away. This approach might be even faster for most cases. 
- We can have two separate concepts that are completely separate - Bulk Operations and Tag Rules

## Analysis

**What needs to Change**

- First, a new way of executing and scheduling shopify bulk operations must be created
- Currently, only updates are done using shopify bulk updates, now we need to have the bulk data fetches happen with Shopify bulk operations
- Multiple actions can request a shopify bulk operation.
    - Init - product, customer order, collections
    - Sync/Update of products/customers/orders
    - Bulk operations
- We need to be smarter about scheduleing data sync operations - if there is one already on the queue, remove any duplicates?
- Currently, the concept of shopify bulk operations has been tightly coupled to the data sync itself. Now we need to separate those two concepts and have a shopify bulk operation which just knows about its state and that it has some File - as opposed to a separate bulk update operation which knows how to handle the data once the shopify operation has been finished

**Possible Pitfalls**
- Having multiple shopify bulk operations might delay the data sync since more type of operations will be on the queue - that might not be such a problem because bulk operations should be rare once we migrate merchats to using Rules

**Shopify Bulk Operation Queue**  
There seems to be a need for one queue to handle all incoming request for Shopify bulk operation.
This queue should not know about the source and what logic causes a request to be made. This queue should govern the rules around shopify bulk updates like: only one operation can be active at a time, etc.

When an operation is complete, we should signal the oppropriate handler based on the TriggerSourceType information. Interesting case is what happens if an bulk operation fails? What happens next???

There should be a single service that governs shopify bulk operation submissions - ShopifyBulkOperationScheduler

### Questions and TODOS
- What happens when a shopify bulk operation fails? Do we delete the queue item and hand over to the handlers to decide how to deal?
- Do we keep the Shopif Bulk Operation items? For how long?

**Should We Event Go With This Plan?**
Let's list pros and cons of each option. 

Keeping All Data  
- (+) Can start an operation without fetching data using Shopify API or Shopify bulk operation  
- (+) Can easily do previews since we have the data 
- (+) Can easily do auto-complete for various fields (better UX) 
- (-) Privacy/security issues  
- (-) Data takes up a lot of space  
- (-) Might be difficult to handle super-large stores

Having Minimal Data
- (+) Should be able to handle all store sizes more easily
- (-) Will need bulk operations to get data for every execution
- (-) More complex to implement
- (-) Difficult to do previews
- (-) No direct impact on customer-driven features and desires

Maybe it is not worth it to try to implement this plan. Even after the feature is completed which will require a lot of changes, the app will not be better for the merchants - it will only be an implemention for optimizing. Currenly we are in no way pressured by our infrastructure to make improvements.

## Implementation
[SKIPPED]

## Summary
Initially I was convinced that we need to implement data on demand but after carefult analysis it turned out that it will not give any benefit to the end customer, at least at this time. We will save saving some data but we will introduce a lot of complexity into the system which does not seem to directly impact the merchants. 

What's more, data on demand will not allow us to offer Full Preview of operations and will not allow us to introduce other features that require having saved data.

Currently there is no substantial evidence to support the implementation of this change.
At some point we might return to this topic if we realize the merchants do not care about having Full Preview or use other features that rely on having data.

