# KnexJs
Knex.js
Learning Objectives
By the end of this lesson, you will be able to:

Run database migrations in Knex.js to configure a schema
Seed (populate) a database using Knex.js
Respond to HTTP requests with database information using Knex.js in an Express server
Installing Dependencies
Your purpose is to give a web server (an Express application) read and write access to a database. So far, you have had to create the database and its schema separately from the application but, by using a tool like Knex.js, a JavaScript-based database query and schema builder, you can abstract away the syntax for those operations into JavaScript.

This sounds interesting, so let us create an npm project and think about what you need in order to get your database querying web server up and running. You'll be submitting a link to this GitHub repository at the end of this lesson, so be sure to follow along closely and push all of your changes.

npm init -y
You will need the following things:

a web server: Express
a database client for your database management system ( DBMS ): Postgres 
a query and schema builder, Knex.js 
Now, to install them, respectively.

npm i express pg knex
Database Support
Though you will be using Postgres, note that, per the Knex documentation  , "Postgres, MSSQL, MySQL, MariaDB, SQLite3, Oracle, and Amazon Redshift" are all supported. It is necessary to install the appropriate database client for your database management system of choice. You have chosen Postgres - good decision.

Initializing Knex
Knex operates based off a configuration file called the knexfile(.js, of course). There is a built in operation that will scaffold this file for you.

npx knex init
Running the above command generates a "Knexfile template" at the root of your project - that is all it does. Let's look at the file in detail.

// knexfile.js
// Update with your config settings.

module.exports = {

  development: {
    client: 'sqlite3',
    connection: {
      filename: './dev.sqlite3'
    }
  },

  staging: {
    client: 'postgresql',
    connection: {
      database: 'my_db',
      user:     'username',
      password: 'password'
    },
    pool: {
      min: 2,
      max: 10
    },
    migrations: {
      tableName: 'knex_migrations'
    }
  },

  production: {
    client: 'postgresql',
    connection: {
      database: 'my_db',
      user:     'username',
      password: 'password'
    },
    pool: {
      min: 2,
      max: 10
    },
    migrations: {
      tableName: 'knex_migrations'
    }
  }

};
In the Knexfile, you have, simply, an object with "key/value" pairs where the keys represent "runtimes", or "lifecycles", for your application. The values represent how you want Knex to connect in a particular lifecycle. Right now, you will make some modifications to the development section since it is currently set up for a SQLite database - remember, you chose to use Postgres.

// knexfile.js
module.exports = {

  development: {
    client: 'pg',
    connection: 'postgres://USER_NAME:USER_PASSWORD@localhost/DATABASE_NAME'
    // replace USER_NAME, USER_PASSWORD, and DATABASE_NAME with your Docker PostgreSQL container's username, password and an *empty* database
    // that you have created on your Docker PostgreSQL container volume
  },

  ...

  // the other two keys, you do not need to modify right now
  // but, as you might guess, they and any other objects you add
  // would configure Knex for connecting in the corresponding lifecycle
}
Now, the Knexfile has what it needs in order to connect to your Postgres database at the provided connection URL using the pg client. This is great news.

Modifying the Database Schema
You are now able to make a connection to the database in the Knexfile. For example, let us create a table that will hold some of your favorite movies. Your knowledge of SQL suggests that the statement might look something like:

-- Create table statement with 4 fields
CREATE TABLE movies (
  id serial PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  genre VARCHAR(50),
  release_date DATE NOT NULL
);
This is still true and as full-stack software engineers you are still responsible for being aware of this syntax. But do you need to write it? No, thank you. Using Knex.js, you can create what is called a "migration", a change to the database schema that can be tracked and version controlled.

npx knex migrate:make create_movies
The choice to name the migration create_movies while a great one, does not determine which table you are creating. Creating the migration, as you can see in the folder structure, adds a migrations/ directory that contains a JavaScript file named after the migration prepended with the datetime of the creation of the migration. This is what it looks like.

// YYYYMMDDHHMMSS_create_movies.js

exports.up = function(knex) {

};

exports.down = function(knex) {

};
The up export represents a forward migration and the down export represents a "rollback".

// YYYYMMDDHHMMSS_create_movies.js
exports.up = function(knex) {
  return knex.schema.createTable('movies', table => {
    table.increments('id'); // adds an auto incrementing PK column
    table.string('title').notNullable();
    table.string('genre');
    table.date('release_date');
    table.timestamps(true, true); // adds created_at and updated_at
  });
};

exports.down = function(knex) {
  return knex.schema.dropTableIfExists('movies');
};
When you run this migration, your movies table will be created with an auto-incrementing PRIMARY KEY column named id, a title column that must have a value, a genre column of data type STRING, and a release_date column. Knex also provides an easy way to (optionally) keep track of the created_at and updated_at values of data entries using the .timestamps method.

Alright, so do you have a table yet? If you inspect your database you will see it does not yet have any relations. You will now run the latest migrations so that your database schema can be updated (or, created for the first time).

npx knex migrate:latest
This command will run all of the up exports from every migration file in the migrations/ directory - you only have one. After running it, you should see that the database now has a movies table with no data in it. Should you seed some data?

Rollback
If you ran

npx knex migrate:rollback
what do you think would happen to the database?

Seeding Data
Your database is actually ready for operation, at this time. During development, though, it would be nice to have seed data, especially when you are testing how a client might consume data coming from the database and are not interested in using data from the production (or other) lifecycle.

So let us make some seed data. Knex.js can do it.

npx knex seed:make initial_movies
This command will create seed data in the seeds directory specified in the Knexfile or in a directory called seeds/ by default.

Seed Files
Seed files are executed in alphabetical order. As such, you must pay attention to what data you are trying to seed with relation to your database schema.

// seeds/initial_movies.js
exports.seed = function(knex) {
  // Deletes ALL existing entries
  return knex('movies').del()
    .then(function () {
      // Inserts seed entries
      return knex('movies').insert([
        {id: 1, title: 'Vicky Cristina Barcelona', genre: 'drama', release_date: '2008-08-15'},
        {id: 2, title: 'Orfeu Negro', genre: 'drama', release_date: '1959-12-21'},
        {id: 3, title: 'Midnight in Paris', genre: 'drama', release_date: '2011-05-20'},
      ]);
    });
};
Would it take fewer characters to write the INSERT statement in SQL? Yes.

Would it be JavaScript? No.

Okay, so let's seed this data.

npx knex seed:run
Now, the movies table has data that you can serve via your Express server.

Touch an app.js and wire it up to use Express.

// app.js
const express = require('express');

const app = express();
const PORT = process.env.PORT || 3000;
const knex = require('knex')(require('path/to/knexfile.js')[process.env.NODE_ENV]);

app.use(bodyParser.json());

app.get('/movies', function(req, res) {
  knex
    .select('*')
    .from('movies')
    .then(data => res.status(200).json(data))
    .catch(err =>
      res.status(404).json({
        message:
          'The data you are looking for could not be found. Please try again'
      })
    );
});

app.listen(PORT, () => {
  console.log(`The server is running on ${PORT}`);
});
What about creating, updating & deleting?

See this section  in the Knex documentation.

Wow, wrote no SQL but still SQLing so hard.
