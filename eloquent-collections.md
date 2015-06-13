# Eloquent: Collections

- [Introduction](#introduction)
- [Available Methods](#available-methods)
- [Custom Collections](#custom-collections)

<a name="introduction"></a>
## Introduction

All multi-result sets returned by Eloquent are an instance of the `Illuminate\Database\Eloquent\Collection` object, including results retrieved via the `get` method or accessed via a relationship. The Eloquent collection object extends the Laravel [base collection](/{{version}}/collections), so it naturally inherits dozens of methods used to fluently work with the underlying array of Eloquent models.

Of course, all collections also serve as iterators, allowing you to loop over them as if they were simple PHP arrays:

	$users = App\User::where('active', 1)->get();

	foreach ($users as $user) {
		echo $user->name;
	}

However, collections are much more powerful than arrays and expose a variety of map / reduce operations using an intuitive interface. For example, let's remove all inactive models and gather the first name for each remaining user:

	$users = App\User::where('active', 1)->get();

	$names = $users->reject(function ($user) {
		return $user->active === false;
	})
	->map(function ($user) {
		return $user->name;
	});

<a name="available-methods"></a>
## Available Methods

### The Base Collection

All Eloquent collections extend the base [Laravel collection](/{{version}}/collections) object; therefore, they inherit all of the powerful methods provided by the base collection class:

<style>
	#collection-method-list > p {
		column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
		column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
	}

	#collection-method-list a {
		display: block;
	}
</style>

<div id="collection-method-list" markdown="1">
[all](/{{version}}/collections#method-all)
[chunk](/{{version}}/collections#method-chunk)
[collapse](/{{version}}/collections#method-collapse)
[contains](/{{version}}/collections#method-contains)
[count](/{{version}}/collections#method-count)
[diff](/{{version}}/collections#method-diff)
[each](/{{version}}/collections#method-each)
[filter](/{{version}}/collections#method-filter)
[first](/{{version}}/collections#method-first)
[flatten](/{{version}}/collections#method-flatten)
[flip](/{{version}}/collections#method-flip)
[forget](/{{version}}/collections#method-forget)
[forPage](/{{version}}/collections#method-forpage)
[get](/{{version}}/collections#method-get)
[groupBy](/{{version}}/collections#method-groupby)
[has](/{{version}}/collections#method-has)
[implode](/{{version}}/collections#method-implode)
[intersect](/{{version}}/collections#method-intersect)
[isEmpty](/{{version}}/collections#method-isempty)
[keyBy](/{{version}}/collections#method-keyby)
[keys](/{{version}}/collections#method-keys)
[last](/{{version}}/collections#method-last)
[map](/{{version}}/collections#method-map)
[merge](/{{version}}/collections#method-merge)
[pluck](/{{version}}/collections#method-pluck)
[pop](/{{version}}/collections#method-pop)
[prepend](/{{version}}/collections#method-prepend)
[pull](/{{version}}/collections#method-pull)
[push](/{{version}}/collections#method-push)
[put](/{{version}}/collections#method-put)
[random](/{{version}}/collections#method-random)
[reduce](/{{version}}/collections#method-reduce)
[reject](/{{version}}/collections#method-reject)
[reverse](/{{version}}/collections#method-reverse)
[search](/{{version}}/collections#method-search)
[shift](/{{version}}/collections#method-shift)
[shuffle](/{{version}}/collections#method-shuffle)
[slice](/{{version}}/collections#method-slice)
[sort](/{{version}}/collections#method-sort)
[sortBy](/{{version}}/collections#method-sortby)
[sortByDesc](/{{version}}/collections#method-sortbydesc)
[splice](/{{version}}/collections#method-splice)
[sum](/{{version}}/collections#method-sum)
[take](/{{version}}/collections#method-take)
[toArray](/{{version}}/collections#method-toarray)
[toJson](/{{version}}/collections#method-tojson)
[transform](/{{version}}/collections#method-transform)
[unique](/{{version}}/collections#method-unique)
[values](/{{version}}/collections#method-values)
[where](/{{version}}/collections#method-where)
[whereLoose](/{{version}}/collections#method-whereloose)
[zip](/{{version}}/collections#method-zip)
</div>

<a name="custom-collections"></a>
## Custom Collections

If you need to use a custom `Collection` object with your own extension methods, you may override the `newCollection` method on your model:

	<?php

	namespace App;

	use App\CustomCollection;
	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * Create a new Eloquent Collection instance.
		 *
		 * @param  array  $models
		 * @return \Illuminate\Database\Eloquent\Collection
		 */
		public function newCollection(array $models = [])
		{
			return new CustomCollection($models);
		}
	}

Once you have defined a `newCollection` method, you will receive an instance of your custom collection anytime Eloquent returns a `Collection` instance of that model. If you would like to use a custom collection for every model in your application, you should override the `newCollection` method on a model base class that is extended by all of your models.
