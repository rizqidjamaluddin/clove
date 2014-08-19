# Clove

Clove is a new way to store and retrieve data in your Laravel 4 projects.

## Using Clove



## FAQ

#### Where's all the code?
Clove is starting off as a proof-of-concept specification as I tinker with different implementations.

#### How is this different from Eloquent?
Eloquent is an ActiveRecord ORM. You build "queries" with it and it gives you magic models that know how to save themselves to your database. Clove uses a Data Mapper, which means you give it vanilla PHP objects to save (with their custom configuration within the repository), and uses a different style to get data back. This gives more separation between your domain logic and how data is saved.

#### Why not use an existing data mapper?
I wanted a fresh view on the pattern (along with a new concept for repositories) - the existing frameworks are great, but building from scratch has its own benefits too.
