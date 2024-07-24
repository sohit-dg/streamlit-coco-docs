Here is the documentation for the `coco-dashboard` code, built on Streamlit. The documentation includes an overview, installation instructions, environment setup, and a detailed description of the main components and functions in the code.

---

# COCO Dashboard Documentation

## Overview

The COCO Dashboard is a web-based application built using Streamlit. It is designed to visualize various data metrics related to farmer activities, such as unique farmers attending screenings, adoption rates, and video screenings, among others. The dashboard retrieves data from a MySQL database and caches results using Redis to optimize performance.

## Installation

1. **Clone the Repository**
    ```bash
    git clone https://github.com/digitalgreenorg/farmstack-backend
    git checkout dev
    cd streamlit/da_registry
    ```

2. **Install Dependencies**
    ```bash
    after creating & activating Virtual environment inside streamlit/da_registry directory, execute the below command : 
    pip install -r requirements.txt
    ```

## Environment Setup

Create a `.env` file in the root directory of the project and add the following environment variables:

```
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_DB=0

COCO_DB_HOST=<your_mysql_host>
COCO_DB_USER=<your_mysql_user>
COCO_DB_PASSWORD=<your_mysql_password>
COCO_DB_NAME=<your_mysql_db_name>
```

## Usage

To run the dashboard, use the following command:

```bash
streamlit run index.py
```

## Code Components

### Imports and Configuration

The code starts with importing necessary libraries and setting up the page configuration for Streamlit.

```python
from concurrent.futures import ThreadPoolExecutor
import json
import math
import time
import streamlit as st
from dotenv import load_dotenv
import os
import redis
import pandas as pd
from mysql.connector.pooling import MySQLConnectionPool

st.set_page_config(layout="wide", initial_sidebar_state="auto", page_title="COCO Dashboard")
st.title("COCO Dashboard")
```

### Load Environment Variables

Environment variables are loaded using `dotenv`.

```python
load_dotenv()
redis_host = os.getenv("REDIS_HOST", "localhost")
redis_port = int(os.getenv("REDIS_PORT", 6379))
redis_db = int(os.getenv("REDIS_DB", 0))
```

### Redis and MySQL Connection

The code establishes connections to Redis and MySQL.

```python
r = redis.Redis(host=redis_host, port=redis_port, db=redis_db, decode_responses=True)

pool = MySQLConnectionPool(
    pool_name="my_pool",
    pool_size=10,
    host=os.getenv("COCO_DB_HOST"),
    user=os.getenv("COCO_DB_USER"),
    password=os.getenv("COCO_DB_PASSWORD"),
    database=os.getenv("COCO_DB_NAME")
)
```

### Query Functions

Several functions are defined to generate SQL queries based on provided parameters. Examples include `getMasterQuery`, `get_unique_farmers_attended_screenings_query`, and `get_unique_farmers_adopting_practice_query`.

```python
def getMasterQuery(start_date, end_date, country, state, district, block, village, project):
    query = f"""
        SELECT ...
        FROM ...
        WHERE ph.time_created BETWEEN '{start_date}' AND '{end_date}'
    """
    if country != 'none':
        query += f" AND gc.country_name = '{country}'"
    # Additional conditions...
    return query
```

### Fetch Data Function

The `fetch_data` function executes the SQL queries and caches the results in Redis.

```python
def fetch_data(query, params=None):
    query_key = f"{query}:{params}"
    cached_data = r.get(query_key)
    if cached_data:
        return json.loads(cached_data)
    connection = pool.get_connection()
    cursor = connection.cursor()
    cursor.execute(query, params)
    data = cursor.fetchall()
    cursor.close()
    connection.close()
    r.setex(query_key, 86400, json.dumps(data))
    return data
```

### Dropdown Population

The `populate_dropdown` function is used to populate the dropdown menus with data retrieved from the database.

```python
def populate_dropdown(data):
    options = ["none"]
    for item in data:
        options.append(item[0])
    return options
```

### Main Function

The `main` function sets up the Streamlit interface, including date pickers, dropdowns, and calls to fetch data and display results.

```python
def main():
    # Link to external CSS file
    with open("./style.css", "r") as f:
        css = f.read()
    st.markdown(f'<style>{css}</style>', unsafe_allow_html=True)

    # Define layout
    col1, col2, col4, col5, col6, col7 = st.columns([1, 1, 1, 1, 1, 1])
    col8, col9, col10, col11 = st.columns([1, 0.5, 1, 1])
    
    # Date range selection
    start_date = col1.date_input("Start Date", value=pd.to_datetime('2021-03-31'))
    end_date = col2.date_input("End Date", value=pd.to_datetime('2024-03-08'))
    
    # Fetch and display data
    with ThreadPoolExecutor(max_workers=10) as executor:
        futures = [
            executor.submit(fetch_data, getMasterQuery(start_date, end_date, "Ethiopia", selected_state, selected_district, selected_block, selected_village, selected_project)),
            # Additional queries...
        ]
        dashboard_data, farmer_screening_data, adoption_by_farmers_data, ... = [future.result() for future in futures]
    
    # Display results in columns
    with col8:
        st.markdown(f'<div class="card"><div class="title">Unique number of farmers who attended screenings</div><div class="sub-title"><span class="bullet_green">&#8226;</span> {unique_farmers_attended_screenings_query}</div></div>', unsafe_allow_html=True)
    # Additional display sections...
    
    # Chart display
    with col21:
        html_content = f"""
        <div id="main" style="width: 100%; height: 300px;background-color: #f0f0f0;"></div>
        <script src="https://cdn.jsdelivr.net/npm/echarts@5.1.2/dist/echarts.min.js"></script>
        <script type="text/javascript">
            // JavaScript for rendering charts
        </script>
        """
        st.components.v1.html(html_content, height=400)

if __name__ == "__main__":
    main()
```

### Styling

The dashboard includes custom CSS for styling the UI components, which is loaded and applied within the `main` function.

```python
with open("./style.css", "r") as f:
    css = f.read()
st.markdown(f'<style>{css}</style>', unsafe_allow_html=True)
```

### Execution

The `main` function is executed when the script runs, setting up the entire dashboard interface.

```python
if __name__ == "__main__":
    main()
```

---

This documentation should help you understand the structure and functionality of the `coco-dashboard` code. Feel free to ask if you need further details or explanations on specific parts.
