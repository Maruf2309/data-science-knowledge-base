
# Clean Pipeline
### 1.1 Cleaing Pipeline
```bash
import pandas as pd
from pandas import DataFrame

def clean(df: DataFrame) -> pd.DataFrame:
    """Return cleaned copy of DataFrame."""
    df = df.copy()

    ## stage1: remove suffixes like 'Sales Center', 'FSC Point', 'DB Point'
    if "Salespoint" in df.columns:
        df["Salespoint"] = (
            df["Salespoint"]
            .astype(str)
            .str.replace(r"\s*(Sales Center|FSC Point|DB Point)\s*$", "", regex=True)
            .str.strip()
        )

    ## Coloumns reformations
    df.columns = [
        col.lower().strip().replace(" ", "_")
        for col in df.columns
    ]

    ## Fill missing values in territory with forward fill (Excel Merged File)
    if "territory" in df.columns:
        df["territory"] = df["territory"].ffill()

    ## Fill missing values in numerical columns with 0
    num_cols = df.select_dtypes(include="number").columns
    df[num_cols] = df[num_cols].fillna(0)

    ## Convert specific column safely (round → int32)
    if "no_of_weighty_party" in df.columns:
        df["no_of_weighty_party"] = (
            df["no_of_weighty_party"]
            .round()  # round up is save for float to int32
            .astype("int32")
        )

    ## Memory optimization: exclude 'fan' and 'super'
    exclude_cols = {"fan", "super"}
    for col in df.select_dtypes(include="int64").columns:
        if col not in exclude_cols:
            df[col] = df[col].astype("int32")

    return df