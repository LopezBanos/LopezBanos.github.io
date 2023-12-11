---
title: Planet Python API - The handy way of connecting with it'
date: 2023-12-11
permalink: /posts/2012/08/blog-post-1/
bibliography: references.bib  
tags:
  - Python
  - Planet
  - API
---
# What is Planet Python API?
Planet Python API allows a user to access satellite data, work with it and 
code data pipelines. I found it quite interesting if you want to download a 
large set of images according to some fixed criteria which is a common task 
when working with satellite data. There is also a command line interface (CLI) 
to work with it, but it does not have the power a programming language has.
<br><br>
Planet Python API has two different approaches in its examples which correspond
with different versions (or styles) of planet, the _old-style_ and the 
_new-style_. Long story short, the new style
uses python functions to define the filters.
```python
    product         = [order_request.product(item_ids, bundle, item_type)]

    request         = order_request.build_request(name     = request_name.replace('.json',''),
                                                 products  = product,
                                                 delivery  = delivery,
                                                 tools     = tools)
```
In contrast, the old-style needs specific dictionaries to create the filters.
```python
{
  "item_types"     : ['PSScene'],
  "filter"         : {
                       "type"         : "AndFilter",
                       "config"       : [
                         "type"       : "RangeFilter",
                         "field_name" : "cloud_cover",
                         "config"     : {"lte"      : 0.5}
                                        ]
                     }
}
```
Unfortunately, not all the examples are rewritten using the 
new-style. For this reason, the examples of the documentation (nowadays) are a
bit confusing since you cannot reproduce all of them with the new-style, you
have to re-write some of them. 
## Handy Module comes to save time
The reason to publish this script is simple, it saves a lot of time if you want
to download a set of images given a filter up to some AOI (Area of Interest) 
stored in a `.json` file. This becomes more interesting if you are working with 
OpenStreetMap data since you can get the `.json` properties using its API.
<br><br>
The module consists of two parts, a custom filter utility and a handy search and
order request utility. Both scripts ease the writing of the code.


# Installation
To download the script clone the repository 
```bash
git clone https://github.com/LopezBanos/HandyPlanetAPI.git
```
To install the required packages 
```bash
pip install -r requirements.txt
```
# Usage
Copy the files `__init__.py`, `planetapi.py` and `utils` folder into the folder
where your `**.json` are stored, **src** directory. The tree folder has to look 
similar to
```bash
├── src
│   ├── utils
│   │   ├── authentication.py
│   │   ├── custom_filter.py
│   │   ├── directory.py
│   │   ├── geometry.py
│   │   └── request.py
│   ├── __init__.py
│   ├── planetapi.py
│   └── **.json
```
where `**.json` represents all the `.json` in the **src** folder. 
## Inserting Credentials
1. On your planet account you can find a token in the settings menu. <br> 
2. Open `planetapi.py`.
3. Copy and paste that token in `API_KEY ='INSERT YOUR API KEY HERE'` in the `planetapi.py` script.

## Modifying the custom filter
The `utils/custom_filter.py` uses the new style of creating filters
(`and_filter`, `range_filter`, `date_range_filter` and `string_in_filter`)
with a geometry filter that is generated with the `.json` files that come from
OpenStreetMap. <br><br>
**Warning:** *If your `.json` files do not come from OSM they might have other formatting and you must do some workaround.*

## Handy Search and Order Request
Inside the `utils/request.py` there are two main functions. 

### Handy Search Request
This function returns the *items_ids* that match our filter and AOI.
```python
handy_search_request(API_KEY, ITEM_TYPES, filter):
```
- **API_KEY:** Account Token from Planet Website.
- **ITEM_TYPE:** Collection or Scene from the Planet where we want to get the images from. 
- **filter:** Custom filter created with `custom_filter.py`. <br>

### Handy Order Request
This function activates and builds the request. In other words, it creates a 
dictionary that keeps every *item_ids* we are interested in and the AOI of 
interest of that *item_id* image. 
```python
handy_order_request(request_name, 
                    item_type, 
                    item_ids, 
                    bundle, 
                    delivery, 
                    tools):
```

- **request_name:** The name which is going to appear on the Planet website.
- **item_ids:** The item_ids we got in the search request. 
- **bundle:** Choose among the bundles Planet offers.
- **delivery:** If you want to download the assets automatically, you can modify the delivery dictionary. Currently, the images are downloaded and stored as `.zip` files.
- **tools:** Clip to AOI tool so that we get just the area we are interested in. <br><br>
**Warning:** *The clipping tool gives a true output if the area intersects 
with our item_id image, in other words, you may get just a single pixel and not 
the whole area of coverage. The reason behind this is that it has not been implemented
yet in the Python Planet API.*

## Issues and Ordered Folders
When downloading and requesting images in the Planet API, it is common to find 
an exception either because the image for that AOI is not available with the 
given filter or the `.json` geometry file is corrupted (having just a single 
point instead of an area). For large data pipelines, this can be a problem 
since it will stop the workflow. One must use a `try` and `except` block to 
deal with the issues which is the current implementation in `planetapi.py`.  

The `utils/directory.py` moves those files that produce issues to the `Issues`
folder and the ones that are requested to the `Ordered` folder. In case the 
folders are not in the current directory it will create them. 