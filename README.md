# Data Warehouse and Analytics Project

## About This Project

This project is my hands on practice of building a full data warehouse from scratch, following the well known SQL data warehouse course by Data with Baraa. I built this to understand how real companies collect messy data from different systems and turn it into something clean and useful for reporting and analysis.

The idea behind the project is simple. You take raw data coming from two different source systems, an ERP system and a CRM system, and you bring it together into one central place where it can actually be trusted and used for decision making.

## The Architecture

I followed the medallion architecture, which is basically a layered approach to organizing data as it moves through the pipeline. There are three layers.

**Bronze Layer**
This is where the raw data lands exactly as it comes from the source files. Nothing is changed here. It is basically a copy of the original CSV files loaded straight into SQL Server tables.

**Silver Layer**
This is where the real cleaning happens. I handled duplicate records, fixed inconsistent formatting, standardized column names, dealt with null values, and made sure the data types made sense. This layer takes the messy bronze data and turns it into something that is actually usable.

**Gold Layer**
This is the final layer where the data is modeled into a proper star schema, with fact and dimension tables. This is the layer that is meant to be used directly for reporting, dashboards, and analysis.

## Tools I Used

SQL Server and SQL Server Management Studio for building and managing the database
Draw.io for sketching out the data architecture and data flow diagrams
Git and GitHub for version control and keeping track of my progress
Notion for planning out the project steps and keeping notes

## What I Actually Did

I started by loading the raw CRM and ERP CSV files into the bronze layer using bulk insert. From there I wrote SQL scripts to clean and transform the data into the silver layer, this included things like removing duplicates, trimming whitespace, fixing date formats, and standardizing values that were written differently across systems but meant the same thing. After that I modeled the cleaned data into fact and dimension tables in the gold layer so it follows a proper star schema that is easy to query and build reports on top of.

## Repository Structure

**datasets** folder contains the raw source CSV files used for the project

**scripts** folder contains all the SQL scripts organized by layer, bronze, silver, and gold

**docs** folder contains diagrams and notes explaining the data architecture, data flow, and data model

**README.md** this file, explaining what the project is about

## Why I Built This

I wanted to actually understand the full journey data takes before it becomes a dashboard someone in a company looks at. It is one thing to write a SQL query, it is another thing to understand why the data needed to be cleaned that way in the first place, and how a warehouse is actually structured behind the scenes. This project helped me connect the dots between raw data and the kind of reporting work I do as a data analyst.

## Credit

This project follows the structure and teaching of the Data Warehouse course by Data with Baraa on YouTube. I built and wrote every script myself as part of learning the concepts, and I made this repository to document my own learning process and to have something practical to show as part of my portfolio.
