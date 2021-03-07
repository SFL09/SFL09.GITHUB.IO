---
layout: post
title: "Data Analytics - SSA's Open Data"
description: "Analyzed SSA's open data by using PySpark."
categories: [DataAnalytics]
tags: [DataAnalytics, Python, PySpark]
redirect_from:
  - /2018/03/09/
---

## Data Analytics


### SSA's Open Data
The Social Security Administration (SSA) makes publicly available a unique dataset that contains, by state, the number of male and female babies born in the US that are given each name in each year since 1910 based on Social Security applications. This data can be downloaded from the [SSA website](https://catalog.data.gov/dataset/baby-names-from-social-security-card-applications-state-and-district-of-columbia-data).

I organized all individual state txt files into a CSV file, including the following variables: state, year, gender, name, and the number of babies. Using this data, I found some interesting facts regarding naming patterns over time and across states or regions.

```python
from pyspark.sql.types import StructType
from pyspark.sql.types import StructField
from pyspark.sql.types import StringType
from pyspark.sql.types import IntegerType

schema = StructType([
    StructField("state", StringType(), False),
    StructField("sex", StringType(), False),
    StructField("year", IntegerType(), False),
    StructField("name", StringType(), False),
    StructField("occurrences", IntegerType(), False),
])

data = spark.read.csv("./namesbystate/*.TXT", schema=schema)
data.createOrReplaceTempView("source")
```

```python
from os import listdir
from shutil import copyfile
from shutil import rmtree

def save(dataframe, path):
    tmp_path = f"{path}_"
    dataframe.coalesce(1).write.csv(tmp_path)
    file, = [f for f in listdir(tmp_path) if f.endswith("csv")]
    copyfile(f"{tmp_path}/{file}", path)
    rmtree(tmp_path)

save(data, "names.csv")
```

### 1st Fun Fact
In AZ, AL, and AR, **Ada** became a popular name in 1984, 2003, and 2013.

### 2nd Fun Fact
Alaska tended to adopt **Ada** in 2014 after **Ada** gained popularity in other states.

```python
name = "Ada"
data_name = spark.sql(
    f"""
    SELECT
        state
        , year
        , SUM(occurrences) AS occurrences
    FROM source
    WHERE name = "{name}"
    GROUP BY
        state
        , year
    ORDER BY
        state
        , year
    """
)
```

### 3rd Fun Fact
The following table shows the ten most frequent gender-neutral names for babies.

| Name     | Female | Male | Ratio |
|:--------:|:------:|:----:|:-----:|
| Rei      | 38     | 38   |  0.0  |
| Loghan   | 23     | 23   |  0.0  |
| Gal      | 15     | 15   |  0.0  |
| Dossie   | 10     | 10   |  0.0  |
| Olie     | 6      | 6    |  0.0  |
| Edris    | 6      | 6    |  0.0  |
| Jaidin   | 6      | 6    |  0.0  |
| Kahne    | 5      | 5    |  0.0  |
| Yihan    | 5      | 5    |  0.0  |
| Asuncion | 5      | 5    |  0.0  |

```python
stat = spark.sql(
    """
    SELECT
        name
        , SUM(CASE WHEN sex = 'F' THEN occurrences ELSE 0 END) AS F
        , SUM(CASE WHEN sex = 'M' THEN occurrences ELSE 0 END) AS M
    FROM source
    GROUP BY name
    """
)
stat.createOrReplaceTempView("stat")

ratio = spark.sql("""
    SELECT
        *
        , ABS(M - F) / (M + F) AS ratio
    FROM stat
    ORDER BY 
        ratio ASC
        , M + F DESC
    """
)
```
