# Tokopedia Scraping: Tenue de Attire - Part 1
#### by JanendraVian
#### September 26, 2023

## A. Background (Introduction)
The initial intent to start this project was motivated by my motivation to learn more about Python, especially data analysis in Python, and web scraping. This is (I think) an Exploratory Data Analysis project. Throughout the process of finishing this project, I made some mistakes and had questions regarding the chosen data and method, which I will get to more details later.

The data are scraped from Tokopedia, specifically the store Tenue de Attire. They are a fashion/garment industry based and made in Indonesia focusing on menswear and effortless styling in daily usage. I've chosen this brand because, honestly, they're cool. There is no urgency or importance regarding the choosing  of the brand. Even so, based on observations on other brands/stores, Tenue de Attire's product names are the most well-written regarding the "neatness" of the item names, their types/categories, and colors. This came in handy when cleaning the data.
|<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/d0ab0b60-1dc0-4840-b56e-c290fcb7446e" height="150">|
|:--:| 
| *Image 1* |

## B. Tasks
### Guiding Questions
"What are some trends in the product sales?"

## Preparation
### *HUGE DISCLAIMER* 
The data is retrieved from web scraping the Tokopedia site. As such, there are some limitations. The data was scraped on September 25, 2023. The amount of discount the items have and the availability of the items are bound to this date. However, the number of sales is the total of sales since the products came out. This matters because the high number of sales of an item might be affected by the amount of discount on the previous dates. So, because the price is subject to fluctuations, the number of sales is not really accurate. That being said, I fully acknowledge the reliability of the data and proceeded with my analysis.
### About the data
- The data is publicly available through Tokopedia.
- There are 1212 unique products in their Tokopedia catalog.
- The number of sales above the count of 30 is grouped into brackets with increments of 10 (e.g. 30+, 40+, 50+, ....., 100+, 250+).

## Process
### Web Scraping 
Using the website, data is filtered in the "Semua Produk" or All Products category and sorted by the number of sales using the "Urutkan" feature and choosing "Pembelian Terbanyak". Automatically, the website shows the items out of stock on page 4.

I used Python on Jupyterlab to scrape [Tenue de Attire's Tokopedia page](https://www.tokopedia.com/tenuedeattire/product/page/1?sort=8). First, I determined the objects I'd need to do my analysis, which are the product names, their prices, the amount of discount and their original prices (if any), the number of sales, and the ratings of each item. Upon inspecting the HTML elements, the objects needed are contained within the class ```css-974ipl```. 
|<img src="https://github.com/JanendraVian/Tokopedia-Store-Scraping/assets/141770727/bfe28e65-cff2-4dc4-9de8-62ded963db25" height="300">|
|:--:| 
| *Image 2* |

The class for each object needed is listed as:
|Objects|Class|
|----|----|
|name|prd_link-product-name css-3um8ox|
|price|prd_link-product-price css-1ksb19c|
|discount|prd_badge-product-discount css-1qtulwh|
|original price|prd_label-product-slash-price css-1u1z2kp|
|sales|prd_label-integrity css-1duhs3e|
|rating|prd_rating-average-text css-t70v7i|

To retrieve the data, I used the Selenium library and Chrome as its browser. To present the data in Python, I used Pandas and BeautifulSoup.

```python
import time
from selenium import webdriver
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
import pandas as pd
```

Here is how I used the libraries. Each page contained up to 80 products and in order to have a thorough inspection of the page, I made the browser scroll to the bottom of the page with 0.5 seconds of increments between each scroll.
```python
url = 'https://www.tokopedia.com/tenuedeattire/product/page/1?sort=8'
driver = webdriver.Chrome()
driver.get(url)

WebDriverWait(driver,5).until(EC.presence_of_element_located((By.CSS_SELECTOR, '#zeus-root')))
time.sleep(2)

for i in range(20):
    driver.execute_script('window.scrollBy(0,250)')
    time.sleep(0.5)

soup = BeautifulSoup(driver.page_source, 'html.parser')
print(soup)

driver.close()
```
Below is how I retrieved the data from the page. I made a loop so that every product data is retrieved. Notice how I made conditions on the discount, original price, items sold, and rating. This is due to not every item has the objects mentioned. For example, if an item has no discount, the code would return *NULL*. So do the other objects with the same condition. I appended the data and exported them as CSV files.
```python
data = []
for item in soup.findAll('div', class_='css-974ipl'):
    ProductName = item.find('div', class_='prd_link-product-name css-3um8ox').text
    ProductPrice = item.find('div', class_='prd_link-product-price css-1ksb19c').text
    Dsc = item.findAll('div', class_='prd_badge-product-discount css-1qtulwh')
    if len(Dsc) > 0:
        Discount = item.find('div', class_='prd_badge-product-discount css-1qtulwh').text
    else:
        Discount = ''
    OrP = item.findAll('div', class_='prd_label-product-slash-price css-1u1z2kp')
    if len(Dsc) > 0:
        OriginalPrice = item.find('div', class_='prd_label-product-slash-price css-1u1z2kp').text
    else:
        OriginalPrice = ''
    Sold = item.findAll('span', class_='prd_label-integrity css-1duhs3e')
    if len(Sold) > 0:
        ItemsSold = item.find('span', class_='prd_label-integrity css-1duhs3e').text
    else:
        ItemsSold = 0        
    Rtg = item.findAll('span', class_='prd_rating-average-text css-t70v7i')
    if len(Rtg) > 0:
        Rating = item.find('span', class_='prd_rating-average-text css-t70v7i').text
    else:
        Rating = ''
    
    data.append(
        (ProductName, ProductPrice, Discount, OriginalPrice, ItemsSold, Rating)
    )

df=pd.DataFrame(data, columns = ['ProductName', 'ProductPrice', 'Discount', 'OriginalPrice', 'ItemsSold', 'Rating'])
print(df)
df.to_csv('Tenue_1.csv')
```

Now here's how my lack of knowledge limits my efficiency. Because I didn't know how to loop and automate data retrieval on different pages, I had to do that 19 times by changing the page indicator in the URL. Honestly, it wasn't that bad but if I had the chance to learn it, I would.

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

The next part will report on data cleaning and transformation process in Excel and the analysis and visualizations.
