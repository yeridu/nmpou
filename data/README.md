# Data

This directory is intentionally empty in the public repository.

## What goes here

The analysis script expects the **2023 national Youth Risk Behavior Survey (YRBS)** Microsoft Access database file to be placed at:

```
data/2023_XXH_YRBS_Data.mdb
```

## Why the file is not redistributed

The YRBS microdata file is publicly available, but the Centers for Disease Control and Prevention (CDC) request that users obtain it directly from their website rather than receiving it from third parties. The Data Sharing Statement in the manuscript points readers to the CDC distribution.

## How to obtain the file

1. Visit the YRBS data page: <https://www.cdc.gov/yrbs/>
2. Navigate to **YRBSS Data & Documentation** and download the **2023 National YRBS Data** in **Microsoft Access (.mdb)** format.
3. Save the file as `2023_XXH_YRBS_Data.mdb` in this directory.

## Reading the file

The script reads the database via the `RODBC` package and the Microsoft Access ODBC driver. On Windows, install the Microsoft Access Database Engine 2016 Redistributable matching the bitness of your R installation (almost always 64-bit). On macOS or Linux, use a Microsoft Access ODBC driver such as the one provided by Actual Technologies, or convert the `.mdb` to a portable format before running.

## What the script does with this directory

The script reads `data/2023_XXH_YRBS_Data.mdb` once at the top of the analysis (Chunk 2) and writes nothing here. All generated outputs land in `output/`, `tables/`, and `figures/`.
