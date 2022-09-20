# cuallee
Meaning `good` in Aztec (Nahuatl)


This library provides an intuitive `API` to describe `checks` for Apache PySpark DataFrames `v3.3.0`.
It is a replacement written in pure `python` of the `pydeequ` framework.

I gave up in _deequ_ as project does not seem to be maintained, and the multiple issues with the callback server.

## Advantages
This implementation goes in hand with the latest API from PySpark and uses the `Observation` API to collect metrics
at the lower cost of computation. 
When benchmarking against pydeequ, `cuallee` uses circa <3k java classes underneath and **remarkably** less memory.
 
> __cuallee__ is inpired by the Green Software Foundation principles, on the advantages of `green` software.


## Checks

### Completeness and Uniqueness
```python
from cuallee import Check
from pyspark.sql import SparkSession
spark = SparkSession.builder.getOrCreate()

# Nulls on column Id
check = Check(CheckLevel.WARNING, "Completeness")
(
    check
    .is_complete("id")
    .is_unique("id")
    .validate(spark, df)
).show() # Returns a pyspark.sql.DataFrame
```

### Date Algebra
```python
# Unique values on id
check = Check(CheckLevel.WARNING, "CheckIsBetweenDates")
df = spark.sql("select explode(sequence(to_date('2022-01-01'), to_date('2022-01-10'), interval 1 day)) as date")
assert (
    check.is_between("date", "2022-01-01", "2022-01-10")
    .validate(spark, df)
    .first()
    .status
)
```

### Value Membership
```python
df = spark.createDataFrame([[1, 10], [2, 15], [3, 17]], ["ID", "value"])
check = Check(CheckLevel.WARNING, "is_contained_in_number_test")
check.is_contained_in("value", (10, 15, 20, 25)).validate(spark, df)
```

### Regular Expressions
```python
df = spark.createDataFrame([[1, "is_blue"], [2, "has_hat"], [3, "is_smart"]], ["ID", "desc"])
check = Check(CheckLevel.WARNING, "matches_regex_test")
check.matches_regex("desc", r"^is.*t$") # only match is_smart 33% of rows.
check.validate(spark, df).first().status == "FAIL"
```

### More...
- `are_complete(*cols)`
- `matches_regex(col, regex)`
- `is_greater_than(col, val)`
- `is_greater_or_equal_than(col, val)`
- `is_less_than(col, val)`
- `is_less_or_equal_than(col, val)`
- `is_equal_than(col, val)`
- `has_min(col, val)`
- `has_max(col, val)`
- `has_std(col, val)`
- `has_percentile(col, value, percentile, precision, coverage)`
- `is_between(col, i, k)`
- `is_between(col, date_1, date_2)`
- `has_min_by(col2, col1, value)`
- `satisfies(predicate, coverage)`