# Laravel Photogram

- As like instagram this app can upload photo with caption. User can update profile with profile picture. User also can add post. One user can follow many users profile. When user login he see all the latest post in his profile. <br>
- Laravel Framework, VueJS, Bootstrap. <br>


### Commands
- Install composer, nodejs <br>
- Install Laravel <br>
- php artisan serve <br>
- composer require laravel/ui <br>
- php artisan ui vue --auth <br>
- npm run <br>
- npm run dev <br>
- php artisan storage:link <br>
- composer require intervention/image <br>
- php artisan make:migration creates_profile_user_pivot_table --create profile_user <br>
- composer require laravel/telescope <br>
- php artisan telescope:install <br>

### User.php
```php

```
## Profile

### create_profile_table.php
```php

```
### Profile.php
```php

```
### ProfileController.php
```php

```
### ProfilePolicy.php
```php

```
## Profile Views

### index.blade.php
```php

```
### edit.blade.php
```php

```

## Posts

### create_posts_table.php
```php

```
### Post.php
```php

```
### PostController.php
```php

```
## Post Views

### index.blade.php
```php

```
### create.blade.php
```php

```
### show.blade.php
```php

```

## Follow

### create_profile_user_pivot_table.php
```php

```
### FollowsController.php
```php

```
### FollowButton.vue
```vue

```

### .env






















