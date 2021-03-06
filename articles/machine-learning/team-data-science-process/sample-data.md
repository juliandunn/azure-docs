---
title: Sample data in Azure blob containers, SQL Server, and Hive tables | Microsoft Docs
description: How to explore data stored in various Azure enviromnents.
services: machine-learning
author: marktab
manager: cgronlun
editor: cgronlun
ms.service: machine-learning
ms.component: team-data-science-process
ms.topic: article
ms.date: 11/13/2017
ms.author: tdsp
ms.custom: "(previous author=deguhath, ms.author=deguhath)"
---
# <a name="heading"></a>Sample data in Azure blob containers, SQL Server, and Hive tables

The following articles describe how to sample data that is stored in one of three different Azure locations:

* [**Azure blob container data**](sample-data-blob.md) is sampled by downloading it programmatically and then sampling it with sample Python code.
* [**SQL Server data**](sample-data-sql-server.md) is sampled using both SQL and the Python Programming Language. 
* [**Hive table data**](sample-data-hive.md) is sampled using Hive queries.

This sampling task is a step in the [Team Data Science Process (TDSP)](https://docs.microsoft.com/azure/machine-learning/team-data-science-process/).

**Why sample data?**

If the dataset you plan to analyze is large, it's usually a good idea to down-sample the data to reduce it to a smaller but representative and more manageable size. This facilitates data understanding, exploration, and feature engineering. Its role in the Cortana Analytics Process is to enable fast prototyping of the data processing functions and machine learning models.

