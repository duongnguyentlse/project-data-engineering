# Project Overview 
Ce projet vous oblige à dresser la liste des 10 plus grandes banques du monde classées par capitalisation boursière en milliards USD. De plus, vous devez transformer les données et les stocker en USD, GBP, EUR et INR selon les informations sur le taux de change mises à votre disposition sous forme de fichier CSV. Vous devez enregistrer le tableau d'informations traitées localement au format CSV et en tant que tableau de base de données. Les gestionnaires de différents pays interrogeront la table de la base de données pour extraire la liste et noter la valeur de la capitalisation boursière dans leur propre devise.



# Objectives📝
* Vous devez effectuer les tâches suivantes pour ce projet
   - Écrivez une fonction d'extraction de données pour récupérer les informations pertinentes à partir de l'URL requise.
   - Transformez les informations disponibles sur le PIB en « Milliard USD » à partir de « Million USD ».
   - Chargez les informations transformées dans le fichier CSV requis et en tant que fichier de base de données.
   - Exécutez la requête requise sur la base de données.
   - Enregistrez la progression du code avec des horodatages appropriés.

## 📦 Install

- Tout d'abord, installez les bibliothèques requises pour ETL :

```bash
pip3 install pandas
```

```bash
pip3 install beautifulsoup4
```

## Implementation
- Après l'installation, nous parcourons le code pour discuter brièvement de la progression

- ensuite, il faut initialiser nos librairies :
```python
import pandas as pd 
import numpy as np 
import requests
import sqlite3
from bs4 import BeautifulSoup
from datetime import datetime
```  

## Directions 🗺
1. Cette fonction extrait les informations tabulaires de l'URL donnée sous la rubrique By Market Capitalization à l'aide de bs4 et les enregistre dans un bloc de données.
```python
def extract(url, table_attribs):
    df = pd.DataFrame(columns = table_attribs)

    page = requests.get(url).text
    data = BeautifulSoup(page, 'html.parser')

    tables = data.find_all('tbody')[0]
    rows = tables.find_all('tr')

    for row in rows:
        col = row.find_all('td')
        if len(col) != 0:
            ancher_data = col[1].find_all('a')[1]
            if ancher_data is not None:
                data_dict = {
                    'Name': ancher_data.contents[0],
                    'MC_USD_Billion': col[2].contents[0]
                }
                df1 = pd.DataFrame(data_dict, index = [0])
                df = pd.concat([df, df1], ignore_index = True)

    USD_list = list(df['MC_USD_Billion'])
    USD_list = [float(''.join(x.split('\n'))) for x in USD_list]
    df['MC_USD_Billion'] = USD_list

    return df
```

2. Cette fonction transforme le bloc de données en ajoutant des colonnes pour Market Capitalization en GBP, EUR et INR, arrondies à 2 décimales, en fonction des informations sur le taux de change partagées sous forme de fichier CSV.
```python
def transform(df, exchange_rate_path):
    csvfile = pd.read_csv(exchange_rate_path)

    # i made here the content for currenct is the keys and the content of 
    # the rate is the values to the crossponding keys
    dict = csvfile.set_index('Currency').to_dict()['Rate']

    df['MC_GBP_Billion'] = [np.round(x * dict['GBP'],2) for x in df['MC_USD_Billion']]
    df['MC_INR_Billion'] = [np.round(x * dict['INR'],2) for x in df['MC_USD_Billion']]
    df['MC_EUR_Billion'] = [np.round(x * dict['EUR'],2) for x in df['MC_USD_Billion']]

    return df
```

3. Cette fonction charge la trame de données transformée dans un fichier CSV de sortie.
```python
def load_to_csv(df, output_path):
    df.to_csv(output_path)
```

4. Cette fonction charge la data frame transformée sur un serveur de base de données SQL sous forme de table.
```python
def load_to_db(df, sql_connection, table_name):
    df.to_sql(table_name, sql_connection, if_exists = 'replace', index = False)
```

5. Cette fonction exécute des requêtes sur la table de la base de données.
```python
def run_query(query_statements, sql_connection):
    for query in query_statements:
        print(query)
        print(pd.read_sql(query, sql_connection), '\n')
```

6. Cette fonction enregistre la progression du code.
```python
def log_progress(msg):
    timeformat = '%Y-%h-%d-%H:%M:%S'
    now = datetime.now()
    timestamp = now.strftime(timeformat)

    with open(logfile, 'a') as f:
        f.write(timestamp + ' : ' + msg + '\n')
```

