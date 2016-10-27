# Laravel 5.3

## Установка 

```
composer create-progect laravel/laravel имя_папки 5.3.*
```

###Настройка роутов:

Файл /Routes/web.php

Простейший роут:

```php
Route::get('/', 'HelloController@index') // Первый аргумент - реквест. / значит корень сайта. Если реквест другой, то слэш писать не нужно.
										 // Второй аргумент - имя контроллера в папке контроллера @ метод контроллера
```
Передача изменяемого значения через реквест
```php
Route::get('articles/{id}', 'ArticlesController@index'); // id из фигурных скоюок будет передано аргуметом в метод контроллера
Route::get('articles/{slug}', 'PagesController@page'); // то же самое
```
###Контроллеры

Создание контроллера:

```
php artisan make:controller PageController
```

Метод контроллера обязательно что-то возвращает:

```php
public function index()
{
	return 'Hello';
}
```
Обычно возвращает имя шаблона, который требуется подключить к данному роуту (методу конроллера).

```php
public function index()
{
	return view('hello');
}
```

Причём если вьюха вложена в какую-то папку, то адрес шаблона указывается следующим образом view('folder.hello')

Для передачи чего либо из контроллера в шаблон в контроллере(методе) создаём какую либо переменную и регистрируем её имя для view в специальном
методе with, например во так:
```php
public function index()
{
	$hello = 'Hallo World';
	return view('hello')->with('hеllo', $hello);
}
```
В шаблон передастся переменная с именем, указаннов в качестве первого аргумента, со значением во втором аргументе.
Но чаще применяется вот такой синтаксис:
```php
public function index()
{
	$first = 'John';
	$last = 'Smith';
	return view('hello', compact('first', 'last'));
}
Тогда в шаблон передадуться переменные $first и $last
```

####

###Шаблоны (Вьюхи)

В laravel используется шаблонизатор blade. Все шаблоны обязательно именуются следующим образом:

имя_шаблона.blade.php, например: index.blade.php.

####Важная особенность - наследование шаблонов

Создадим базовый шаблон layout.blade.php. Поместим в него базовую html структуру.
В местах, в которых планируются какие либо изменения в шаблонах-наследниках необходимо проставить метки
@yield('имя_секции'), наприер @yield('content').

В шаблоне наследнике в первой строке необходимо указать наследуемый шалон extends('имя_шаблона')
Для вставки изменяемого контента пишем @section('content'), а в конце вставки пишем @stop.

Также в шаблонизаторе доступны такие конструкции как @if @else @endif. @unless (если), например @unless if ! - если не.
@foreach @endforeach и $foreelse - на случай если массив пустой, например можно поместить ответ - "Статей нет!".

Итнетерсный подход при формировании ссылок в шаблоне:

```php
<a href="{{action('ArticlesController@single', [$article->id])}}">{{$article->title}}</a> // Вставляется внутрь цикла @foreach

//Или так
<a href="{{url('/articles', $article->id)}}">{{$article->title}}</a>
```

###Работа с базой данных:

Нужные параметры задаются в файле .env в корне проекта. Дополнительные настройки в файле /config/database.php. По умолчанию константы файла .env 
имеют приоритет перед тем, что записано в /config/database.php, однако в том же /config/database.php этот приоритет можно легко поменять.

####Миграции:
Для корректной работы миграций желательно доустановить пакет Doctrine/DBAL:

```
composer require doctrine/dbal
```

Все файлы миграций хранятся в одном месте, путь: /database/migrations/

Создаётся миграция командой:

```
php artisan make:migration имя_миграции
```
или с автозаполнением
```
php artisan make:migration имя_миграции --create="articles"
```
В папке с миграциями появится соответствующий файл с заданным нами именем, в котором будет лежать класс под этим же именем. В классе будет 2 метода
up() и down().
В методе up как правило что то создаётся в БД, в методе же down наоборот что-то грохается. Может быть и обратная ситуация, когда миграция должна грохнуть
какую-то таблицу или колонку в таблице.
   
Итак в методе up() создаём таблицу 
```php
Schema::create('articles', function (Blueprint $table)
{
// Здесь создаём сущности (колонки таблицы)
}
```
Что прописываем в тело функции:
```php
$table->increments('id'); // айдиншик автоинкремент
$table->string('name');   // Строковый тип varchar(255)
$table->timestamps();	 // Временной тип	
```
Также можем указать доп параметры, значения по умолчанию, внешние ключи и др.

Например:
```php
$table->string('email')->unique()
```

Для применения миграции команда:
```
php artisan migrate
```
для отката:
```
php artisan migrate:rollback
```

