# Eloquent Viewable

[![Packagist](https://img.shields.io/packagist/v/cyrildewit/eloquent-viewable.svg?style=flat-square)](https://packagist.org/packages/cyrildewit/eloquent-viewable)
[![Travis branch](https://img.shields.io/travis/cyrildewit/eloquent-viewable/2.0.svg?style=flat-square)](https://travis-ci.org/cyrildewit/eloquent-viewable)
[![StyleCI](https://styleci.io/repos/94131608/shield?style=flat-square)](https://styleci.io/repos/94131608)
[![Codecov branch](https://img.shields.io/codecov/c/github/cyrildewit/eloquent-viewable/2.0.svg?style=flat-square)](https://codecov.io/gh/cyrildewit/eloquent-viewable)
[![Total Downloads](https://img.shields.io/packagist/dt/cyrildewit/eloquent-viewable.svg?style=flat-square)](https://packagist.org/packages/cyrildewit/eloquent-viewable)
[![license](https://img.shields.io/github/license/cyrildewit/eloquent-viewable.svg?style=flat-square)](https://github.com/cyrildewit/eloquent-viewable/blob/master/LICENSE.md)

> **Note:** This is an unstable branch!

This Laravel >= 5.5 package allows you to associate views with Eloquent models.

Once installed you can do stuff like this:

```php
// Return total views count
$post->views()->count();

// Return total views count that have been made since 20 February 2017
$post->views()->period(Period::since('2017-02-20'))->count();

// Return total views count that have been made between 2014 and 216
$post->views()->period(Period::create('2014', '2016'))->count();

// Return total unique views count (based on visitor cookie)
$post->views()->unique()->count();

// Record a new view
$post->views()->record();

// Record a new view with session delay between views
$post->views()->sessionDelay(now()->addHours(2))->record();
```

## Overview

Eloquent Vieable allows you to associate views with Eloquent models. It's designed with simplicity, performance and

Eloquent Viewable is a flexible and easy to use Laravel package to associate views with Eloquent Models. It's designed for large and small projects. Instead of having a simple counter that increments by each view, this package will provide you a full history of the views.

This package is not built with the intent to collect analytical data. It is made to simply store the views of a Laravel Eloquent model. You would this package for models like: Post, Video, Course and Hotel, but of course, you can use this package as you want.

### Features

Here are some of the main features:

* Associate views with Eloquent models
* Get total views count
* Get views count of a specific period
* Get unique views count
* Smart views count cacher helper

## Documentation

In this documentation, you will find some helpful information about the use of this Laravel package.


### Table of contents

1. [Getting Started](#getting-started)
    * [Requirements](#requirements)
    * [Installation](#installation)
2. [Usage](#usage)
    * [Preparing your model](#preparing-your-model)
    * [Recording views](#storing-views)
    * [Storing views with expiry date](#storing-views-with-expiry-date)
    * [Retrieving views counts](#retrieving-views-counts)
    * [Order models by views count](#order-models-by-views-count)
    * [`Views` helper](#views-helper)
3. [Optomizing](#optimizing)
    * [Queuing views](#queueing-views)
    * [Caching view counts](#caching-view-counts)
4. [Extending](#extending)
    * [Using your own model](#using-your-own-model)
    * [Using a custom crawler detector](#using-a-custom-crawler-detector)
5. [Recipes](#recipes)
    * [Creating helper methods for frequently used period formats](#creating-helper-methods-for-frequently-used-period-formats)

## Getting Started

### Requirements

This package requires **PHP 7+** and **Laravel 5.5+**.

Lumen is not supported!

#### Version information

| Version | Illuminate | Status         | PHP Version |
|---------|------------|----------------|-------------|
| 3.0     | 5.5 - 5.7  | _In Development_ | >= 7.1.0    |
| 2.0     | 5.5 - 5.7  | Active support | >= 7.0.0    |
| 1.0     | 5.5 - 5.6  | Bug fixes only | >= 7.0.0    |

### Installation

First, you need to install the package via Composer:

```winbatch
composer require cyrildewit/eloquent-viewable
```

Secondly, if you want to make some basic changes like giving the `views` table a different name or creating the table on a different connection, you can configure that by publishing the config file with:

```winbatch
php artisan vendor:publish --provider="CyrildeWit\EloquentViewable\EloquentViewableServiceProvider" --tag="config"
```

Alternatively, if you want to make bigger changes in the migrations, you can publish them using:

```winbatch
php artisan vendor:publish --provider="CyrildeWit\EloquentViewable\EloquentViewableServiceProvider" --tag="migrations"
```

Finally, you need to run the `migrate` command:

```winbatch
php artisan migrate
```

#### Register service provider manually

If you like to register packages manually, you can add the following provider to your application's providers list.

```php
// config/app.php

'providers' => [
    // ...
    CyrildeWit\EloquentViewable\EloquentViewableServiceProvider::class,
];
```

*Note:* this can be done via automatic package discovery!

## Usage

### Preparing your model

To associate views with a model, the model must implement the following interface and trait.

```php
use Illuminate\Database\Eloquent\Model;
use CyrildeWit\EloquentViewable\HasViews;
use CyrildeWit\EloquentViewable\HasViewsTrait;

class Post extends Model implements HasViews
{
    use HasViewsTrait;

    // ...
}
```

### Recording views

To make a view record, you can call the `recordView` method on your model.

```php
$post->recordView();
```

The best place where you should place it is inside your controller. For example:

```php
// PostController.php
public function show(Post $post)
{
    $post->recordView();

    return view('post.show', compact('post'));
}
// ...
```

**Note:** This package filters out crawlers by default. Be aware of this when testing, because Postman is for example also a crawler.

### Recording views with session delays

Adding a delay between view records is easiliy achievable by calling

If you want to add a delay between views from the same session, you can use the available `addViewWithExpiryDate` method on your viewable model.

```php
$post->addViewWithExpiryDate(Carbon::now()->addHours(2));
```

This method will add a new view to your model and it will add a record in the user's session. If you call this method multiple times, you will see that views count will not increment. After the current date time has passed the expiry date, a new view will be stored.

### Retrieving views counts

After adding the `Viewable` trait to your model, you will be able to call `getViews()` and `getUniqueViews()` on your viewable model. Both methods accepts an optional `Period` instance.

```php
/**
 * Get the total number of views.
 *
 * @param  \CyrildeWit\EloquentViewable\Support\Period
 * @return int
 */
public function getViews($period = null): int;

/**
 * Get the total number of unique views.
 *
 * @param  \CyrildeWit\EloquentViewable\Support\Period
 * @return int
 */
public function getUniqueViews($period = null) : int;
```

#### Period class

_Be aware that the following code isn't valid PHP syntax!_

```php
// Create a new Period instance.
Period::create(DateTime $startDateTime = null, DateTime $endDateTime = null);

// Create a new Period instance with only a start date time.
Period::since(DateTime $startDateTime);

// Create a new Period instance with only a end date time.
Period::upto(DateTime $endDateTime);
```

```php
// Period instance with a start date time of today minus the given days.
Period::pastDays(int $days);

// Period instance with a start date time of today minus the given weeks.
Period::pastWeeks(int $weeks);

// Period instance with a start date time of today minus the given months.
Period::pastMonths(int $months);

// Period instance with a start date time of today minus the given years.
Period::pastYears(int $years);
```

```php
// Period instance with a start date time of now minus the given seconds.
Period::subSeconds(int $seconds);

//Period instance with a start date time of now minus the given minutes.
Period::subMinutes(int $minutes);

// Period instance with a start date time of now minus the given hours.
Period::subHours(int $hours);

// Period instance with a start date time of now minus the given days.
Period::subDays(int $days);

// Period instance with a start date time of now minus the given weeks.
Period::subWeeks(int $weeks);

// Period instance with a start date time of now minus the given months.
Period::subMonths(int $months);

// Period instance with a start date time of now minus the given years.
Period::subYears(int $years);
```

### Order models by views count

The viewable trait adds two scopes to your model: `scopeOrderByViews` and `scopeOrderByUniqueViews`.

#### Retrieve Viewable models by views count

```php
Post::orderByViews()->get(); // descending
Post::orderByViews('asc')->get(); // ascending
```

#### Retrieve Viewable models by unique views count

```php
Post::orderByUniqueViews()->get(); // descending
Post::orderByUniqueViews('asc')->get(); // ascending
```

### `Views` helper

Namespace: `use CyrildeWit\EloquentViewable\Views`.

#### Saving views

```php
Views::create($post)->addView();
```

#### Saving views with expiry date

```php
Views::create($post)->addViewWithExpiryDate(Carbon::now()->addHours(2));
```

#### Retrieving views counts

```php
Views::create($post)->getViews();
```

#### Get views by viewable type

If you want to know how many views a specific viewable type has, you can use the static `getViewsByType` method on the `Views` class.

```php
Views::getViewsByType(Post::class);
Views::getViewsByType('App\Post');
```

You can also pass an instance of an Eloquent model. It will get the fully qualified class name by calling the `getMorphClass` method on the model.

```php
Views::getViewsByType($post);
```

#### Get most viewed viewables by type

To get a collection of Eloquent models sorted by most views and type, you can use the provided static `getMostViewedByType` method. It accepts a limit as second argument.

Please note that this method does the same as `Post::orderByViews()->take(10);`.

```php
// Get top 10 most viewed by type
Views::getMostViewedByType(Post::class, 10);
Views::getMostViewedByType('App\Post', 10);

// and by passing an instance of an eloquent model
Views::getMostViewedByType($post, 10);
```

#### Get least viewed viewables by type

Please note that this method does the same as `Post::orderByViews('asc')->take(10);`.

```php
// Get top 10 least viewed by type
Views::getleastViewedByType(Post::class, 10);
Views::getleastViewedByType('App\Post', 10);

// and by passing an instance of an eloquent model
Views::getleastViewedByType($post, 10);
```

#### Get the views count of viewables per period

If you want to get a collection of views per period, you can call the static `getViewsPerPeriod` method on the `Views` class.

The parameter signature looks like this:

```php
getViewsPerPeriod(string $dimension, $period, $viewableType = null): Collection;
```

The first argument should be one of the following options:

* minute
* hour
* day
* week
* month
* year

The second argument should be an instance of `\CyrildeWit\EloquentViewable\Support\Period`. More information about this class can be found [here](period-class).

The third argument is optional. It can be used to get only the views of a specific type. For example `App\Post`. It should be a fully qualified class name. You can retrieve it using the `getMorphClass` method on an Eloquent model.

##### Example

The following code example will return the total number of views per week and between two weeks ago and today. In this example the of today is `2018-07-08 00:00:00`.

```php
Views::getViewsPerPeriod('week', Period::pastWeeks(2))
```

Result:

```text
[
    ['week' => '31', 'views' => 526],
    ['week' => '32', 'views' => 630]
]
```

#### Get the unique views count of viewables per period

If you want to get a collection of unique views per period, you can call the static `getUniqueViewsPerPeriod` method on the `Views` class.

It has the same parameter signature as the static `getViewsPerPeriod` method.

See the section here above for information about how you can use this method.

## Configuration

### Queue the ProcessView job

When calling the `->addView()` method on your model, it will save a new view in the database with some data. Because this can slow down your application, you can turn queuing on by changing the value of `store_new_view` under `jobs` in the configuration file. Make sure that your app is ready for queuing. If not, see the official [Laravel documentation](https://laravel.com/docs/5.6/queues) for more information!

### Extending

If you want to extend or replace one of the core classes with your own implementations, you can override them:

* `CyrildeWit\EloquentViewable\View`
* `CyrildeWit\EloquentViewable\ViewableService`
* `CyrildeWit\EloquentViewable\CrawlerDetector\CrawlerDetectAdapter`

_**Note:** Don't forget that all custom classes must implement their original interfaces_

#### Replace `View` model with custom implementation

```php
$this->app->bind(
    \CyrildeWit\EloquentViewable\Contracts\View::class,
    \App\CustomView::class
);
```

#### Replace `ViewableService` service with custom implementation

```php
$this->app->singleton(
    \CyrildeWit\EloquentViewable\Contracts\ViewableService::class,
    \App\Services\CustomViewableService::class
);
```

#### Replace `CrawlerDetectAdapter` class with custom implementation

```php
$this->app->singleton(
    \CyrildeWit\EloquentViewable\Contracts\CrawlerDetector::class,
    \App\Services\CrawlerDetector\CustomAdapter::class
);
```

## Tips &amp; Tricks

### Creating helper methods for frequently used period formats

#### App\Post

```php
public function getViewsSince(DateTime $sinceDateTime)
{
    return $this->getViews(Period::since($sinceDateTime));
}

public function getViewsUpto(DateTime $uptoDateTime)
{
    return $this->getViews(Period::upto($uptoDateTime));
}

public function getViewsBetween(DateTime $sinceDateTime, DateTime $uptoDateTime)
{
    return $this->getViews(Period::create($sinceDateTime, $uptoDateTime));
}

public function getViewsInPastDays(int $days)
{
    return $this->getViews(Period::pastDays($days));
}
```

#### resources/views/post/show.blade.php

```html
Page views since 2014: {{ $post->getViewsSince(Carbon::create(2014)) }}
Page views upto 2016: {{ $post->getViewsUpto(Carbon::create(2016)) }}
Page views between 2016 - 2018: {{ $post->getViewsBetween(Carbon::create(2016), Carbon::create(2018)) }}

Page views in the past 5 days: {{ $post->getViewsInPastDays(5) }}
```

## Upgrading

Please see [UPGRADING](UPGRADING.md) for detailed upgrade guide.

## Changelog

Please see [CHANGELOG](CHANGELOG-2.0.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Credits

* [Cyril de Wit](https://github.com/cyrildewit)
* [All Contributors](../../contributors)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.
