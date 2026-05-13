+++
date = '2025-01-09T13:50:02+01:00'
title = 'La Coctelera'
cover= 'img/shaker_small.jpg'
description = 'A collaborative cocktail data base'

+++

##### *La Coctelera* is a project to create a collaborative open database for cocktail recipes.

> This project was born in 2024 and it was inspired by the [AeroPrecipe](https://aeroprecipe.com/) web site. On that site, people can post their own recipes for making delicious coffee using an AeroPress brewer. The recipes are organised into categories and voted on, which makes the site a great place to share and learn about coffee and try out new recipes from other users.
>
> Many web sites and blogs offer cocktail recipes, but there is no such a place like AeroPrecipe for cocktails as of today. This project aims to fill that gap, and foster an open community behind the passion of preparing and drinking cocktails.

# Goals

The main goal of the project is to deliver a web site alike to AeroPrecipe, in which users can post their recipes, and develop together a data base for cocktail recipes.

Recipes will be organised, based on categories, ingredients and so on, so it will be easy for people to find recipes based on types of liquors, types of glass or any other attributes. Also, a voting system will be used to help highlight the recipes that most of the people enjoy.

From a more technical point of view, the project aims to deliver a segregated back-end service from the main front-end that people would use to access the information of the database. This way, people with programming skills can develop their own clients for specific platforms, such as Android, simply connecting to the back-end service.

# Front-end

The main access point to the data base will be a web page that people could use to simply access cocktail recipes, or they could register and post their own recipes.

The development of the front-end is still in an early stage. If you are interested in it, please visit this repository in GitHub:

<a href="https://github.com/felipet/lacoctelera_frontend" class="button inline">Front-end repository in GitHub</a>

# Back-end

An open back-end service is developed to allow other interested people to connect their own clients to the open database. Despite developing a segregated solution requires more work, we believe this would encourage programmers to develop clients for other platforms in the future.

The back-end offers a REST API whose documentation can be accessed at the projects page:

<a href="https://nubecita.eu/coctelera/api/v0/" class="button inline">API documentation</a>

The API includes several resources that allow accessing data regarding ingredients, authors and recipes. **Resources to access data are open**, however resources to push or modify data of the data base require an authentication token to be provided. Tokens can be freely requested (see the link included in the API docs), and the main purpose of such security measure is to prevent spamming or malicious usage of the data base.

## Development

The back-end service is close to reach the first stable version. If you are interested in the development or the source code, it is open source and available at:

<a href="https://github.com/felipet/lacoctelera_backend" class="button inline">Back-end repository in GitHub</a>

The service is temporally deployed at [nubecita.eu](https://nubecita.eu/coctelera/api/v0/). The data base only contains a few example recipes at the moment, but it could start to be used for testing purposes. 

If you visit the API docs page, a Swagger UI is delivered that works as demo of the allowed operations with the API.

For instance, it is possible to issue a request to check if there is any recipe about how to prepare a *Cosmopolitan* using **curl** as follows:

```bash
curl -X 'GET' \
  'https://nubecita.eu/coctelera/api/v0/recipe?name=cosmopolitan' \
  -H 'accept: application/json'
```

