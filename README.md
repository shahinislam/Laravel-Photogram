# Laravel Photogram

- This project is clone as like instagram. User have to login and register. It will be able to follow/unfollow. Also able to edit profile and profile image. Add new post with choose image option. News feed based on following people. <br>
- Laravel 7 | VueJS | Bootstrap. <br>


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

### web.php
```php
Auth::routes();

Route::get('/email', function () {
   return new \App\Mail\NewUserWelcomeMail();
});

Route::post('/follow/{user}', 'FollowsController@store');

Route::get('/', 'PostsController@index');
Route::get('/p/create', 'PostsController@create');
Route::get('/p/{post}', 'PostsController@show');
Route::post('/p', 'PostsController@store');

Route::get('/profile/{user}', 'ProfilesController@index')->name('profile.show');
Route::get('/profile/{user}/edit', 'ProfilesController@edit')->name('profile.edit');
Route::patch('/profile/{user}', 'ProfilesController@update')->name('profile.update');

```
![1  login](https://user-images.githubusercontent.com/33843231/80362664-d3581280-88a4-11ea-89ef-90fa160bdce7.png)

### User.php
```php
    protected static function boot()
    {
        parent::boot();
        static::created(function ($user){
           $user->profile()->create([
               'title' => $user->username,
           ]);

            Mail::to($user->email)->send(new NewUserWelcomeMail());
        });
    }

    public function posts()
    {
        return $this->hasMany(Post::class)->orderBy('created_at', 'DESC');
    }

    public function profile()
    {
        return $this->hasOne(Profile::class);
    }

    public function following()
    {
        return $this->belongsToMany(Profile::class);
    }
```
## Profile

### create_profiles_table.php
```php
        $table->id();
        $table->unsignedBigInteger('user_id');
        $table->string('title')->nullable();
        $table->text('description')->nullable();
        $table->string('url')->nullable();
        $table->string('image')->nullable();
        $table->timestamps();

        $table->index('user_id');
```
### Profile.php
```php
    protected $guarded = [];

    public function profileImage()
    {
        $imagePath = ($this->image) ? $this->image : 'profile/Bsk7KeJihkFDHyXnGQiG3ad6IbcdUfdsihbQYMc3.png';

        return '/storage/' . $imagePath;
    }

    public function user()
    {
       return $this->belongsTo(User::class);
    }

    public function followers()
    {
        return $this->belongsToMany(User::class);
    }
```
### ProfileController.php
```php
    public function index(User $user)
    {
        $follows = (auth()->user()) ? auth()->user()->following->contains($user->id) : false;

        $postCount = Cache::remember(
            'count.posts.' . $user->id,
            now()->addSeconds(30),
            function () use($user) {
                return $user->posts->count();
        });
        $followersCount = Cache::remember(
            'count.followers.' . $user->id,
            now()->addSeconds(30),
            function () use($user) {
                return $user->profile->followers->count();
        });
        $followingCount = Cache::remember(
            'count.following.' . $user->id,
            now()->addSeconds(30),
            function () use($user) {
                return $user->following->count();
        });

        return view('profiles.index',
            compact('user', 'follows', 'postCount', 'followersCount', 'followingCount'));
    }

    public function edit(User $user)
    {
        $this->authorize('update', $user->profile);

        return view('profiles.edit', compact('user'));
    }

    public function update(User $user)
    {
        $this->authorize('update', $user->profile);

        $data = request()->validate([
            'title' => 'required',
            'description' => 'required',
            'url' => 'url',
            'image' => '',
        ]);


        if(request('image')){
            $imagePath = request('image')->store('profile', 'public');
            $image = Image::make(public_path("storage/{$imagePath}"))->fit(1000, 1000);
            $image->save();

            $imageArray = ['image' => $imagePath];
        }

        auth()->user()->profile->update(array_merge(
            $data,
            $imageArray ?? []
        ));

        return redirect("/profile/{$user->id}");
    }
```
### ProfilePolicy.php
```php
    public function update(User $user, Profile $profile)
    {
        return $user->id == $profile->user_id;
    }
```
## Profile Views

![3  Profileindex](https://user-images.githubusercontent.com/33843231/80362612-bf141580-88a4-11ea-8de4-6a0911902073.png)

### index.blade.php
```php
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row">
        <div class="col-3 p-5">
            <img src="{{ $user->profile->profileImage() }}" class="rounded-circle w-100">
        </div>
        <div class="col-9 pt-5">
            <div class="align-items-baseline d-flex justify-content-between">

                <div class="d-flex align-items-center pb-3">
                    <div class="h4">{{ $user->username }}</div>

                    <follow-button follows="{{ $follows }}" user-id="{{ $user->id }}"></follow-button>
                </div>

                @can('update', $user->profile)
                    <a href="/p/create" class="btn btn-success">Add New Post</a>
                @endcan
            </div>

            @can('update', $user->profile)
                <a href="/profile/{{ $user->id }}/edit">Edit Profile</a>
            @endcan

            <div class="d-flex">
                <div class="pr-5"><strong>{{ $postCount }}</strong> posts</div>
                <div class="pr-5"><strong>{{ $followersCount }}</strong> followers</div>
                <div class="pr-5"><strong>{{ $followingCount }}</strong> following</div>
            </div>
            <div class="pt-4 font-weight-bold">{{ $user->profile->title }}</div>
            <div>{{ $user->profile->description }}</div>
            <div><a href="#">{{ $user->profile->url }}</a></div>
        </div>
    </div>
    <div class="row pt-5">
        @foreach($user->posts as $post)
            <div class="col-4 pb-4">
                <a href="/p/{{ $post->id }}">
                    <img src="/storage/{{ $post->image }}" class="w-100">
                </a>
            </div>
        @endforeach
    </div>
</div>

@endsection

```
### edit.blade.php
![4  ProfileEdit](https://user-images.githubusercontent.com/33843231/80362605-bc192500-88a4-11ea-808c-e80b69a68d8e.png)
```php
@extends('layouts.app')

@section('content')
    <div class="container">
        <form action="/profile/{{ $user->id }}" enctype="multipart/form-data" method="post">

            @csrf
            @method('PATCH')

            <div class="row">
                <div class="col-8 offset-2">

                    <div class="row">
                        <h1>Edit Profile</h1>
                    </div>
                    <div class="form-group row">
                        <label for="title" class="col-md-4 col-form-label">Title</label>

                        <input id="title"
                               name="title"
                               type="text"
                               class="form-control @error('title') is-invalid @enderror"
                               value="{{ old('title') ?? $user->profile->title }}"
                               autocomplete="title" autofocus>

                        @error('title')
                            <span class="invalid-feedback" role="alert">
                                <strong>{{ $message }}</strong>
                            </span>
                        @enderror
                    </div>
                    <div class="form-group row">
                        <label for="description" class="col-md-4 col-form-label">Description</label>

                        <input id="description"
                               name="description"
                               type="text"
                               class="form-control @error('description') is-invalid @enderror"
                               value="{{ old('description') ?? $user->profile->description }}"
                               autocomplete="description" autofocus>

                        @error('description')
                            <span class="invalid-feedback" role="alert">
                                <strong>{{ $message }}</strong>
                            </span>
                        @enderror
                    </div>
                    <div class="form-group row">
                        <label for="url" class="col-md-4 col-form-label">URL</label>

                        <input id="url"
                               name="url"
                               type="text"
                               class="form-control @error('url') is-invalid @enderror"
                               value="{{ old('url') ?? $user->profile->url }}"
                               autocomplete="url" autofocus>

                        @error('url')
                            <span class="invalid-feedback" role="alert">
                                <strong>{{ $message }}</strong>
                            </span>
                        @enderror
                    </div>
                    <div class="row form-group">
                        <label for="image" class="col-md-4 col-form-label">Profile Image</label>
                        <input type="file" class="form-control-file" id="image" name="image">
                        @error('image')
                        <strong>{{ $message }}</strong>
                        @enderror
                    </div>
                    <div class="row pt-4">
                        <button class="btn btn-primary">Save Profile</button>
                    </div>
                </div>
            </div>

        </form>
    </div>
@endsection
```

## Posts

### create_posts_table.php
```php
            $table->id();
            $table->unsignedBigInteger('user_id');
            $table->text('caption');
            $table->string('image');
            $table->timestamps();

            $table->index('user_id');
```
### Post.php
```php
    protected $guarded = [];

    public function user()
    {
        return $this->belongsTo(User::class);
    }
```
### PostController.php
```php

public function __construct()
    {
        $this->middleware('auth');
    }

    public function index()
    {
        $users = auth()->user()->following()->pluck('profiles.user_id');

        $posts = Post::whereIn('user_id', $users)->with('user')->latest()->paginate(5);

        return view('posts.index', compact('posts'));
    }

    public function create()
    {
        return view('posts.create');
    }

    public function store()
    {
        $data = request()->validate([
            'caption' => 'required',
            'image' => ['required', 'image'],
        ]);

        $imagePath = request('image')->store('uploads', 'public');

        $image = Image::make(public_path("storage/{$imagePath}"))->fit(1200, 1200);
        $image->save();

        auth()->user()->posts()->create([
            'caption' => $data['caption'],
            'image' => $imagePath,
        ]);

        return redirect('/profile/' . auth()->user()->id);
    }

    public function show(Post $post)
    {
        return view('posts.show', compact('post'));
    }
```
## Post Views

![2  PostIndex](https://user-images.githubusercontent.com/33843231/80362635-c6d3ba00-88a4-11ea-88fd-5de742747d0d.png)

### index.blade.php
```php
@extends('layouts.app')

@section('content')
    <div class="container">
        @foreach($posts as $post)
            <div class="row">
                <div class="col-6 offset-3">
                    <a href="/profile/{{ $post->user->id }}">
                        <img src="/storage/{{ $post->image }}" class="w-100">
                    </a>
                </div>
            </div>
            <div class="row pt-2 pb-4">
                <div class="col-6 offset-3">
                    <div>
                        <p>
                            <span class="font-weight-bold">
                                <a href="/profile/{{ $post->user->id }}">
                                    <span class="text-dark">{{ $post->user->username }}</span>
                                </a>
                            </span>
                            {{ $post->caption }}
                        </p>
                    </div>
                </div>
            </div>
        @endforeach

        <div class="row">
            <div class="col-12 d-flex justify-content-center">
                {{ $posts->links() }}
            </div>
        </div>
    </div>
@endsection
```
### create.blade.php

![5  PostCreate](https://user-images.githubusercontent.com/33843231/80362680-de12a780-88a4-11ea-8b75-92c25d8646e2.png)

```php
@extends('layouts.app')

@section('content')
    <div class="container">
        <form action="/p" enctype="multipart/form-data" method="post">

            @csrf

            <div class="row">
                <div class="col-8 offset-2">

                    <div class="row">
                        <h1>Add New Post</h1>
                    </div>
                    <div class="form-group row">
                        <label for="caption" class="col-md-4 col-form-label">Post Caption</label>

                        <input id="caption"
                               name="caption"
                               type="text"
                               class="form-control @error('caption') is-invalid @enderror"
                               value="{{ old('caption') }}"
                               autocomplete="caption" autofocus>

                        @error('caption')
                            <span class="invalid-feedback" role="alert">
                                <strong>{{ $message }}</strong>
                            </span>
                        @enderror
                    </div>
                    <div class="row form-group">
                        <label for="image" class="col-md-4 col-form-label">Post Image</label>
                         <input type="file" class="form-control-file" id="image" name="image">
                        @error('image')
                            <strong>{{ $message }}</strong>
                        @enderror
                    </div>
                    <div class="row pt-4">
                        <button class="btn btn-primary">Add New Post</button>
                    </div>
                </div>
            </div>

        </form>
    </div>
@endsection
```
### show.blade.php

![6  PostShow](https://user-images.githubusercontent.com/33843231/80362668-d4893f80-88a4-11ea-940b-70dc32680b43.png)

```php
    <div class="container">
        <div class="row">
            <div class="col-8">
                <img src="/storage/{{ $post->image }}" class="w-100">
            </div>

            <div class="col-4">
                <div>
                    <div class="d-flex align-content-center">
                        <div class="pr-3">
                            <img src="{{ $post->user->profile->profileImage() }}" style="max-width: 40px;" class="w-100 rounded-circle">
                        </div>
                        <div>
                            <div class="font-weight-bold">
                                <a href="/profile/{{ $post->user->id }}">
                                    <span class="text-dark">{{ $post->user->username }}</span>
                                </a>
                                <a href="#" class="pl-2">Follow</a>
                            </div>
                        </div>
                    </div>
                    <hr>
                    <p>
                        <span class="font-weight-bold">
                            <a href="/profile/{{ $post->user->id }}">
                                <span class="text-dark">{{ $post->user->username }}</span>
                            </a>
                        </span>
                        {{ $post->caption }}
                    </p>
                </div>
            </div>
        </div>
    </div>
```

## Follow

### create_profile_user_pivot_table.php
```php
            $table->id();
            $table->unsignedBigInteger('profile_id');
            $table->unsignedBigInteger('user_id');

            $table->timestamps();
```
### FollowsController.php
```php
class FollowsController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
    }

    public function store(User $user)
    {
        return auth()->user()->following()->toggle($user->profile);
    }
}
```
### FollowButton.vue
```vue
<template>
    <div>
        <button @click="followUser" v-text="buttonText" class="btn btn-primary ml-4"></button>
    </div>
</template>

<script>
    export default {

        props: ['userId', 'follows'],

        mounted() {
            console.log('Component mounted.')
        },

        data: function () {
            return {
                status: this.follows,
            }
        },

        methods: {
            followUser: function () {
                axios.post('/follow/' + this.userId)
                    .then(response => {
                        this.status = ! this.status;

                        console.log(response.data);
                    })
                    .catch(errors => {
                        if(errors.response.status == 401) {
                            window.location = '/login';
                        }
                    });
            }
        },

        computed: {
            buttonText() {
                return (this.status) ? 'Unfollow' : 'Follow';
            }
        }
    }
</script>

```

### .env

```
APP_NAME=PhotoGram
APP_ENV=local
APP_KEY=base64:VKTXAPUMP9/utlPWipfh+uZbfycKj7KLmMxMVcPJvfg=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack

DB_CONNECTION=sqlite

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=78c155999374ef
MAIL_PASSWORD=43ac4fd3dc5fcf
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=shahin@gmail.com
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

```






















