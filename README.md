# Master product catalog creation pipeline

## Description

This Python pipeline cleans and aggregates large datasets of software product names along with
metadata such as seller's website URL, product category and product description. Duplicated
products (with slightly different names) are then identified using fuzzy string matching and
removed from the dataset to form a master product catalog of unique products.

## Goals

- Aggregate software data from multiple sources
- Clean and normalize software names and metadata
- Identify and remove duplicate products (e.g., 'Amazon SES' vs 'Amazon Simple Email Service (SES)')
- Prepare a high-quality dataset for downstream use

## Project structure

- `product_catalogue_creation.ipynb` Jupyter Notebook
  - Pipeline for aggregating, cleaning and deduplicating software data
- `input` directory containing CSV inputs
- `output` directory containing pipeline outputs

## Requirements

- `pandas 2.2.3`
- `rapidfuzz 3.13.0`

## How to use

- Add CSV files to `input` directory. The following columns must be present:
  - `product_name`
  - `seller_website`
  - `description`
  - `product_category`
- NB: Other columns may be present but they will be discarded by the pipeline.
- Set duplicate thresholds for similarity scores between products
  - Scores above the thresholds indicate duplicated products
- Click 'Run All' to run all cells in the `product_catalogue_creation.ipynb` Jupyter Notebook
- Find outputs in `output` directory:
  - `duplicates.csv`
    - The pairs of product identified as duplicates by fuzzy matching
  - `duplicated_product_catalogue.csv`
    - The list of products aggregated from all data sources BEFORE deduplication
  - `master_product_catalogue.csv`
    - The list of products aggregated from all data sources AFTER deduplication
  - `scores.csv`
    - All the pairs of products and their fuzzy matching similarity scores

## Overview of pipeline steps

- Required packages are installed and imported
- CSV files are read from `input` directory into Pandas DataFrames
- Column names are standardised if required
- Unused columns are removed
- All data frames are concatenated into one
- Product names, descriptions, categories and website URLs are cleaned
- Exact duplicates are removed (based on product name and website URLs separately)
- Products are sorted alphabetically
- Each product is compared to n products above and n products below in the ordered list using fuzzy matching
  - Similarity scores (0-100) are computed for each of the following fields using fuzzy matching:
      - Product name 
      - Seller's website URL 
      - Product description 
      - Product category
- Pairs of similar products are identified based on similarity scores exceeding set thresholds
- Duplicated products are removed from the dataset
- Outputs are written to `output` directory

## Limitations

- Fuzzy matching is limited to identify duplicates, especially with acronyms and short strings:
  - Similarity score between 'Amazon SES' and 'Amazon Simple Email Service' is only 47
  - Similarity score between '.NET' and '.NET 4.5' is only 67
- Fuzzy matching can identify different products as duplicates, especially with short strings:
  - Similarity score between 'aBILLity' and 'Ability' is very high but they're different products
- Website URL data is not always available and can be subject to the same issues
  - Similarity score between abillity.com and ability.com is very high
- Categories and description are not standardised across data sources so vary in wording
  - Methods that can understand semantics such as LLMs are required to identify similarities in categories and descriptions

## Further improvements

- Use additional variables (e.g. headquarters) to reinforce confidence when identifying duplicates
  - Downside: Likely to have missing values and not present in all data sources
- Use LLMs to identify semantic similarities in product categories and descriptions
- Use a similarity calculation that puts more weight on the first few characters being identical
  - E.g. '.NET' and '.NET 4.5' get a higher score because the start is the same with only a variation at the end
- Incorporate name and URL lengths into similarity calculations to prevent inaccurate results with short strings
- Incorporate manually compiled list of similar names for most popular products
  - Downside: Needs to be kept updated over time
- Storing all pairs and their scores in a Pandas DataFrame (even when the similarity scores are low) is not ideal for efficiency and scalability
  - This was done so that results would be easier to analyse and present
  - In production, only keep the potentially duplicated pairs in memory
