# Clove

Clove is a new way to store and retrieve data in your Laravel 4 projects.

Building principles:
* We're after elegant, clear, and easier-to-work-with code
* A bit of extra complexity on the repositories is acceptable to afford a better domain layer
* Prefer explicit declarations over assumptions and magic
* Flexibility of backend and caching is important
* Repositories should act like collections

## Using Clove

### Create the entity

```php
class Post implements Clove\Entity {
  
  public function __construct($title, $body) {
    $this->title = $title;
    $this->body = $body;
  }
  
  public function getTitle() {
    return $this->title;
  }
  
  public function getBody() {
    return $this->body;
  }
  
}
```

### Create a repository

```php
class PostRepository extends Clove\SqlRepository {
  
  public function getFields(Entity $entity) {
    return [
      new SqlField('Title', $entity->getTitle(), ['type' => 'varchar', 'length' => 200, 'required' => true]),
      new SqlField('Body',  $entity->getBody(),  ['type' => 'text', 'required' => true]),
    ];
  }
  
}
```

### Use it

#### Creating, saving and getting entities

#### Custom repository methods

#### Collection-style repository methods

## FAQ

#### Where's all the code?
Clove is starting off as a proof-of-concept specification as I tinker with different implementations.

#### That repository `getFields` looks like a lot of work.
The example above shows the explicit programmatic option; this gives a lot of flexibility. In the future, there will be annotation- and configuration-based mappings available.

#### How is this different from Eloquent?
Eloquent is an ActiveRecord ORM. You build "queries" with it and it gives you magic models that know how to save themselves to your database. Clove uses a Data Mapper, which means you give it vanilla PHP objects to save (with their custom configuration within the repository), and uses a different style to get data back. This gives more separation between your domain logic and how data is saved.

#### Why not use an existing data mapper?
I wanted a fresh view on the pattern (along with a new concept for repositories) - the existing frameworks are great, but building from scratch has its own benefits too.
