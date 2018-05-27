---
layout: article
mathjax: true
title:  "Using Trait in Laravel"
date:   2017-01-16 21:48:30 +1100
tags: [php,Laravel]
category: blog
key: using-trait-in-laravel
---

Recently I got a chance to build a CRM system using [Laravel](https://laravel.com/) and [Backpack](https://backpackforlaravel.com/).

When building the CRM, I learned how to use Trait in PHP. Trait is a feature introduced in PHP 5.4 that let you create a mixin class which can be added to any other class you want. It can't be instantiated or extended, you simply use it. Literally, you just `use` it, and everything in it becomes a part of the class that uses it.

In my case I wanted to setup permissions to each CRUD functionality based on user's role. If the role has read permission, I want to enable `list` and `show` access in Backpack. If the role has write permission, I want to enable `create` and `update`, so on and so forth.

In Backpack you can setup the access in a CRUD controller by doing:

```
$this->crud->allowAccess(['list', 'show']);
$this->crud->denyAccess(['create', 'update', 'delete']);
```

Considering that I have 5 CRUD controllers for all my models, I don't want to repeat this code in every controller.

So I figured that I can do this using a Trait class.

Simply created a Trait class `app/Traits/Access.php`. And in the Trait class I can do this:

```
<?php

namespace App\Traits;

use Illuminate\Support\Facades\Auth;

trait Access
{
    public function setupAccess()
    {
        $user = Auth::user();

        $permissions = [];
        foreach ($user->roles as $role) {
            foreach ($role->permissions()->pluck('name') as $permission) {
                if (!in_array($permission, $permissions)) {
                    array_push($permissions, $permission);
                }
            }
        }

        if (in_array('read', $permissions)) {
            $this->crud->allowAccess(['list', 'show']);
        } else {
            $this->crud->denyAccess(['list', 'show']);
        }

        //... setup other permissions
    }
}
```

What's cool (and weird in some way) about the Trait class is that I can just assume `$this->crud` is available to me, because I will be using the Trait in my CRUD controller, and everything that the CRUD controller has will be available to the Trait class.

Then in my controller I only need to import the Trait, use it then call the method in the constructor, or in Backpack's case, in the `setup` method:

```
use App\Traits\Access;

class UserCrudController extends CrudController
{
    use Access;

    public function setup()
    {
        $this->setupAccess();
    }
    // ... other things
}
```

As a result, I can do the same thing in other controllers without repeating the same code in every controller.

It is quite simple but also pretty powerful.
