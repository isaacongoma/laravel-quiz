# LaravelQuiz
Pacote para adicionar questionários a um projeto Laravel.

## Requisitos Mínimos
- PHP 7.0
- Laravel 5.8
- Laravel Datatables 9.0

## Instalação
Para instalar, basta utilizar o comando abaixo:
```php
composer require brenofortunato/laravel-quiz
```

Em seguida, publique os assets:
```php
php artisan vendor:publish --provider="PandoApps\Quiz\QuizServiceProvider"
```

## Configuração
Certifique-se de que não existam tabelas com os nomes **questionnaires**, **question_types**, **questions**, **alternatives**, **executables** e **answers**. Caso existam, remova-as ou renomeie-as, não se esqueça dos models, views e tudo o que tiver relação com as tabelas citadas. Quando estiver pronto, execute a migration:
```php
php artisan migrate
```

Em seguida, execute o seeder **QuestionTypeSeeder**:
```php
php artisan db:seed --class=QuestionTypeSeeder
```

Abra o arquivo **config/quiz.php** e edite o array models para atender suas necessidades, conforme descrições abaixo:
```php
	'models' => [
		'executable'               => App\User::class,      // Model que responderá o questionário
		'executable_column_name'   => 'name',               // Nome da coluna que representa a descrição do model que executa o questionário
		'parent_type'              => App\Holding::class,   // Model que é dono do questionário
		'parent_id'                => 'holding_id',         // Nome da coluna que representa a FK para o model que é dono do questionário
		'parent_url_name'          => 'holdings',           // Nome da tabela do model que é dono do questionário
	]
```

Adicione o relacionamento abaixo ao model que responderá o questionário (no caso do exemplo acima, em **User**):
```php
	/**
	 * @return \Illuminate\Database\Eloquent\Relations\MorphToMany
	 **/
	public function answeredQuestionnaires()
	{
		return $this->morphToMany(\PandoApps\Quiz\Models\Questionnaire::class, 'executable')->withPivot('id', 'score', 'answered')->withTimestamps();
	}
```

E o relacionamento abaixo ao model que é dono do questionário (no caso do exemplo, em **Holding**):
```php
	/**
	* @return \Illuminate\Database\Eloquent\Relations\MorphMany
	**/
	public function questionnaires()
	{
		return $this->morphMany(\PandoApps\Quiz\Models\Questionnaire::class, 'parent');
	}
```

