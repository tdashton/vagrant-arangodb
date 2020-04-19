# multi model database

arango is a three typed db supporting multiple types:

1. graph db
2. document db
3. key / value store

# joins with verticies vs edges

## verticies

if you know in advance you will only need a certain depth of joining collections (verticies) are better

## edges

if you know that you will want to traverse an undetermined number of times, or that the "join" actually contains the bulk of the information to query then you are better off using an edge

# query examples

see some examples below

based on https://github.com/neunhoef/exampleData/tree/master/ecommerce

## standard where

```
FOR c IN customers
    FILTER c.name == 'Hugo'
```

## joins

```
FOR s IN sales
    FOR i IN items
        FILTER i._id == s._to
        COLLECT bill = s.billingId INTO items
        FOR c in CUSTOMERS
            FILTER c._id == items[0].s._from
            RETURN { date: items[0].s.date, billingId: bill,
                name: c.name, price: SUM(items[*].i.price),
                items[*].i }
```

## aggregation / group

```
FOR c IN customers 
    # optionally insert a FILTER here
    COLLECT WITH COUNT INTO count
    RETURN {numberOfCustomers: count}
```

### more advanced

```
FOR c IN customers 
    # optionally insert a FILTER here
    COLLECT y = SUBSTRING(c.memberSince, 0, 4) WITH COUNT INTO count
    SORT y DESC
    RETURN {year: y, numberOfCustomers count}
```

## graphs

```
FOR i IN items
    LET buyers =
        # 1..1 is a range of depths
        (FOR v in 1..1 INBOUND item GRAPH "sales" RETURN v)
    LET nr = LENGTH(buyers)
    SORT nr DESC
    RETURN { description: item.description,
            orderId: item.orderId, nrSales: nr }
```

# edge query example

With the "vertice, edge and path" information returned in the query, so the UI can visualize it.

```
FOR vertice,edge,path IN 1 INBOUND
    "vertices/1" edges // vertices is the vertex collection, and edges the edge collection
```

# graph question

If you do not know how deep into the joins you need to go in advance, the graph function is better suited.

In general: something is linked, so you want to follow the links.