# Big Data Midterm — Anush Khachatryan

## Dataset
[Instacart Market Basket Analysis](https://www.kaggle.com/datasets/psparks/instacart-market-basket-analysis/data)

## Project Structure
```
midterm/
├── Anush_Khachatryan_Big_Data_Midterm.ipynb   # Main notebook
├── requirements.txt                            # Dependencies
├── README.md                                   # This file
└── data/                                       # Dataset files
    ├── orders.csv
    ├── products.csv
    ├── order_products__prior.csv
    ├── order_products__train.csv
    ├── aisles.csv
    └── departments.csv
```

## Requirements
```
pyspark==4.1.1
pandas~=3.0.2
matplotlib~=3.10.9
```

Install dependencies:
```bash
pip install -r requirements.txt
```

## How to Run
1. Clone the repository
2. Place the dataset files inside the `data/` folder
3. Open `Anush_Khachatryan_Big_Data_Midterm.ipynb` in Jupyter
4. Run all cells (Kernel → Restart & Run All)

## Sections
1. **Data Quality Analysis** — Detecting and explaining non-obvious data issues
2. **Data Manipulation, Joins, and Cleaning** — Joins, normalization, top-3 products per user
3. **Spark Job Analysis and Optimization** — Execution plan analysis, bottleneck identification, optimized pipeline (~5x speedup)
4. **Analytics Report and Visualization** — Key insights and 5 visualizations
5. **Bias, Limitations, and Scalability Discussion** — Data quality impact and 100x scalability analysis