Adicione as rotas em **routes/web.php**:
```php
	Route::group(['prefix' => config('quiz.models.parent_url_name'). '/{' . config('quiz.models.parent_id'). '}'], function () {
		Route::group(['prefix' => 'questionnaires'], function () {
			Route::get('/',                                          ['as'=>'questionnaires.index',   'uses'=>'\PandoApps\Quiz\Controllers\QuestionnaireController@index']);
			Route::get('/create',                                    ['as'=>'questionnaires.create',  'uses'=>'\PandoApps\Quiz\Controllers\QuestionnaireController@create']);
			Route::post('/',                                         ['as'=>'questionnaires.store',   'uses'=>'\PandoApps\Quiz\Controllers\QuestionnaireController@store']);
			Route::get('/{questionnaire_id}',                        ['as'=>'questionnaires.show',    'uses'=>'\PandoApps\Quiz\Controllers\QuestionnaireController@show']);
			Route::match(['put', 'patch'], '/{questionnaire_id}',    ['as'=>'questionnaires.update',  'uses'=>'\PandoApps\Quiz\Controllers\QuestionnaireController@update']);
			Route::delete('/{questionnaire_id}',                     ['as'=>'questionnaires.destroy', 'uses'=>'\PandoApps\Quiz\Controllers\QuestionnaireController@destroy']);
			Route::get('/{questionnaire_id}/edit',                   ['as'=>'questionnaires.edit',    'uses'=>'\PandoApps\Quiz\Controllers\QuestionnaireController@edit']);
		});

		Route::group(['prefix' => 'questions'], function () {
			Route::get('/',                                         ['as'=>'questions.index',   'uses'=>'\PandoApps\Quiz\Controllers\QuestionController@index']);
			Route::get('/{question_id}',                            ['as'=>'questions.show',    'uses'=>'\PandoApps\Quiz\Controllers\QuestionController@show']);
			Route::match(['put', 'patch'], '/{question_id}',        ['as'=>'questions.update',  'uses'=>'\PandoApps\Quiz\Controllers\QuestionController@update']);
			Route::delete('/{question_id}',                         ['as'=>'questions.destroy', 'uses'=>'\PandoApps\Quiz\Controllers\QuestionController@destroy']);
			Route::get('/{question_id}/edit',                       ['as'=>'questions.edit',    'uses'=>'\PandoApps\Quiz\Controllers\QuestionController@edit']);
		});

		Route::group(['prefix' => 'alternatives'], function () {
			Route::get('/',                                        ['as'=>'alternatives.index',   'uses'=>'\PandoApps\Quiz\Controllers\AlternativeController@index']);
			Route::get('/{alternative_id}',                        ['as'=>'alternatives.show',    'uses'=>'\PandoApps\Quiz\Controllers\AlternativeController@show']);
			Route::match(['put', 'patch'], '/{alternative_id}',    ['as'=>'alternatives.update',  'uses'=>'\PandoApps\Quiz\Controllers\AlternativeController@update']);
			Route::delete('/{alternative_id}',                     ['as'=>'alternatives.destroy', 'uses'=>'\PandoApps\Quiz\Controllers\AlternativeController@destroy']);
			Route::get('/{alternative_id}/edit',                   ['as'=>'alternatives.edit',    'uses'=>'\PandoApps\Quiz\Controllers\AlternativeController@edit']);
		});

		Route::group(['prefix' => 'executables'], function () {
			Route::get('/',                                     ['as'=>'executables.index',         'uses'=>'\PandoApps\Quiz\Controllers\ExecutableController@index']);
			Route::get('/{questionnaire_id}/questionnaire',     ['as'=>'executables.statistics',    'uses'=>'\PandoApps\Quiz\Controllers\ExecutableController@statistics']);
			Route::get('{executable_id}/',                      ['as'=>'executables.show',          'uses'=>'\PandoApps\Quiz\Controllers\ExecutableController@show']);
			Route::get('{questionnaire_id}/create/{model_id}',  ['as'=>'executables.create',        'uses'=>'\PandoApps\Quiz\Controllers\ExecutableController@create']);
			Route::post('{questionnaire_id}/store',             ['as'=>'executables.store',         'uses'=>'\PandoApps\Quiz\Controllers\ExecutableController@store']);
			Route::post('start',                                ['as'=>'executables.start',         'uses'=>'\PandoApps\Quiz\Controllers\ExecutableController@start']);
		});

		Route::group(['prefix' => 'answers'], function () {
			Route::get('/',                                     ['as'=>'answers.index',   'uses'=>'\PandoApps\Quiz\Controllers\AnswerController@index']);
			Route::get('/{answer_id}',                          ['as'=>'answers.show',    'uses'=>'\PandoApps\Quiz\Controllers\AnswerController@show']);
		});
	});
```

Adicione o questionário ao menu em **resources/views/layouts/menu.blade.php**, substituindo **request()->PARENT_ID** pelo correspondente em seu caso (no exemplo, seria **request()->holding_id**):
```html
	<li class="{{ (Request::is('*questionnaires*') || Request::is('*questions*') || Request::is('*alternatives*')) ? 'active' : '' }}">
		<a href="{!! route('questionnaires.index', request()->PARENT_ID) !!}"><i class="far fa-list-alt sidebar-icons"></i><span>{!! \Lang::choice('tables.questionnaires','p') !!}</span></a>
		@if(Request::is('*questions*') && request()->questionnaire_id)
			<ul class="treeview-menu">
				<li class="{{ Request::is('*questions*') ? 'active text-bold' : '' }}">
					<a class="treeview-link" href="{!! route('questions.index', [request()->PARENT_ID, 'questionnaire_id' => request()->questionnaire_id]) !!}"><i class="fas fa-question sidebar-icons-treeview"></i><span>{!! \Lang::choice('tables.questions','p') !!}</span></a>
				</li>
			</ul>
		@endisset
		@if(Request::is('*alternatives*') && request()->question_id)
			<ul class="treeview-menu">
				<li class="{{ Request::is('*alternatives*') ? 'active text-bold' : '' }}">
					<a class="treeview-link" href="{!! route('alternatives.index', [request()->PARENT_ID, 'question_id' => request()->question_id]) !!}"><i class="fas fa-check-square sidebar-icons-treeview"></i><span>{!! \Lang::choice('tables.alternatives','p') !!}</span></a>
				</li>
			</ul>
		@endisset
	</li>
```

