# test_secrets.py

This document contains the content of the `test_secrets.py` Databricks notebook, converted to Markdown format for readability and sharing.

---

## Imports
```python
import os
```

---

## Databricks Widget
```python
dbutils.widgets.text("Reporting_month", "", "Reporting_month")
```

---

## Get Current Reporting Month
```python
current_reporting_month = dbutils.widgets.get("Reporting_month")
```

---

## Print Reporting Month
```python
print(current_reporting_month)
```

---

## More Imports
```python
from pyspark.sql.functions import *
from datetime import datetime, timedelta
from pyspark.sql.types import *
import numpy as np
from delta.tables import *
import pytz
import json
from datetime import date
from dateutil.relativedelta import relativedelta
```

---

## Data Location Configuration
```python
# change data_location
s3BaseLocation = 's3://dataos-core-prod-team-iibi-iipa/'
data_location =  's3://dataos-core-prod-team-iibi-iipa/data_products/Standard_refresh/'
spark.conf.set("spark.databricks.io.cache.enabled","true")
baseDestLoc = data_location
```

---

## Function: Read Data From Spectrum
```python
def readDataFromSpectrum(query):
  tempS3Dir = s3BaseLocation + 'temp_trans'
  port = '5439'
  spectrum_instance = 'dataos-core-prod-team-iibi.hpdataos.com'
#   "iibi-redshift.hpdataos.com"
  spectrum_url = spectrum_instance + ":" + port
  spectrum_db = "prdii"
  spectrum_user = "srv_rc_reporting"
  spectrum_password = os.environ['REDSHIFT_PW']
  aws_iam_role = 'arn:aws:iam::828361281741:role/redshift-copy-unload-team-iibi'

  # jdbcHostname =f'jdbc:redshift://{spectrum_url}/{spectrum_db}?user={spectrum_user}&password={spectrum_password}&ssl=false'
  jdbcHostname =f'jdbc:redshift://{spectrum_url}/{spectrum_db}?user={spectrum_user}&password={spectrum_password}&ssl=true&sslfactory=com.amazon.redshift.ssl.NonValidatingFactory"'
  #true&sslfactory=com.amazon.redshift.ssl.NonValidatingFactory"'
  
  return  (spark.read.format("com.databricks.spark.redshift") \
          .option("url", jdbcHostname)\
          .option("query",query)\
          .option("aws_iam_role", aws_iam_role)\
          .option("tempdir", tempS3Dir) \
          .load())
```

---

## Query Example
```python
query='''select * from retailer_compensation.overall_summary '''
```

---

## Read Data
```python
final_summary_overall_row=readDataFromSpectrum(query)
```

---

## Display Data
```python
final_summary_overall_row.display()
```
