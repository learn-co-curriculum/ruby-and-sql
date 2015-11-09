# Ruby and SQL

## Objectives

1. Understand why it is essential for our Ruby programs to interact with our SQL databases. 
2. Understand how the sqlite3 Ruby gem provides us with a library of Ruby code that can execute commands against a SQL database. 

## Why Does Ruby need to Interact with SQL?

We understand the need for persistence, or saving data, in the applications we build. After all, if a web application prompted you to sign up every time you visited the webpage and never remembered any of your personal data, you would be unlikely to use it. 

We know that our programs seek to model real-world environments and situations. For example, if our application will have users, we are likely to create a `Users` model, or class, that produces user objects, each of which contain the attributes and behaviors of an individual user. If our application is designed to help teachers keep track of their students and assignments, it is likely to have a `Teacher`, `Student` and `Assignment` class. If our app is designed for a user to store and browse music, it will likely have `Song`, `Artist` and `Genre` models. 

We've also learned how to create SQL databases in which each table in a database stores information about a particular entity. Our music player application might come with a database that has an artists, songs and genres table. 

What does is mean for our Ruby program that handles songs, artists and genres to have a database? How can the Ruby objects produced by a `Song`, `Artist` or `Genre` class be persisted, such that a user can store their music library and interact with it again and again? 

It is possible to write Ruby code that:

* Connects your Ruby program to a specific database.
* Stores information that represents Ruby objects in that database. 

In other words, we can write Ruby code in our music player program that creates and connects to a music program database and stores information representing each individual artist, song and genre that our program produces. 

We can do this with the sqlite3 Ruby gem. 

## The SQLite3-Ruby Gem

The SQLite3-Ruby gem provides us with a library of code that allows us to connect our Ruby program directly to a SQLite database. 

Let's think about the ways in which we would need to interact with a SQL database in order to store information about a given program. 

We would need to:

* Create a database.
* Create database tables, including defining their column names and types. 
* Execute SQL statements that insert data into those tables and retrieve data from those tables. 

Let's take a look at each desired action and compare accomplishing it using SQL statements in the terminal versus using Ruby code provided to us by the SQLite3-Ruby gem. 

### Creating a Database

For the purposes of this example, we'll use a pets domain model. Our program has an `Owners` class and a `Cats` class. We want to be able to store information about owners and cats in a SQL database. 

To create such a database in the terminal, we would use the follwoing:

```bash
sqlite3 pets_database.db
```

To create a new database from *inside our Ruby program* (cool!), we would use the following code:

```ruby
SQLite3::Database.new('db/pets.db')
```

This will create a new database called "pets.db", stored inside the `db` subdirectory of our app *and* it will return a Ruby object that **represents the connection between our Ruby program and our newly-created SQL database**. Here's a look at the object that gets returned by the line of code above:

```ruby
#<SQLite3::Database:0x007f9d6c294508
 @authorizer=nil,
 @busy_handler=nil,
 @collations={},
 @encoding=nil,
 @functions={},
 @readonly=false,
 @results_as_hash=nil,
 @tracefunc=nil,
 @type_translation=nil>
```

This object is created for us by the code provided by the SQLite-Ruby gem. Don't worry too much about what is going on under the hood. The important thing to understand is that this is the object that connects the rest of our Ruby program, i.e. any code we write to create artists, songs and genres, to our SQL database. 

There are a number of methods available to us, provided by the SQLite-Ruby gem, that we can call on the above object to execute commands against our database. Let's move on to take a look at some of those methods now. 

### Executing SQL Statements Against a Database

Once you create a database, you'll want to execute commands against it, such as creating a table and inserting data into that table. 

In the terminal, once you execute `sqlite3 pets_database.rb`, you'll be dropped into your SQLite3 prompt. Here we might create a table like this:

```bash
CREATE TABLE cats (
  id INTEGER PRIMARY KEY,
  name TEXT,
  age INTEGER,
  breed TEXT,
);  
```

Let's take a look at the equivalent of executing this SQL statement using the SQLite3-Ruby gem and writing code inside our Ruby program. 

First, we would have created our database with the following:

```ruby
database_connection = SQLite3::Database.new('db/pets.db')
```

Now, we have a variable, `database_connection`, set equal to the return value of calling `SQLite3::Database.new('db/pets.db')`. If you recall from above, the return value of that method call is an object that represents the connection between our Ruby program and our newly created database. 

