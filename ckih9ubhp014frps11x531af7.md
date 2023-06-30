---
title: "Two Users Authentication with Laravel"
datePublished: Wed Dec 09 2020 10:30:19 GMT+0000 (Coordinated Universal Time)
cuid: ckih9ubhp014frps11x531af7
slug: two-users-authentication-with-laravel
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1607509789933/DXyXan3yh.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1607509800984/ughWm5Zs1.png
tags: laravel, php, hashnode, 100daysofcode, devcommunity

---

## Short Story
As a developer, you may want to build an application that can accommodate different categories of users. For example, you want to work on a marketplace eCommerce website, where you have the customers and the business entrepreneur or a learning platform, where you have the teachers and the students. Now this are different categories of users performing different roles with different functionalities and it is important to have two separate tables to hold this users data, therefore you will need to have two different guards to deal with there authentication.

## Laravel Guard
Guards in Laravel are used to define how users are Authenticated for each request, either by session or by token.
Providers on the other hand, determine how the users are retrieved from the persistent storage. Basically, the provider just determine the model you are using to for the authentication. the User model is the default.

The default guard your Laravel application comes with on installation, is the a `session` guard. It maintain state using sessions and cookies. So when ever you call the function `Auth::user();` you are automatically using the default provider. So if your application for example has  students and teachers as it users, then you most have created a model for both. you can easily just create provider and guard for the two models (`Student` and `Teachers` ).



 [*Laravel Docs*](https://laravel.com/docs/7.x/authentication#:~:text=Guards%20define%20how%20users%20are,retrieved%20from%20your%20persistent%20storage.)  

## Configuration. 
First Step is to configure your application [ create a new provider for the student and teacher model, and register them as guards ]. to do this, you will move to `Auth.php` file in the `config` folder. it's actually just a php file returning your configurations. so by default you will see the `web` and `api` guard with the users provider for the both of them.

```
    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
        'api' => [
            'driver' => 'token',
            'provider' => 'users',
            'hash' => false,
        ],
    ],
``` 
You can create more guards by adding an array in the format below.
```
    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
      'student' => [
            'driver' => 'session',
            'provider' => 'students',
        ],
      'teacher' => [
            'driver' => 'session',
            'provider' => 'teachers',
        ],
        'api' => [
            'driver' => 'token',
            'provider' => 'users',
            'hash' => false,
        ],
    ],
``` 
While student and teacher are the recently added guards, their respective providers are students and teachers. which also need to be configured as we are yet to do that. 

So, to create the `providers`, you will see a providers array something like this:

```
'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\Models\User::class,
        ],
    ],

``` 
Just like the guard, you can just create new arrays using the same format as it is above, with the name of the providers as indicated in the guard.

```
'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\Models\User::class,
        ],
       'students' => [
            'driver' => 'eloquent',
            'model' => App\Models\Student::class,
        ],
       'teacher' => [
            'driver' => 'eloquent',
            'model' => App\Models\Teacher::class,
        ],
    ],
``` 
the `model` is the class you wish the users to be authenticated from. 

## Functions
Now after the configuration you may want to attempt the `login`, `logout` or any other function. well it's really easy. just as before, but this time you have to specify the `guard` you want the user to be authenticated by. 
### example

```
  Login function:
Auth::guard('student')->attempt([
   'email'=>$request['email'],
    'password'=>$request['password']
]);

Logout function:
Auth::guard('student')->logout();

get user information:
Auth::guard("student")->user();

``` 
## Middleware
You may want to set up an `auth middleware` to protect a particular `route`. for example. students are not to have access to to the teachers dashboard page.

you can easily do that by adding `auth:<guard name>` to your `route`.

#### example


```
Route::get('/dashboard', [
App\Http\Controllers\StudentsController::class, 'index'
])->middleware('auth:student');
``` 
### or

```
Route::post('/dashboard', [ 
    'uses'=> 'StudtenController@index',
    'as' => 'dashboard',
    'middleware'=> 'auth:student'

]);
``` 









