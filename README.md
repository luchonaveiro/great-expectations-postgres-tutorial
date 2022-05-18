# Great Expectations Tutorial

[![Generic badge](https://img.shields.io/badge/great_expectations-0.15.5-blue.svg)](https://docs.getdbt.com/dbt-cli/cli-overview)
[![Generic badge](https://img.shields.io/badge/PostgreSQL-13-blue.svg)](https://www.postgresql.org/)
[![Generic badge](https://img.shields.io/badge/Python-3.7-blue.svg)](https://www.python.org/)
[![Generic badge](https://img.shields.io/badge/Docker-20.10.6-blue.svg)](https://www.docker.com/)

On the following tutorial we are going to spin up a local `PostgreSQL` database, create some tables, insert data to them, and create some data validations using [Great Expectations](https://greatexpectations.io/) library.

## Great Expectations
`Great Expectations` is the leading tool for **validating**, **documenting**, and **profiling** your data to maintain quality and improve communication between teams.

With `Great Expectations`, you can assert what you expect from the data you load and transform, and catch data issues quickly – *Expectations* are basically unit tests for your data. Not only that, but `Great Expectations` also creates data documentation and data quality reports from those *Expectations*. Data science and data engineering teams use `Great Expectations` to:
- Test data they ingest from other teams or vendors and ensure its validity. 
- Validate data they transform as a step in their data pipeline in order to ensure the correctness of transformations. 
- Prevent data quality issues from slipping into data products. 
- Streamline knowledge capture from subject-matter experts and make implicit knowledge explicit. 
- Develop rich, shared documentation of their data.

### Expectations
*Expectations* are assertions about your data. In `Great Expectations`, those assertions are expressed in a declarative language in the form of simple, human-readable Python methods. 

### Automated data profiling
Writing pipeline tests from scratch can be tedious and overwhelming. `Great Expectations` jump starts the process by providing automated data profiling. The library profiles your data to get basic statistics, and automatically generates a suite of Expectations based on what is observed in the data.

### Data Validation
Once you’ve created your *Expectations*, `Great Expectations` can load any batch or several batches of data to validate with your suite of *Expectations*. `Great Expectations` tells you whether each *Expectation* in an *Expectation Suite* passes or fails, and returns any unexpected values that failed a test, which can significantly speed up debugging data issues!

### Data docs
`Great Expectations` renders *Expectations* to clean, human-readable documentation, which we call Data Docs, see the screenshot below. These HTML docs contain both your *Expectation Suites* as well as your data Validation Results each time validation is run – think of it as a continuously updated data quality report.

## Project Setup

To start with the tutorial, we should create the `PostgreSQL` and insert the data to it. To do so, we can execute the following commands:
```
$ cd db
$ docker compose up
```
This will create a databse called `raw` and three tables:
- `jaffle_shop.customers`
- `jaffle_shop.orders`
- `stripe.payment`

Once the data is inserted, we can start building the data profiles and data validations using `Great Expectations`.
```
$ pip3 install great_expectations==0.15.5
$ great_expectations init
```

This will create the directory called `great_expectations` containig all the necessary files. Now, we should configure `Great Expectations` to use the local `PostgresDB` as its backend:
```
$ great_expectations datasource new
```

There we should choose `Relational Database (SQL)` and select `PostgreSQL`. A `jupyter notebook` will open and there we can configure the local database with the credentials defines on the `db/docker-compose.yml` to access the database.

Once the connection is defined, we can create the data *Expectations*`. To create them, you can use the profiler over a batch of data, but you should rely also on business insights, to understand if those data validations proposed autoamtically makes sense or not. As a guideline, here you can find the different *[Expectations](https://great-expectations.readthedocs.io/en/v0.3.2/glossary.html)*.

Let's create the *Expectations* for `jaffle_shop.customers` table.
```
$ great_expectations suite new
```

Select the 3rd option (the one that states `Automatically, using a profiler`), choose the table to validate, and choose a name, or keep the proposed one. This will open a new `jupyter notebook` that once runned, will create the necessary `.json` file defining the data *Expectations*. In case we want to review or add new validations, we can execute:
```
great_expectations suite edit jaffle_shop.orders.warning # this should be the name selected
```

There we can choose `Interactively, with a sample batch of data`, and a notebook will open, and there we can add the new validations. Once runned, the `.json` filestored on `expectations` directory, will be modified with the proper validations.

In this way, we can create the *Expectations* of all the desires tables from the database.

Once the *Expectations* have been defined, we can run them by executing:
```
$ great_expectations checkpoint new jaffle_shop__orders
```
That will open another `jupyer notebook`, where we can modify the configurations to select the proper *Expectations* to be triggered when executing that particular `checkpoint`. This command will create the proper `.yml` file with the configuration on the `checkpoints` directory. 

Finally, run the validations, we execute:
```
$ great_expectations checkpoint run jaffle_shop__orders
```

Also, we can open the `uncommited/data_docs/local_site/index.html` file to see the result of the different runs, together with the definition of each *Expectation*.