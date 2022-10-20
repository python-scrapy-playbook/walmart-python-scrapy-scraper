# walmart-python-scrapy-scraper
Python Scrapy spider that scrapes product data from [Walmart.com](https://www.walmart.com/). 

These scrapers extract the following fields from Walmart product pages:

- Product Type
- Product Name
- Brand
- Average Rating
- Manufacturer Name
- Description
- Image Url
- Price
- Currency
- Etc.

The following article goes through in detail how this Walmart spider was developed, which you can use to understand the spiders and edit them for your own use case.

[Python Scrapy: Build A Walmart.com Scraper](https://scrapeops.io/python-scrapy-playbook/python-scrapy-walmart-scraper/)

## ScrapeOps Proxy
This Walmart spider uses [ScrapeOps Proxy](https://scrapeops.io/proxy-aggregator/) as the proxy solution. ScrapeOps has a free plan that allows you to make up to 1,000 requests per month which makes it ideal for the development phase, but can be easily scaled up to millions of pages per month if needs be.

You can [sign up for a free API key here](https://scrapeops.io/app/register/main).

To use the ScrapeOps Proxy you need to first install the proxy middleware:

```python

pip install scrapeops-scrapy-proxy-sdk

```

Then activate the ScrapeOps Proxy by adding your API key to the `SCRAPEOPS_API_KEY` in the ``settings.py`` file.

```python

SCRAPEOPS_API_KEY = 'YOUR_API_KEY'

SCRAPEOPS_PROXY_ENABLED = True

DOWNLOADER_MIDDLEWARES = {
    'scrapeops_scrapy_proxy_sdk.scrapeops_scrapy_proxy_sdk.ScrapeOpsScrapyProxySdk': 725,
}

```


## ScrapeOps Monitoring
To monitor our scraper, this spider uses the [ScrapeOps Monitor](https://scrapeops.io/monitoring-scheduling/), a free monitoring tool specifically designed for web scraping. 

**Live demo here:** [ScrapeOps Demo](https://scrapeops.io/app/login/demo) 

![ScrapeOps Dashboard](https://scrapeops.io/assets/images/scrapeops-promo-286a59166d9f41db1c195f619aa36a06.png)

To use the ScrapeOps Proxy you need to first install the monitoring SDK:

```

pip install scrapeops-scrapy

```


Then activate the ScrapeOps Proxy by adding your API key to the `SCRAPEOPS_API_KEY` in the ``settings.py`` file.

```python

SCRAPEOPS_API_KEY = 'YOUR_API_KEY'

# Add In The ScrapeOps Monitoring Extension
EXTENSIONS = {
'scrapeops_scrapy.extension.ScrapeOpsMonitor': 500, 
}


DOWNLOADER_MIDDLEWARES = {

    ## ScrapeOps Monitor
    'scrapeops_scrapy.middleware.retry.RetryMiddleware': 550,
    'scrapy.downloadermiddlewares.retry.RetryMiddleware': None,
    
    ## Proxy Middleware
    'scrapeops_scrapy_proxy_sdk.scrapeops_scrapy_proxy_sdk.ScrapeOpsScrapyProxySdk': 725,
}

```

If you are using both the ScrapeOps Proxy & Monitoring then you just need to enter the API key once.


## Running The Scrapers
Make sure Scrapy and the ScrapeOps Monitor is installed:

```

pip install scrapy scrapeops-scrapy

```

To run the Walmart spiders you should first set the search query parameters you want to search by updating the `keyword_list` list in the spiders:

```python

def start_requests(self):
    keyword_list = ['laptop']
    for keyword in keyword_list:
        payload = {'q': keyword, 'sort': 'best_seller', 'page': 1, 'affinityOverride': 'default'}
        walmart_search_url = 'https://www.walmart.com/search?' + urlencode(payload)
        yield scrapy.Request(url=walmart_search_url, callback=self.parse_search_results, meta={'keyword': keyword, 'page': 1})

```

Then to run the spider, enter one of the following command:

```

scrapy crawl walmart

```


## Customizing The Walmart Scraper
The following are instructions on how to modify the Walmart scraper for your particular use case.

Check out this [guide to building a Walmart.com Scrapy spider](https://scrapeops.io/python-scrapy-playbook/python-scrapy-walmart-scraper/) if you need any more information.

### Configuring Walmart Product Search
To change the query parameters for the product search just change the keywords and locations in the `keyword_list` lists in the spider.

For example:

```python

def start_requests(self):
    keyword_list = ['laptop', 'ipad', '']
    for keyword in keyword_list:
        payload = {'q': keyword, 'sort': 'best_seller', 'page': 1, 'affinityOverride': 'default'}
        walmart_search_url = 'https://www.walmart.com/search?' + urlencode(payload)
        yield scrapy.Request(url=walmart_search_url, callback=self.parse_search_results, meta={'keyword': keyword, 'page': 1})

```

You can also change the sorting algorithm to one of: ``best_seller``, `best_match`, `price_low` or `price_high`.

### Extract More/Different Data
The JSON blobs the spiders extract the product data from are pretty big so the spiders are configured to only parse some of the data. 

You can expand or change the data that gets extract by changing the yield statements:

```python

yield {
        'keyword': response.meta['keyword'],
        'page': response.meta['page'],
        'position': response.meta['position'],
        'id':  raw_product_data.get('id'),
        'type':  raw_product_data.get('type'),
        'name':  raw_product_data.get('name'),
        'brand':  raw_product_data.get('brand'),
        'averageRating':  raw_product_data.get('averageRating'),
        'manufacturerName':  raw_product_data.get('manufacturerName'),
        'shortDescription':  raw_product_data.get('shortDescription'),
        'thumbnailUrl':  raw_product_data['imageInfo'].get('thumbnailUrl'),
        'price':  raw_product_data['priceInfo']['currentPrice'].get('price'), 
        'currencyUnit':  raw_product_data['priceInfo']['currentPrice'].get('currencyUnit'),  
    }

```

### Speeding Up The Crawl
The spiders are set to only use 1 concurrent thread in the settings.py file as the ScrapeOps Free Proxy Plan only gives you 1 concurrent thread.

However, if you upgrade to a paid ScrapeOps Proxy plan you will have more concurrent threads. Then you can increase the concurrency limit in your scraper by updating the `CONCURRENT_REQUESTS` value in your ``settings.py`` file.

```python
# settings.py

CONCURRENT_REQUESTS = 10

```

### Storing Data
The spiders are set to save the scraped data into a CSV file and store it in a data folder using [Scrapy's Feed Export functionality](https://docs.scrapy.org/en/latest/topics/feed-exports.html).

```python

custom_settings = {
        'FEEDS': { 'data/%(name)s_%(time)s.csv': { 'format': 'csv',}}
        }

```

If you would like to save your CSV files to a AWS S3 bucket then check out our [Saving CSV/JSON Files to Amazon AWS S3 Bucket guide here](https://scrapeops.io//python-scrapy-playbook/scrapy-save-aws-s3)

Or if you would like to save your data to another type of database then be sure to check out these guides:

- [Saving Data to JSON](https://scrapeops.io/python-scrapy-playbook/scrapy-save-json-files)
- [Saving Data to SQLite Database](https://scrapeops.io/python-scrapy-playbook/scrapy-save-data-sqlite)
- [Saving Data to MySQL Database](https://scrapeops.io/python-scrapy-playbook/scrapy-save-data-mysql)
- [Saving Data to Postgres Database](https://scrapeops.io/python-scrapy-playbook/scrapy-save-data-postgres)

### Deactivating ScrapeOps Proxy & Monitor
To deactivate the ScrapeOps Proxy & Monitor simply comment out the follow code in your `settings.py` file:

```python
# settings.py

# ## Enable ScrapeOps Proxy
# SCRAPEOPS_PROXY_ENABLED = True

# # Add In The ScrapeOps Monitoring Extension
# EXTENSIONS = {
# 'scrapeops_scrapy.extension.ScrapeOpsMonitor': 500, 
# }


# DOWNLOADER_MIDDLEWARES = {

#     ## ScrapeOps Monitor
#     'scrapeops_scrapy.middleware.retry.RetryMiddleware': 550,
#     'scrapy.downloadermiddlewares.retry.RetryMiddleware': None,
    
#     ## Proxy Middleware
#     'walmart_scraper.middlewares.ScrapeOpsProxyMiddleware': 725,
# }

```

