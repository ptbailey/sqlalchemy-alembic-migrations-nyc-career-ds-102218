
# Mechanics of Alembic Migrations

## Objective

1.  Understand how migrations help us manipulate database schema
2.  Learn how to write migrations for SQLAlchemy databases with Alembic

## Migrations

In the "Mapping and Table Creation" lab, we learned how to use Python to map classes to a database table.  We named a database and established a connection to that database, wrote a class that could map to a table in that database, then executed `create_all()` to create the database and that table.

Remember how if we realized that we made a mistake creating our schema, we were forced to delete the database file and try again?  The problem with `create_all()` is that it only can create tables from scratch.  It cannot modify them afterwards.  SQLAlchemy looks at our models whenever we run `create_all()`.  If the database already has a table matching a class in our models file, SQLAlchemy skips over it until it finds something new to map to the database.  Therefore, we cannot make changes to our models, re-execute `create_all()`, and expect these changes to be reflected in our database.  Wouldn't it be nice if there were a way to make changes without having to delete the earlier version of the database and re-create the newer version?

Fortunately, there is!  We can use migrations to manage any changes made to our database schema.  Migrations are sort of like a version control for databases.  We can make a table, populate it with data, and, if we decide to add a column later on, we can write a migration to make this change without worrying too much about losing the data in the table.  Migration tools aim to minimize the effect a change in the database schema has on the existing data stored in the tables.  If we later decide to undo this change, we can rollback the migration to return to our database's prior state.

## SQLAlchemy Migrations with Alembic

[Alembic](http://alembic.zzzcomputing.com/en/latest/) is the migration tool we use with SQLAlchemy.  Alembic provides us with a simple way to create and drop tables, and add, remove, and alter columns.  Fork and clone this repository and we'll walk through writing Alembic migrations together.

> To install Alembic, run `pip install alembic` in your terminal

#### Step 1: Define model classes and configure the new database

Even though we can use Alembic to create tables for us, we still need to configure the database, establish the connection, and write our initial models as we have done previously.

*  In `models.py` let's create an Artist model.  Each Artist will have a name, a genre, an age, and a hometown.  Make sure you include an ``__init__`` construction function so we can instantiate new Artist objects later on.
*  Below the Artist model, include the following lines of code to configure our new database:
    ```
    engine = create_engine('sqlite:///artists.db')
    Base.metadata.create_all(engine)
    ```

*  Now, let's run `python models.py` in our terminal to execute all the code in this script and generate the `artists.db` script containing our database.

#### Step 2: Initialize and setup the Alembic environment



*  In your terminal run the following command to initialize the Alembic environment:

```python
alembic init alembic
```
Notice that Alembic has generated a number of things for us, including the `alembic` subdirectory and the `alembic.ini` file.  Our migration scripts will appear inside the `alembic/versions` folder, but first we must tell Alembic to talk to our database.

* Set "sqlalchemy.url" (line 38) in `alembic.ini` to point to our database.  Our database name is the string we provided to the `create_engine` function when we first configured the database.

```python
sqlalchemy.url = sqlite:///artists.db
```
We can test our connection by running `alembic current` in the terminal.  You should see the following output:

```
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
```

#### Step 3: Generate a migration



* Run the command below to generate a migration script:

```python
alembic revision -m "baseline"
```
The `-m` tag names our migration "baseline".  Notice that a migration script has been created in the `alembic/versions/` subdirectory for us!  This is where we will write our migrations.  Alembic created a unique id and empty `upgrade` and `downgrade` functions.

```python
# alembic/versions/<auto_generated_revision_id>_baseline.py

from alembic import op
import sqlalchemy as sa


def upgrade():
    pass


def downgrade():
    pass
```

**A note on Upgrade/Downgrade**

Upgrade contains the code that executes when we run our migration whereas downgrade contains the code executed when we rollback a migration.  Therefore, it is generally a good idea to make our `downgrade` functions have the inverse functionality of the the code in the `upgrade` function.  For example, if we want to create a table in the `upgrade`, we ought to provide a drop table command in the `downgrade`.

#### Step 4: Write the migration

We already have a table for artists, but we forgot to make a table for songs!  Let's fill in the `upgrade` function with code that will create our songs table.  While we're at it, let's also include the code for dropping the songs table in the `downgrade` function in case we need to reverse these changes for whatever reason.

* Add the code below to the revision script:

```python
def upgrade():
    op.create_table(
        'songs',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('name', sa.String()),
        sa.Column('length', sa.Integer())
        )


def downgrade():
    op.drop_table('songs')
```

We use `op` from Alembic to specify the kind of alteration we wish to make to our database schema.  Alembic already aliased sqlalchemy as `sa` in the revision script.  We use this to specify the columns we want to include in our new table as well as the datatypes to use in each column.

* Finally, we are ready to execute the migration.  Run the code below in your terminal, which ensures that our database is up-to-date with the most recent revision.
```python
alembic upgrade head
```


> **Note:** If we wanted to rollback the previous revision, we could instead run `alembic downgrade -1`.

#### Step 5: Update the models to match the table changes

The last thing we need to do is modify our models so that they match the changes we made to our database.  In this case, we added a `songs` table, so we need to add a Song class to the `models.py` file.

* Add a model class for Song in the `models.py` file.  Make sure to include the constructor function that initializes each Song instance with a name and length, just as we specified in the migration.

* Now uncomment the code in `insert.py` and play around with are updated database!

## Summary

In this lesson, we learned that we can use migrations to update a database schema without having to start from scratch.  Setting up migrations requires some more work up front; however, it makes our lives much easier later on when our programs get more complicated.

First, we setup our database and initial table as usual.  Then, we initialized the Alembic environment and configured it to communicate with our database.  We then generated a migration and learned `upgrade` and `downgrade`.  We added code to these functions to be executed when we run the migration.  Finally, we migrated these changes and updated our models to match these changes.