7. Ici, vous définissez les entités requises et appelez les fonctions pertinentes dans le bon ordre pour terminer le projet. Notez que cette partie ne se trouve dans aucune fonction.
```python
url = 'https://web.archive.org/web/20230908091635/https://en.wikipedia.org/wiki/List_of_largest_banks'
exchange_rate_path = 'exchange_rate.csv'

table_attribs = ['Name', 'MC_USD_Billion']
db_name = 'Banks.db'
table_name = 'Largest_banks'
conn = sqlite3.connect(db_name)
query_statements = [
        'SELECT * FROM Largest_banks',
        'SELECT AVG(MC_GBP_Billion) FROM Largest_banks',
        'SELECT Name from Largest_banks LIMIT 5'
    ]

logfile = 'code_log.txt'
output_csv_path = 'Largest_banks_data.csv'

log_progress('Preliminaries complete. Initiating ETL process.')

df = extract(url, table_attribs)
log_progress('Data extraction complete. Initiating Transformation process.')


df = transform(df, exchange_rate_path)
log_progress('Data transformation complete. Initiating loading process.')

load_to_csv(df, output_csv_path)
log_progress('Data saved to CSV file.')

log_progress('SQL Connection initiated.')


load_to_db(df, conn, table_name)
log_progress('Data loaded to Database as table. Running the query.')

run_query(query_statements, conn)
conn.close()
log_progress('Process Complete.')
```

## Result Snapshots 📸
![Screen Shot 2024-03-22 at 11 42 54 PM](https://private-user-images.githubusercontent.com/110859185/315985267-7e2f00d5-46c1-4e9e-9636-12f829e2a1a7.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MTExMDYxMDIsIm5iZiI6MTcxMTEwNTgwMiwicGF0aCI6Ii8xMTA4NTkxODUvMzE1OTg1MjY3LTdlMmYwMGQ1LTQ2YzEtNGU5ZS05NjM2LTEyZjgyOWUyYTFhNy5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjQwMzIyJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI0MDMyMlQxMTEwMDJaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT00OTE5ZDhkOTY4MzkzNWFkMWZhOGRmMzlhZWY5N2ViNjkwZTNlMmE2YmFkNzlkNjY4MDEzYjFmMDIyZTVjMzA5JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.jzspFP04gthkHHV0YtsmEocD5hSQDVWLgu8S8cS1P4g)
![Screen Shot 2024-03-22 at 11 52 37 PM]([https://github.com/danielngtlse/project-data-engineering/issues/1#issuecomment-2014855160](https://private-user-images.githubusercontent.com/110859185/315986651-9f8e3618-86a5-4852-88dd-a73450574df0.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MTExMDYxMDIsIm5iZiI6MTcxMTEwNTgwMiwicGF0aCI6Ii8xMTA4NTkxODUvMzE1OTg2NjUxLTlmOGUzNjE4LTg2YTUtNDg1Mi04OGRkLWE3MzQ1MDU3NGRmMC5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjQwMzIyJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI0MDMyMlQxMTEwMDJaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT0wM2QzMzMzM2E4ZjA3OTM5ZjYxNTdhMmZhNTgzYjIyOTFjN2NjNjhmZjk4YjU2YWI4ZTE2ZDQxMDViODNlOWMxJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.ZCGHUR_OswVqKvpwRkGlKT85rxyhU2aN0LKhqT5HwEQ)https://private-user-images.githubusercontent.com/110859185/315986651-9f8e3618-86a5-4852-88dd-a73450574df0.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MTExMDYxMDIsIm5iZiI6MTcxMTEwNTgwMiwicGF0aCI6Ii8xMTA4NTkxODUvMzE1OTg2NjUxLTlmOGUzNjE4LTg2YTUtNDg1Mi04OGRkLWE3MzQ1MDU3NGRmMC5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBVkNPRFlMU0E1M1BRSzRaQSUyRjIwMjQwMzIyJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDI0MDMyMlQxMTEwMDJaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT0wM2QzMzMzM2E4ZjA3OTM5ZjYxNTdhMmZhNTgzYjIyOTFjN2NjNjhmZjk4YjU2YWI4ZTE2ZDQxMDViODNlOWMxJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.ZCGHUR_OswVqKvpwRkGlKT85rxyhU2aN0LKhqT5HwEQ)















