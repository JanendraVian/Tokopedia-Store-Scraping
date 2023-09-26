# Tokopedia Scraping: Tenue de Attire - Part 2
#### by JanendraVian
#### September 26, 2023
This is the continuation of [Part 1 of Tokopedia Scraping: Tenue de Attire](https://github.com/JanendraVian/Tokopedia-Store-Scraping/blob/main/README_Part%201.md#tokopedia-scraping-tenue-de-attire---part-1).

### Data Cleaning & Transformation
#### Python
In order to do some data cleaning, I need to merge the 19 CSV files into one. The merged files are then exported as a CSV file. Here's how it's done:
```python
import os
import pandas as pd
from collections import Counter

master_df = pd.DataFrame()
for file in os.listdir(os.getcwd()):
    if file.endswith('.csv'):
        master_df = master_df.append(pd.read_csv(file))
        
master_df.to_csv('TenueAll.csv', index=False)
```

The merged file was then observed and looked for errors and other parts that might need some tweaking. I needed separate columns on the dominant colors of the products, the product types, and the product availability status. Other than that, I needed to do some basic data cleaning like trimming and reformating the prices to numbers, discounts to percentages, and items sold as numbers. 

The first step I did was to determine the dominant color of each product. From observation, some products only have 1 color, some have 2 colors, and some have no information on color. Products with 2 colors use their first mentioned color as the dominant color (e.g. "Blue White" means its dominant color is Blue). Products that don't mention their colors are always printed-art garments, so to determine their colors is unwise. I left it blank. So, I wrote a code that can fetch me the most occurring words in the ```ProductName``` column.
```python
n = 100
DataFrame['ProductName'].value_counts()[:n].index.tolist()
```
I manually sorted through the list and wrote down all the colors. There are 34 distinct colors. In order to automatically determine each product's dominant colors, I ran a code so that the first one to appear in the string would be the dominant color. Then I populate them into a new column.
```python
df['DominantColor'] = df['ProductName'].str.extract('(Navy|White|Black|Grey|Blue|Brown|Green|Olive|Red|Cream|Mustard|Maroon|Khaki|Yellow|Orange|Sage|Terracotta|Charcoal|Mocca|Sand|Turquoise|Salmon|Mint|Burgundy|Tosca|Bronze|Nude|Caramel|Indigo|Brick|Gold|OIive|Turqoise|Army)')
df.to_csv('TenueCompleted.csv')
```
With these, I'm done with Python for this project.

#### Excel
As mentioned in the previous step, I needed to make new columns on the product types and the product availability status. Other than that, I needed to do some basic data cleaning like trimming and reformating the prices to numbers, discounts to percentages, and items sold as numbers.

After I did all that, I decided to categorize the number of sales. Because sales above 30 are not specified and written as, for example, 30+, I decided to drop the + and the number of sales would represent the least number of sales for each product. So 30+ became 30, 40+ became 40, and so on. 

A quick observation on the Tokopedia page indicated that there are 17 product types. To determine the product types, I wrote a formula in the ```ProductTypes``` column as:
```
=IF(ISNUMBER(SEARCH("Case",B2)),"Phone Case",IF(ISNUMBER(SEARCH("Clutch",B2)),"Clutch Bag",IF(ISNUMBER(SEARCH("Hat",B2)),"Hat",IF(ISNUMBER(SEARCH("Pack",B2)),"Hygiene Pack",IF(ISNUMBER(SEARCH("Tote",B2)),"Totebag",IF(ISNUMBER(SEARCH("Totebag",B2)),"Totebag",IF(ISNUMBER(SEARCH("Bag",B2)),"Bag",IF(ISNUMBER(SEARCH("Flannel",B2)),"Plaid/Flannel",IF(ISNUMBER(SEARCH("Polo",B2)),"Polo Shirt",IF(ISNUMBER(SEARCH("Blazer",B2)),"Blazer",IF(ISNUMBER(SEARCH("Cardigan",B2)),"Cardigan",IF(ISNUMBER(SEARCH("Kaos",B2)),"T-Shirt",IF(ISNUMBER(SEARCH("Long",B2)),"Long Sleeve Shirt",IF(ISNUMBER(SEARCH("Celana Pendek",B2)),"Shorts",IF(ISNUMBER(SEARCH("Cargo",B2)),"Cargo Pants",IF(ISNUMBER(SEARCH("Pants",B2)),"Trousers",IF(ISNUMBER(SEARCH("Jacket",B2)),"Jacket","Short Sleeve Shirt")))))))))))))))))
```
|<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/96d807e1-a81a-46ec-a229-8cd04912bd88" height="300">|
|:--:| 
| *Image 1* |

To my surprise, that formula is accurate. 

Products that are not in stock are listed from the 4th page onwards. Manually, I made a new column to categorize products' availability status into "In Stock" and "Out of Stock".

This concludes the processing phase.

## Analysis
### General Findings
There are a total of 1213 unique products in the Tokopedia catalog. Short-sleeve shirts make up 42.8% and Plaid/flannels make up 21.1% of the total products (_Image 2_.) Color-wise, navy makes up the most product color with 13.04% followed by black with 11.88% (_Image 3_.). However, products with colors that are undefined or are printed art collectively make up 12.21% of the total products. 
|<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/dfd3b479-2bd1-49d2-96ab-0de8d6a1e345" width="320">|<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/8d1736de-5618-4db8-b22c-a604f0ac3e42" width="320">|
|:--:|:--:|
| *Image 2* | *Image 3* |

### Sales 
Before proceeding to read this part, [*PLEASE READ THE DISCLAIMER*](https://github.com/JanendraVian/Tokopedia-Store-Scraping/blob/main/README_Part%201.md#huge-disclaimer). The inaccuracy mainly affected price and number of sales. That being said, let's continue. 

The chart shows there are no clear relationships between the products' original price and the number of sales (*Image 4*). Relationships between discounted prices and the number of sales are also not clear (*Image 5*). However, when looking into the relationships between discount percentage and sales, there is a slight upward trend where a higher discount results in a higher number of sales (*Image 6*). The chart doesn't clearly show the distribution because the number of sales is more varied the smaller the numbers are while the higher numbers are grouped, causing overlaps. 
<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/7cad5a21-aadd-4e72-b09e-acc80d867de7" width="320">|<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/ddd32756-32aa-41d8-9b9e-06dfe9f63028" width="320">|<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/82d15bc5-cf98-4047-b320-0702812b64a7" width="320">|
|:--:|:--:|:--:|
| *Image 4* | *Image 5* | *Image 6* |

In the top 10 average items sold by the product types, we can see that T-shirts have the most sales while Short Sleeve Shirts are ranked 5 even though they have the most variety of products (*Image 7*). On the other hand, the top 10 average items sold by dominant colors are led by the color Black followed by Navy (*Image 8*).
|<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/81f6b50d-d459-4991-9ce2-5927db397bc1">|<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/a85250a3-dd85-4ef4-b249-5b7458aaef4f">|
|:--:|:--:|
| *Image 7* | *Image 8* |

Observing the product type by average rating chart shows that the first 4 product types have the perfect average ratings.
|<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/2c429630-0a5e-404d-b4e8-8fea8117f017">|
|:--:| 
| *Image 9* |

Lastly, here is the distribution of sales by product types and dominant colors
|<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/cf092402-4f4c-4f8a-b06d-05ed3a0b0829">|
|:--:| 
| *Image 10* |

## Conclusion
1. More accurate and recent data is needed to make a more accurate analysis and better data-driven decisions.
2. The percentage of discount affects the number of sales.
3. T-shirts are the most sought-after products. 
4. Black is the most desired color.
5. For myself, I need to learn more about Python, especially in Data Analytics and Data Science.

If you have any inquiries, suggestions, or critiques, please contact me through the means listed on my profile. Thank you!
