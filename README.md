 
````markdown
# PySpark Data Cleaning – Population by Age Dataset

This project demonstrates how to clean and transform a messy `.tsv` dataset using **PySpark** in Google Colab. It includes column renaming, string parsing, cleaning of year-wise data, type conversion, and saving the cleaned output as a zipped CSV file.

---

##  Tools & Technologies

- Python
- PySpark
- Google Colab
- TSV file (`population_by_age.tsv.txt`)

---

##  Dataset

The raw dataset `population_by_age.tsv.txt` contains messy column names, combined fields (e.g., indicator and country), and year-wise data with special characters (e.g., `:` or `e`).

---

##  Data Cleaning Steps

### 1. Install PySpark
```python
!pip install pyspark
````

### 2. Import Libraries

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, split, regexp_replace, isnan, when, count
```

### 3. Start Spark Session

```python
spark = SparkSession.builder.appName("DataCleaning").getOrCreate()
```

### 4. Read TSV File

```python
df = spark.read.option("delimiter", "\t").option("header", True).csv("population_by_age.tsv.txt")
df.show(5)
```

### 5. Rename Messy Column

```python
df = df.withColumnRenamed("indic_de,geo\\time", "indicator_country")
```

### 6. Split Column into `indicator` and `country`

```python
df = df.withColumn("indicator", split(col("indicator_country"), ",").getItem(0))
df = df.withColumn("country", split(col("indicator_country"), ",").getItem(1))
df = df.drop("indicator_country")
```

### 7. Trim Column Names

```python
for colname in df.columns:
    df = df.withColumnRenamed(colname, colname.strip())
```

### 8. Clean Year Columns (Remove non-numeric characters)

```python
year_columns = ['2008', '2009', '2010', '2011', '2012', '2013', '2014',
                '2015', '2016', '2017', '2018', '2019']

for year in year_columns:
    df = df.withColumn(year, regexp_replace(col(year), "[^0-9.]", ""))
```

### 9. Cast Year Columns to Float

```python
for year in year_columns:
    df = df.withColumn(year, col(year).cast("float"))
```

### 10. Check for Null Values

```python
df.select([count(when(col(c).isNull(), c)).alias(c) for c in year_columns]).show()
```

### 11. Fill Nulls with Zero

```python
df = df.fillna(0)
```

### 12. Show Cleaned Data

```python
df.select("indicator", "country", "2008", "2009", "2010").show(10)
```

---

##  Save & Download Cleaned Data

### Save as CSV (in a folder)

```python
df.write.mode("overwrite").option("header", True).csv("cleaned_population")
```

### Zip the Folder

```python
import shutil
shutil.make_archive("cleaned_population", 'zip', "cleaned_population")
```

### Download the Zipped Folder in Colab

```python
from google.colab import files
files.download("cleaned_population.zip")
```

---

##  Output

* Cleaned CSV files inside `cleaned_population/`
* Zipped archive: `cleaned_population.zip`

---

##  Learning Outcomes

* Handling messy column names and delimiters.
* Splitting and renaming columns.
* Cleaning and converting data types using PySpark.
* Saving and downloading data from Google Colab.

---

##  Folder Structure

```
.
├── messy_data.py
├── population_by_age.tsv.txt
├── cleaned_population/
│   └── part-*.csv
├── cleaned_population.zip
└── README.md
```

---
 
