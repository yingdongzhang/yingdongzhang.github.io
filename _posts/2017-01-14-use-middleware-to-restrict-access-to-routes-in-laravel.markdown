---
layout: post
title:  "Use Middleware to restrict access to routes in Laravel"
date:   2017-01-14 20:30:30 +1100
categories: [post,php,Laravel]
comments: true
---

Sometimes you want to restrict users with certain roles to access certain routes. This is how I learned to do it with middleware.

Consider the situation where you don't want users with role `user` to access the following routes: "permission", "user", and "role", only users with role `admin` can access them.

In Laravel you can do this by creating a middleware, and apply it on the routes you want to perform role check.

The following example is using [Backpack](https://backpackforlaravel.com/) and [Backpack PermissionManager](https://github.com/Laravel-Backpack/PermissionManager).

First I created a middleware called `RoleMiddleware`. And the code is quite simple:

```
<?php

namespace App\Http\Middleware;

use Closure;
use Auth;

class RoleMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next, ...$roles)
    {
        if (! $request->user()->hasAnyRole($roles)) {
            abort(403);
        }

        return $next($request);
    }
}
```

In my usage the third parameter is an array of roles but it can also be a single variable if it suits your needs. Don't forget to register your middleware in `Kernel.php`:

```
protected $routeMiddleware = [
    ...
    'role' => \App\Http\Middleware\RoleMiddleware::class,
];
```

Then in my routes file, I can restrict the access for certain roles by doing:

```
Route::group(['middleware' => 'role:super admin,admin'], function () {
    CRUD::resource('permission', '\Backpack\PermissionManager\app\Http\Controllers\PermissionCrudController');
    CRUD::resource('user', '\Backpack\PermissionManager\app\Http\Controllers\UserCrudController');
    CRUD::resource('role', '\Backpack\PermissionManager\app\Http\Controllers\RoleCrudController');
});
```

Here I say for the following routes only users with role `super admin` and/or `admin` can access the following routes. Then it hits the middleware, the variable `$roles` will be `['super admin', 'admin']`

Of course you still want both normal users and admins to access the non-sensitive areas of your application. In this case you simply apply the same logic on the other routes:

```
Route::group(['middleware' => 'role:super admin,admin,user'], function () {
    Route::get('dashboard', '\Backpack\Base\app\Http\Controllers\AdminController@dashboard');
    CRUD::resource('contact', '\App\Http\Controllers\Admin\ContactCrudController');
    // ... any other route that is public to every role
});
```
