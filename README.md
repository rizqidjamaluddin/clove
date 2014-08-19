# Clove

Clove is a new way to store and retrieve data in your Laravel 4 projects.

## Principles

Entities should be entities - they shouldn't care about how they're persisted at all. Anything should be a persistable entity, as well. Clove puts the persistence on the repository side, where this logic belongs. This gives total flexibility on how persistence works, avoiding magical mutators or translations, and opening the possibility to use caching software or even flat files to store entities. All this affords a clean, enjoyable entity.

Clove repositories try their best to act like collections, as well, allowing for more intuitive and uniform interaction. See below for examples.

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
      new SqlField('title', $entity->getTitle(), ['type' => 'varchar', 'length' => 200, 'required' => true]),
      new SqlField('body',  $entity->getBody(),  ['type' => 'text', 'required' => true]),
    ];
  }
  
  /**
   * Get post with the longest content.
   * Note: getBuilder is only available in SqlRepository; it contains a laravel query builder instance.
   *
   * @return Post|null
   */
  public function longest() {
    return $this->getBuilder()->orderBy(DB::raw('LENGTH(body)'))->first();
  }
  
}
```

### Use it

#### Creating, saving and getting entities

Inject the repository through constructor injection, or `App::make`. The methods you see on the PostRepository work like their kin from `Illuminate\Support\Collection`.

```php
class PostController {
  
  function __construct(PostRepository $postRepository) {
    $this->postRepository = $postRepository;
  }
  
  public function create() {
    $post = new Post(Input::get('title'), Input::get('body'));
    $this->postRepository->push($post); // can also use ->persist($post)
    
    // return ... with $post
  }
  
  public function delete($postId) {
    $this->postRepository->forget($postId);
  }
  
  public function homepage($page = 0) {
    $posts = $this->postRepository->slice($page*5, 5);
    
    // return ... with $posts
  }
  
  public function show($postId) {
    $post = $this->postRepository->get($postId); // can also use ->postRepository[$postId]
    
    // return ... with $post
  }
  
}
```

#### Custom repository methods

These work as you'd expect; just call them on the repository.

```php
$longestPost = $this->postRepository->longest();
```

#### Collection-style repository methods

```php

// get all the entities; can be hefty
$allThePosts = $this->postRepository->all();

// turn them all to array form
$allThePostsInArrayForm = $this->postRepository->toArray();

// split posts into parts of 5
$chunks = $this->postRepository->chunk(5);

```

### Using related entities

Clove doesn't currently have built-in tools to load relationships. Instead, I suggest loading them out of their appropriate repository from the related entities:

```php
class Comment implements Entity{ /* ... */ }

class Post implements Entity {
  
  /* ... */
  
  public function getComments() {
    return App::make("CommentRepository")->allFor($this);
  }
  
}

class CommentRepository extends SqlRepository {
  
  /* ... */
  
  public function allFor(Post $post) {
    return $this->getBuilder()->where('post_id', $post->id)->get();
  }
  
}
```

```php
$post = $postRepository->first(); // could also do $postRepository[0]
$comments = $post->getComments();
```

The custom method also provides an opportunity to use joins or other forms of aggressive loading to avoid the N+1 problem if you run into it.

## FAQ

#### Where's all the code?
Clove is starting off as a proof-of-concept specification as I tinker with different implementations.

#### That repository `getFields` looks like a lot of work.
The example above shows the explicit programmatic option; this gives a lot of flexibility. In the future, there will be annotation- and configuration-based mappings available.

#### How is this different from Eloquent?
Eloquent is an ActiveRecord ORM. You build "queries" with it and it gives you magic models that know how to save themselves to your database. Clove uses a Data Mapper, which means you give it vanilla PHP objects to save (with their custom configuration within the repository), and uses a different style to get data back. This gives more separation between your domain logic and how data is saved.

#### Why not use an existing data mapper?
I wanted a fresh view on the pattern (along with a new concept for repositories) - the existing frameworks are great, but building from scratch has its own benefits too.
