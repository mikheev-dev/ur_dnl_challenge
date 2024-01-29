# Decisions notes

Here I explain some decisions I made during completing this assignment.

Please note, it's only one way of realisation, and in case of real work I'd connect with team to discuss details before implementation.

## Repo structure

I use simple repository with submodules linked to actual implementations of ur_scrapper and ur_service.
It simulates a production ready solution, where we develop service in separated repos.

## Scrapper Architecture

I use multiprocessing and async calls inside process to retrieve the data. In fact, my scrapper is a pipeline

```
[base URParts url] object
  ||
  ||
  Queue for URL stored manufacturers
  ||
  ||
Puller process          ==== Manufacturers objects queue ====   Manufacturer objects saver process
        |_ Gather async Manufacturers parses                       |_ File saver (serialization to csv)
  ||
  ||
  Queue for URL stored categories (and meta from [Manufacturer] level)
  ||
  ||
Puller process           ==== Categories objects queue ====      Category objects saver process
        |_ Gather async Categories parses                           |_ File saver (serialization to csv)
  ||
  ||
  Queue for URL stored models (and meta from [Manufacturer, Category] level)
  ||
  ||
Puller process           ==== Models objects queue ====      Models objects saver process
        |_ Gather async Models parses                               |_ File saver (serialization to csv)
  ||
  ||
  Queue for URL stored parts (and meta from [Category, Models] level)
  ||
  ||
Puller process           ==== Parts objects queue ====      Parts objects saver process
        |_ Gather async Parts parses                                |_ File saver (serialization to csv)


```

Why the structure is so overcomplicated? 
1. There are millions (>4,5mlns) objects to parse from the URParts web-site.
2. Do it using recursive async calls is not a solution, it's hard to limit the number of calls.
3. The URParts site limits the possible number of connections, and the more connects you open, the more response time increases. It also depends on day time, sometimes the response time for only one request was more than 5 seconds)
4. Pipeline structure allows to limit the number of requests in one moment to the source ( and it's possible to adapt it )
5. Multiprocessing is useful to real parallelize CPU intensive operations (f.e handling and saving to files), and allows to scale the steps of pipeline in case of need.
6. It's modular, and is easy to be modified for new data type or for more complex data handling.

## Scrapping to files, not to databases

My scrapper scraps the web-site not to the database directly but to files.
I simulate here separated contours of getting raw data from sources, and pushing prepared data to the database. Of course, it's possible to save raw data to standalone tables.
Also, we could decide to move data to NoSQL database, and having raw data it's easier to do it.

## Scrapper doesn't use ORM to upload data

I don't use ORM to upload data to databases, but upload it using pandas and applying normalisation after it.
* **Note** I understand that uploading data to RAM, it's possible to meet that RAM is not enough. But for this task datasets are small, and this solution is working one.

There are two reasons:
1. Simulation of a case when data pipelines are done independently from services (maybe using not Python at all).
2. Because of the getting data way, data is not normalised. It's faster and more effective to normalise using SQL migrations than manipulating objects in memory.

## Database structure
```
manufacturers                             
-------------                             
id | name                                 
 ^
 |      models                                  parts_models
 |      -------------                           ------------------
  ----- manufacturer_id | name | id <---------- model_id | part_id 
                                                              |
categories                    parts_categories                |             parts
----------                    -------------------             |             --------------------
name | id <------------------ category_id | part_id ----------------------> id | number | spec
```
