
1. `composer create-project laravel/laravel photoshow`
2. Make controllers
	1. `php artisan make:controller AlbumsController`
	2. `php artisan make:controller PhotoController`
3. Make models
	1. `php artisan make:model Album -m`
	2. `php artisan make:model Photo -m`
4. Setup databases
	1. update db information in `.env`
	2. add table fields for model in `migrations/date_create_table.php` by using `$table->type('field_name')`
	3. Get around laravel migration bug by doing:

		```PHP
		// AppServiceProvider.php
		use Illuminate\Support\Facades\Schema;
		...
		// in boot function
		Schema::defaultStringLength(191);
		```
  4. `php artisan migrate` to create the tables

5. Create blade templates in resources folder, in controllers use `return view('albums.index');` to show the views.
  1. Use [LaravelCollective/HTML](https://laravelcollective.com/docs/master/html) to build custom component for code reuse
  2. `composer require "laravelcollective/html":"^5.7.0"`
		```PHP
		// add below to config/app.php => providers
		Collective\Html\HtmlServiceProvider::class,

		// add below to config/app.php => aliases
		'Form' => Collective\Html\FormFacade::class,
		'Html' => Collective\Html\HtmlFacade::class,
		```
  * `php artisan make:provider FormServiceProvider`
  * Add `App\Providers\FormServiceProvider::class` to `config/app.php providers` array

		```PHP
		// App/Providers/FormServiceProvider boot function
		Form::component('text', 'components.form.text', ['name', 'value' => null, 'attributes' => []]);
		Form::component('textarea', 'components.form.textarea', ['name', 'value' => null, 'attributes' => []]);
		Form::component('submit', 'components.form.submit', ['value' => 'Submit', 'attributes' => []]);
		Form::component('hidden', 'components.form.hidden', ['name', 'value' => null, 'attributes' => []]);
		Form::component('file', 'components.form.file', ['name',  'attributes' => []]);
		```

  * create `views/components/form/{text,textarea,submit,hidden,file}.blade.php`

		```PHP
		/*
		<label>
			{{Form::label($name)}}
			{{Form::text($name, $value, $attritutes)}}
		</label>
		*/
		```
  * Create the form

		```PHP
		@section('content')
		  <h3>Create Album</h3>
		  {!!Form::open(['action' => 'AlbumsController@store','method' => 'POST', 'enctype' => 'multipart/form-data'])!!}
		    {{Form::text('name','',['placeholder' => 'Album Name'])}}
		    {{Form::textarea('description','',['placeholder' => 'Album Description'])}}
		    {{Form::file('cover_image')}}
		    {{Form::submit('submit')}}
		  {!! Form::close() !!}
		@endsection
		```

6. Create Routes in routes/web.php `Route::get('/', 'AlbumsController@index')`

7. File upload handling with controller
	```PHP
	/** AlbumsController.php **/

	public function store(Request $request) {}
		$this->validate($request, [
			'name' => 'required',
			'cover_image' => 'image|max:1999'
		]);

		// Get filename with extension
		$filenameWithExt = $request->file('cover_image')->getClientOriginalName();

		// Get just the filename
		$filename = pathinfo($filenameWithExt, PATHINFO_FILENAME);

		// Get extension
		$extension = $request->file('cover_image')->getClientOriginalExtension();

		// Create new filename
		$filenameToStore = $filename.'_'.time().'.'.$extension;

		// Uplaod image
		$path= $request->file('cover_image')->storeAs('public/album_covers', $filenameToStore);

		// Create album
		$album = new Album;
		$album->name = $request->input('name');
		$album->description = $request->input('description');
		$album->cover_image = $filenameToStore;

		$album->save();

		return redirect('/albums')->with('success', 'Album Created');
	}
	```

8. `php artisan storage:link` to create link for file storage
9. Set relationship between `album` and `photo`
	```php
	/** App\Album.php **/
	  protected $fillable = array('name', 'description', 'cover_image');
	public function photos(){
		return $this->hasMany('App\Photo');
	}

	/** App\Photo **/
	protected $fillable = array('album_id', 'description', 'photo', 'title', 'size');

	public function album(){
		return $this->belongsTo('App\Album');
	}
	```