Now let's make that cats table. 

```ruby
database_connection.execute("CREATE TABLE IF NOT EXISTS cats(id INTEGER PRIMARY KEY, name TEXT, breed TEXT, age INTEGER)")
```

Here we use the **`#execute` method provided to us by the SQLite3-Ruby gem.** This method takes in an argument of a string that contains valid SQL statements. The method is called on the object that represents the connection between our Ruby program and our database. 

We can give the `#execute` method an argument of *any valid SQL statement, as a sting*. We can use it, as above, to create a table. We can also use it to add data to and retrieve data from a table:

```ruby
database_connection.execute("INSERT INTO cats (name, breed, age) VALUES ('Maru', 'scottish fold', 3)") 
```

## Persisting Ruby Objects

What's so great about being able to talk to our database from within our Ruby program? Well, now that we see we can use code like the lines above to store data in our database directly from our program, we can use this code to store data about our existing Ruby objects. 

Let's say we have a class, `Cat`, that produces individual cat objects. 

```ruby
class Cat
  attr_accessor :name, :breed, :age
  
  @@all = []
  
  def initialize(name, breed, age)
    @name = name
    @breed = breed
    @age = age
    @@all << self
  end
  
  def self.all
    @@all
  end
```

Here we have a class variable, `@@all`, that points to an array of all of our existing cat objects. Each cat objects has a number of attributes: `name`, `breed` and `age`. 

With the use of our SQLite3-Ruby gem, we can take this list of cat objects, stored in the `@@all` array and retrievable thanks to our `Cats.all` class method, and use the information about each cat to store a record in the cats table of our pets database. Let's take a look:

```ruby
Cat.all.each do |cat|
  database_connection.execute("INSERT INTO cats (name, breed, age) VALUES (#{cat.name}, #{cat.breed}, #{cat.age}"))
end
```

We use plain old string interpolation to grab the attributes of a given cat and insert that data into our database, with the help of our `database_connection` object. Amazing! We just took a collection of Ruby objects and stored the data describing those objects directly in our SQL database. 

**Advanced:** Note that the cat record we are storing in our database *is not the actual Ruby object*. We are simply taking the data that describes that object and storing it in the database. When we run code to select those records from our cats table, we do not receive Ruby `Cat` objects back from the database. Instead, we receive collections of attributes that describe the individual cats we previously created. 

Let's check it:

```ruby
database_connection.execute("SELECT * FROM cats")
# => [[1, "Maru", "scottish fold", 3]]
```

Let's take a closer look at the return value of selecting a record from our database, from within our Ruby program. 

```ruby
[[1, "Maru", "scottish fold", 3]]
```

Our return value is an array of arrays. This particular array only contains one child array, because we only inserted one record into our cats table. The child array that represents a single cat has four index elements, an element representing the primary key integer ID, the cat's name, breed and age. 

## Conclusion

We'll learn more about storing data describing Ruby objects in and retrieving data describing Ruby objects from a database later on. For now, it's important to understand:

* Why it is so useful to be able to connect our Ruby program and our database: because we can store information describing our Ruby objects!
* How to use the code provided by the SQLite3-Ruby gem to created a connection between your Ruby program and your database. 










The end goal is for students to understand all of the stuff around doing this (https://github.com/learn-co-curriculum/pokemon-scraper/blob/solution/bin/run.rb) and this (https://github.com/learn-co-curriculum/pokemon-scraper/blob/solution/bin/sql_runner.rb)

 - Start with explaining that we've done a lot with SQL but we need to actually bring it into ruby
 - Compare to the command line commands. So things like when I type sqlite3 <DBNAME> that's the same as SQLite3::Database.new('<DBNAME')

 
db = SQLite3::Database.new('db/pokemon.db')
sql_runner = SQLRunner.new(db)

class SQLRunner
  def initialize(db)
    @db = db
  end

  def execute_schema_migration_sql
    sql = File.read('db/schema_migration.sql')
    execute_sql(sql)
  end

  def execute_create_hp_column
    sql = File.read('db/alter_table_migration.sql')
    execute_sql(sql)
  end

  def execute_sql(sql)
     sql.scan(/[^;]*;/m).each { |line| @db.execute(line) } unless sql.empty?
  end
end