Для добавления колонки:
```php
Schema::table('articles', function (Blueprint $table) {
            $table->text('excerpt');
        });
```
Для удаления:
```php
Schema::table('articles', function (Blueprint $table)
        {
            $table->dropColumn('excerpt');
        });
```

###Модели
####Eloquent(Active record)

Создание модели:
```
php artisan make:model Article
```
Или указать вложенность 
```
php artisan make:model Models\Article
```

Для отладки целесообразно использовать консоль PHP, запускается командой:
```
php artisan tinker
```
Добавление записи в таблицу БД:
```
$article = new App\Models\Article; // создаём объект модели, его сущность - строка в таблице БД
$article->title = 'My First Article'; // Присваивам значения для поля таблицы title
$article->body = 'Lorem ipsum';
$article->published_at = Carbon\Carbon::now(); // Время публикации
$article->save(); // Сохраняем, и в таблице должна появиться запись
$article->title = 'My Updated First Article'; // Тут же можем обновить отдельные поля
$article->save(); // Та же самая команда
```
Коллекции:
```php
$article = App\Models\Article::where('body', 'Lorem ipsum')->get(); // Достаём все записи, удовлетворяющие условиям
$article = App\Models\Article::where('body', 'Lorem ipsum')->first(); // Достаём одну запись из таблицы
```
Создаём новую запись
```php
$article = App\Models\Article::create(['title' => 'New Article', 'body' => 'New Body', 'published_at' => Carbon\Carbon::now()]);
```
И получаем ошибку:
```
Illuminate\Database\Eloquent\MassAssignmentException with message 'title'
```
Сработала защита от дурака. В модель необходимо добавить переменную - массив $fillable.

```php
	protected $fillable = [
			'title',
			'body',
			'published_at'
		]; // Перечисляем все поля, которые могут заполняться методом массового заполнения
```
Теперь можно повторить создание записи и всё сработает.
Можно обновлять ещё вот так:
```php
$article->update(['body' => 'Update AGAIN']);
```

```
App\Models\Article::all()->toArray(); // Выборка всех записей в формате массива
App\Models\Article::find(2)->toArray(); // Поиск по первичному ключу
```

###Формы

Для работы форм потребуется установка доппакетов

```
composer require laravelcollective/html
```
После этого в конфиг в файл app.php в секцию (массив) providers нужно добавить элемент
```php
Collective\Html\HtmlServiceProvider::class
```
В секцию aliases:

```php
		'Form' => Collective\Html\FormFacade::class,
        'Hlml' => Collective\Html\HtmlFacade::class,
```

Далее для добавления формы в шаблон нужно добавить следующую конструкцию:
```php
	{!! Form::open() !!}
		
    {!! Form::close() !!}
```
Наполним таблицу полями:	
```php
	{!! Form::open() !!}
        <div class="form-group">
            {!! Form::label('title', 'Заголовок:') !!}
            {!! Form::text('ешеду', null, ['class' => 'form-control']) !!}
        </div>

        <div class="form-group">
            {!! Form::label('body', 'Сожержимое:') !!}
            {!! Form::textarea('body', null, ['class' => 'form-control']) !!}
        </div>
        <div class="form-group">
            {!! Form::submit('Добавить статью', ['class' => 'btn btn-primary form-control']) !!}
        </div>

    {!! Form::close() !!}
```