Adicione as traduções das tabelas em **resources/lang/pt_BR/tables.php**:
```php
	'questionnaires'        => '[s] Questionário         |[p] Questionários',
	'questions'             => '[s] Questão              |[p] Questões',
	'alternatives'          => '[s] Alternativa          |[p] Alternativas',
	'question_types'        => '[s] Tipo da Questão      |[p] Tipo das Questões',
	'answers'               => '[s] Resposta             |[p] Respostas',
```

## Personalização
As instruções abaixo não são necessárias, mas servem de orientação para uma maior personalização do pacote.

Caso queira modificar as traduções exibidas nas datatables, edite o arquivo **resources/lang/vandor/pandoapps/pt_BR/datatable.php**.

Caso queira modificar as views, edite os arquivos no diretório **resources/views/vendor/pandoapps**.

Para modificar as datatables, crie um cópia delas em **app/DataTables**. Utilize os arquivos abaixo como base (não se esqueça de mudar o namespace para **App\DataTables**):
- [QuestionnaireDataTable](https://github.com/BrenoFortunato/laravel-quiz/blob/master/src/DataTables/QuestionnaireDataTable.php)
- [QuestionDataTable](https://github.com/BrenoFortunato/laravel-quiz/blob/master/src/DataTables/QuestionDataTable.php)
- [AlternativeDataTable](https://github.com/BrenoFortunato/laravel-quiz/blob/master/src/DataTables/AlternativeDataTable.php)
- [ExecutableDataTable](https://github.com/BrenoFortunato/laravel-quiz/blob/master/src/DataTables/ExecutableDataTable.php)
- [AnswerDataTable](https://github.com/BrenoFortunato/laravel-quiz/blob/master/src/DataTables/AnswerDataTable.php)

Para modificar as controllers, crie um cópia delas em **app/Http/Controllers**. Utilize os arquivos abaixo como base (não se esqueça de mudar o namespace para **App\Http\Controllers**):
- [QuestionnaireController](https://github.com/BrenoFortunato/laravel-quiz/blob/master/src/Controllers/QuestionnaireController.php)
- [QuestionController](https://github.com/BrenoFortunato/laravel-quiz/blob/master/src/Controllers/QuestionController.php)
- [AlternativeController](https://github.com/BrenoFortunato/laravel-quiz/blob/master/src/Controllers/AlternativeController.php)
- [ExecutableController](https://github.com/BrenoFortunato/laravel-quiz/blob/master/src/Controllers/ExecutableController.php)
- [AnswerController](https://github.com/BrenoFortunato/laravel-quiz/blob/master/src/Controllers/AnswerController.php)

Ao modificar as controllers, não se esqueça de atualizar as rotas. Por exemplo, se **QuestionnaireController** for modificada, altere o bloco sob o prefixo **questionnaires** para:
```php
	Route::group(['prefix' => 'questionnaires'], function () {
		Route::get('/',                                          ['as'=>'questionnaires.index',   'uses'=>'QuestionnaireController@index']);
		Route::get('/create',                                    ['as'=>'questionnaires.create',  'uses'=>'QuestionnaireController@create']);
		Route::post('/',                                         ['as'=>'questionnaires.store',   'uses'=>'QuestionnaireController@store']);
		Route::get('/{questionnaire_id}',                        ['as'=>'questionnaires.show',    'uses'=>'QuestionnaireController@show']);
		Route::match(['put', 'patch'], '/{questionnaire_id}',    ['as'=>'questionnaires.update',  'uses'=>'QuestionnaireController@update']);
		Route::delete('/{questionnaire_id}',                     ['as'=>'questionnaires.destroy', 'uses'=>'QuestionnaireController@destroy']);
		Route::get('/{questionnaire_id}/edit',                   ['as'=>'questionnaires.edit',    'uses'=>'QuestionnaireController@edit']);
	});
```