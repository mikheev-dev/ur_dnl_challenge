# URParts scrapper and service

Author: Pavel Mikheev https://github.com/mikheev-dev

Task description:
1. Web scraping, data extraction of a static website
Scrape a static website.
Look at https://www.urparts.com/index.cfm/page/catalogue.
Check for a manufacturer, then look into the categories, then the models and the result-set where you find the part number before ‘ - ‘ and the part category.
Scrape the whole catalog from the website and load it into a database.
Containerize the scraper and database.

2. Develop an API
Implement the API as a web service using the FastAPI framework.
Write one or multiple endpoints to expose the data.
Add query-parameters to the endpoint(s), e.g. ?manufacturer=Ammann .
Enable and configure Swagger in FastAPI.
Containerize the web service.



## Setup
* Run 
```git submodule update --init --recursive```

## Run
* Run 
```docker compose up```
* First of all the database is up, and web-server and scrapper wait it.

* Note here that URParts web-site scrapping requires about **7 minutes** to finish. During that time the web-service is **available**, but returns 404 errors with details.

* In case you have no time to wait, you could upload data to database from dumped files. For that in .env set variable 
```UPLOAD_DATA_WITHOUT_SCRAPPING=true```


## Stop
* Run
```docker compose down```

## Swagger
* Open http://0.0.0.0:8000/docs#