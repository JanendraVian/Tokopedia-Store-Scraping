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

Now here's how my lack of knowledge limited my efficiency. Because I didn't know how to loop and automate data retrieval on different pages, I had to do that 19 times by changing the page indicator in the URL. Honestly, it wasn't that bad but if I had the chance to learn it, I would.

[The next part](https://github.com/JanendraVian/Tokopedia-Store-Scraping/blob/main/README_Part%202.md#tokopedia-scraping-tenue-de-attire---part-2) will focus on the data cleaning and transformation process in Excel and the analysis and visualizations.
