# Binance_ETL_Assignment
The goal of this project is to carry out ETL process end-to-end. We have fetched and extracted data from the Binance website using the Binance API-Key.
The API key and link have been obtained from the Binance website, and the `requests` library has been used to fetch data and extract it.

```
import requests
url = 'https://api.binance.com/api/v3/trades'
params = {
    'symbol': 'BTCUSDT',
    'limit': 100
}
data = requests.get(url, params, timeout=10)
raw_data = data.json()
```
We have transformed the data into a data frame for readability: pd.DataFrame(df)`

After fetching data, we continued to use Python to transform the data. In this case, we just added an ID column and moved it to the far left of the table.

```
df = df.rename(columns={'id':'trade_id'})
df['id'] = range(1, len(df) + 1)
cols = ['id']+[col for col in df.columns if col != 'id']
df = df[cols]
```

After carrying out the transformation, if and where needed, we have to move the data to the PostgreSQL database. This is done by using the psycopg2 library and moving the data to a remote database

```
import psycopg2 as psy

connect = psy.connect(
    host = 'democlass-mbaabuharun8-7833.l.aivencloud.com',
    port= '21586',
    database= 'defaultdb',
    user= 'avnadmin',
    password= 'AVNS_qtmYefsqSkfVKViT5Vk'
)

curs = connect.cursor()

curs.execute('''
    CREATE TABLE IF NOT EXISTS binance_data(
        ID INTEGER,
        Price INTEGER,
        QUANTITY INTEGER,
        QUOTE_QTY INTEGER,
        IS_BUYER_MAKER TEXT)
''')

for index, row in df.iterrows():
    curs.execute(
        '''
        INSERT INTO binance_data(ID,Price,Quantity,Quote_Qty,Is_Buyer_Maker)
            VALUES(%s,%s,%s,%s,%s)
        ''',
            (row['id'],row['price'],row['qty'],row['quoteQty'],row['isBuyerMaker']))
```

Finally, we have to commit all the processes we have taken to push the data into a database

```
connect.commit()

curs.close()

connect.close()
```

### Technologies used
- Python
- PostgreSQL
- Aiven
