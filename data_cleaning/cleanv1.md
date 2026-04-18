# Data Cleaning
## Clean the data
```bash
* Handle Missing Values
df['mail'].fillna("Unknown")

# Fix Dtypes
df['date'] = pd.to_datetime(
    df['date'])

df['price'] = (
    df['price']
    .str.replace('$','')
    .astype(float))

# Remove Duplicate
df = df.drop_duplicates(
    subject = ['order_id'],
    keep='last')

print(f"clean: {df.shape})
```