Далее для того, чтобы ловить отправленное формой необходимо отправить ещё один маршрут:
```php
Route::post('articles/', 'ArticlesController@store');
```	
И создать для него метод в контроллере:
```php
	public function store()
    {
		$input = Request::all(); // Изымаем значение из поста
    }
```
Кроме того, в котроллере поменяем use Illuminate\Http\Request; на use Request;
Ну и поправляем action формы: В скобки {!! Form::open() кладем значение ['url' => 'articles'], 
получаем {!! Form::open(['url' => 'articles']).


Далее в методе store() добавляем запись из формы в таблицу и делаем редирект

```php
	public function store()
    {
		$input = Request::all();
		$input['published_at'] = Carbon::now(); // Дата публикации формируется автоматически - текущее время. Можно добавить форму в поле
		Article::create($input);
		return redirect('articles');
    }
```



Образец кода для файла helpers

```php

    <?php
    
    function pr($p)
    {
        echo "<pre>";
        if (is_array($p) || is_object($p)) {
            print_r($p);
        } else if ( is_bool($p) || empty($p) || (is_string($p) && trim($p) == '') ) {
            var_export($p);
        } else {
            print_r($p);
        }
        echo "</pre>";
    }
    
    function uncache($url)
    {
        $url =  '/' . $url;
        $url = str_replace('//', '/', $url);
        $full_path = public_path() . $url;
        
        $pref = 'not_found';
        if(file_exists($full_path)){
            $pref = filemtime($full_path);
        }
        
        return $url . '?' . $pref;
    }
    
    /**
     * добавляет к текущему запросу дополнительные параметры
     * @param $arr
     * @return string
     */
    function add_to_query($arr) {
        return http_build_query(array_merge(Input::query(), $arr));
    }
    
    /**
     * удаляет параметры у текущего запроса
     * @param $arr
     * @return string
     */
    function del_from_query($arr) {
        $tmp = Input::query();
        foreach ($arr as $val) {
            if (isset($tmp[$val])) {
                unset($tmp[$val]);
            }
        }
        return http_build_query($tmp);
    }
    
    /**
     * Вернет слово в правильном склонении в зависимости от кол-ва
     * @param $number
     * @param $one
     * @param $two
     * @param $five
     * @return mixed
     */
    function plurar($number, $one, $two , $five) {
        if (($number - $number % 10 ) % 100 != 10) {
            if ($number % 10 == 1 ) {
                $result = $one;
            } elseif ( ($number % 10 >= 2 ) && ($number % 10 <= 4) ) {
                $result = $two;
            } else {
                $result = $five;
            }
        } else {
            $result = $five;
        }
        return $result ;
    }
    
    /**
     * Добавит к названиям таблиц префиксы
     * @param $str
     * @return mixed
     */
    function table_prefix($str) {
        
        preg_match_all('/([A-z0-9]+\.[A-z0-9]+)/ui', $str, $arr);
        $prefix = DB::getTablePrefix();
        
        if (isset($arr[0])) {
            foreach ($arr[0] as $item) {
                if (!is_numeric($item)) {
                    $tmp = explode('.', trim($item));
                    if (count($tmp) == 2) {
                        $new = '`'.$prefix.$tmp[0].'`.`'.$tmp[1].'`';
                        $str = str_replace($item, $new, $str);
                    }
                }
            }
        }
        
        return $str;
    }
    
    /**
     * Возвращает SQL с подставленными значениями
     * @param $builder Illuminate\Database\Query\Builder
     * @return str
     */
    function toSql($builder) {
        $sql = $builder->toSql();
        foreach($builder->getBindings() as $binding)
        {
            if ($binding instanceof \DateTime) {
                $binding = $binding->format('Y-m-d H:i:s');
            }
            
            $value = is_numeric($binding) ? $binding : "'".$binding."'";
            $sql = preg_replace('/\?/', $value, $sql, 1);
        }
        return $sql;
    }
    
    /**
     * Выведет информацию о времени существования сайта
     * @param $start_year
     * @return string
     */
    function siteExist($start_year) {
        return (date('Y') <= $start_year) ? $start_year : $start_year.' — '.date('Y');
    }
    
    /**
     * Выведт значени поля у пользователя или пустоту
     * @param $fname
     * @return string
     */
    function user_field($fname) {
        return Auth::user() ? Auth::user()->{$fname} : NULL;
    }
    
    /**
     * @return string
     * @param string $str - Исходная строка
     * @param int $length - Размер результирующей строки
     * @param string $char - Символ которым дополнять строку
     * @param string $pos - Позиция добавления r/l
     * @desc Дополняем строку до нужного размера
     */
    function fit_line($str, $length, $char = ' ', $pos = 'r') {
        $str_length = mb_strlen($str, 'UTF-8');
        if ($str_length < $length) {
            $count = $length - $str_length;
            while ($count > 0) {
                $str = ($pos == 'r') ? $str.$char : $char.$str;
                $count--;
            }
        }
        return $str;
    }
    
    /**
     * Проверяет включены ли Кукисы у пользователя
     * @return bool
     */
    function isCookieEnable() {
        if (isset($_SERVER['HTTP_COOKIE'])) {
            return true;
        }
        return false;
    }
    
    /**
     * Вспомогательная функция для русской раскладки
     * @param $str
     * @param null $encoding
     * @return string
     */
    if(!function_exists('mb_ucfirst')) {
        function mb_ucfirst($str, $encoding = NULL) {
            if($encoding === NULL)  {
                $encoding    = mb_internal_encoding();
            }
            
            return mb_substr(mb_strtoupper($str, $encoding), 0, 1, $encoding) . mb_substr($str, 1, mb_strlen($str)-1, $encoding);
        }
    }
    
    function bytesToHuman($bytes)
    {
        $units = ['B', 'Kb', 'Mb', 'Gb', 'Tb', 'Pb'];
        
        for ($i = 0; $bytes > 1024; $i++)
        {
            $bytes /= 1024;
        }
        
        return round($bytes) . '&nbsp;' . $units[ $i ];
    }

```
