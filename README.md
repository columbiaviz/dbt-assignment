The following are instructions to get dbt running locally on your computer.  
The instructions are for Mac OS, using pip as the Python package installer.  
We assume you are using Python 3.x


### Setup



Installation

    # create Python virtual environment for assignment
    vf new dbt

    # use custom dbt-duckdb adapter
    git clone https://github.com/columbiaviz/dbt-duckdb.git
    cd dbt-duckdb 
    pip install .


    # !!You can skip this if starting from the github repo
    # create new dbt project using duckdb
    dbt init test --adapter duckdb


Download the duckdb database containing incarceration statistics

    # run in the command line, or manually download the file
    wget "https://www.dropbox.com/s/6bq1q8bf3xnfb6t/incarceration.db"

If this database file does NOT work with your installed version of duckdb, it's OK.  You can create the table yourself:

    # 1. download the CSV file

    # 2. run duckdb:
    duckdb

    # 3. create the table in the duckdb prompt.  "D" below is their prompt
    D CREATE TABLE incarcerations as SELECT * FROM read_csv_auto(PATHTOCSV)

Now we configure your global dbt profile to actually use duckdb, 
**You can skip this if starting from the github repo**


    # edit ~/.dbt/profiles.yml and add the following lines
    # !! change path to wherever your duckdb database files is
    dbt_duckdb:
      target: dev
      outputs: 
        dev:
          type: duckdb
          path: /Users/eugenewu/code/dbt/test/incarceration.db

Now we configure your new dbt project to know about duckdb (yea, seems redundant)
and install a useful dbt utility package
**You can skip this if starting from the github repo**

    # edit ./dbt_project.yml and set
    profile: 'dbt_duckdb'

    # edit ./packages.yml and add the following lines
    packages:
      - git: git@github.com:dbt-labs/dbt-utils.git
        revision: 0.7.4


Run the following command to install the dependencies 

    dbt deps


### Create the transforms


First, [register the base tables in duckdb as sources](https://docs.getdbt.com/docs/building-a-dbt-project/using-sources):

    # edit models/example/schema.yml and add the following lines
    sources:
      - name: incarceration
        schema: main
        tables:
          - name: incarceration



Now, you will add `.sql` files (called a "model" in dbt) under [models/example/](./models/example) to define transformations,
and register each `sql file` in [models/example/schema.yml](./models/example/schema.yml).


Some things you may find helpful/tips

* You may find the [unpivot command](https://github.com/dbt-labs/dbt-utils#unpivot-source) to be very useful.
* The following are a list of attributes that do NOT encode data in the attribute names:

    'yfips','year','fips','state','county_name','urbanicity','region',
    'division','commuting_zone','metro_area','land_area','jail_rated_capacity' 

* First identify the real attributes encoded in the dataset's attribute names,
  define them using a dbt model, and then populate them using more models

### Run everything


    dbt run

Look in your database file to see if everything worked correctly!

### Resources:
- Learn more about dbt [in the docs](https://docs.getdbt.com/docs/introduction)
- Check out [Discourse](https://discourse.getdbt.com/) for commonly asked questions and answers
- Join the [chat](https://community.getdbt.com/) on Slack for live discussions and support
- Find [dbt events](https://events.getdbt.com) near you
- Check out [the blog](https://blog.getdbt.com/) for the latest news on dbt's development and best practices


