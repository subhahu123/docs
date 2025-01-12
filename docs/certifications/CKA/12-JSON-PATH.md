# JSON PATH

Query language for json

## Dictionaries

`$` is root element.

```json
{
    "car": {
        "price": "$25,000"
    }
}
```

Query price via `$.car.price`:

```json
[ "$25,000" ]
```

## Lists

```json
[
    1,
    2,
    3
]
```

Query via `$[INDEX]`:

`$[0]` is `[1]`

can do multiple: `[0,2]` is `[1,3]`

Can slice via `[START:END_EXCLUSIVE]`
Can step via `[START:END_EXCLUSIVE:STEP]`
Can get last via `[-1]` or `[-1:]`

## Criteria

More complex queries

```json
[
    1,
    2,
    3,
    4
]
```

`?()` for query.

`@` each item in list

So, get items `> 2`:

```
$[ $( @ > 2) ]`
```

Result: `[3,4]`

Can use `==`, `!=`, `in`, `nin`

Can check strings via `?(@.SOME_KEY == "val")`

## Wildcards

We can also use wildcards, eg all matches:

```
$[*].price
```

Or with Dictionaries:

```
$*.car
```

## JSON PATH in Kubernetes

```shell
kubectl get pods -o=jsonpath='{ .item[0].spec }'
```

`$` is added if not there.

We can combine queries for multiple results:

```shell
kubectl get pods -o=jsonpath='{ .item[0].spec }{ .items[*].metadata }'
```

We can prettify via `{ "\n" }` between queries.

We can loop with a `{range .items[*]}{ ... }{ end }`, and access internal elements inside.

Can also create custom columns via `-o=custom-columns=COL_NAME:JSON_PATH`


