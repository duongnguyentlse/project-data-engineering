# 💰 Project: Capitalisation boursière des plus grandes banques - ETL & Conversion multi-devises

## Présentation
Ce projet consiste à créer un processus **ETL** pour :
- Extraire les **10 plus grandes banques mondiales** classées par **capitalisation boursière (en milliards USD)**.
- Convertir ces données en plusieurs devises : **USD, GBP, EUR, INR** à partir d'un fichier CSV de taux de change.
- Sauvegarder les résultats dans un fichier CSV localement et dans une **base de données SQL**.
- Permettre à des gestionnaires de différents pays de **consulter la capitalisation boursière dans leur propre devise** via des requêtes SQL.

---

## Objectifs
- Écrire une fonction d’extraction des données à partir de l’URL cible.  
- Transformer la capitalisation boursière de million à milliard USD.  
- Convertir les données vers d’autres devises à partir des taux fournis.  
- Charger les données dans un **fichier CSV** et dans une **base de données SQLite**.  
- Interroger la base de données pour obtenir les informations demandées.  
- Enregistrer les étapes du traitement avec des **horodatages**.

---

## Installation des dépendances

Avant d’exécuter le projet, installez les bibliothèques nécessaires :

```bash
pip install pandas
pip install beautifulsoup4


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
https://github.com/danielngtlse/project-data-engineering/issues/1













