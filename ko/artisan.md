# Artisan 콘솔

- [소개](#introduction)
- [명령 작성](#writing-commands)
    - [명령 생성](#generating-commands)
    - [명령 구조](#command-structure)
    - [폐쇄 명령](#closure-commands)
- [입력 기대치 정의](#defining-input-expectations)
    - [인수](#arguments)
    - [옵션](#options)
    - [입력 배열](#input-arrays)
    - [입력 설명](#input-descriptions)
- [명령 I / O](#command-io)
    - [입력 검색](#retrieving-input)
    - [입력 요청](#prompting-for-input)
    - [출력 쓰기](#writing-output)
- [명령 등록](#registering-commands)
- [프로그래밍 방식으로 명령 실행](#programmatically-executing-commands)
    - [다른 명령에서 명령 호출](#calling-commands-from-other-commands)

<a name="introduction"></a>

## 소개

Artisan은 Laravel에 포함 된 명령 줄 인터페이스입니다. 응용 프로그램을 빌드하는 동안 도움이 될 수있는 여러 가지 유용한 명령을 제공합니다. 사용 가능한 모든 Artisan 명령 목록을 보려면 `list` 명령을 사용할 수 있습니다.

```
php artisan list
```

모든 명령에는 명령의 사용 가능한 인수 및 옵션을 표시하고 설명하는 "도움말"화면도 포함되어 있습니다. 도움말 화면을 보려면 명령 이름 앞에 `help` .

```
php artisan help migrate
```

#### 라 라벨 REPL

모든 Laravel 애플리케이션에는 [PsySH](https://github.com/bobthecow/psysh) 패키지로 구동되는 REPL 인 Tinker가 포함됩니다. Tinker를 사용하면 Eloquent ORM, 작업, 이벤트 등을 포함하여 명령 줄에서 전체 Laravel 애플리케이션과 상호 작용할 수 있습니다. Tinker 환경에 들어가려면 `tinker` Artisan 명령을 실행하십시오.

```
php artisan tinker
```

<a name="writing-commands"></a>

## 명령 작성

Artisan과 함께 제공되는 명령 외에도 사용자 지정 명령을 만들 수도 있습니다. 명령은 일반적으로 `app/Console/Commands` 디렉토리에 저장됩니다. 그러나 Composer에서 명령을로드 할 수있는 한 자신의 저장 위치를 자유롭게 선택할 수 있습니다.

<a name="generating-commands"></a>

### 명령 생성

새 명령을 생성하려면 `make:command` Artisan 명령을 사용하십시오. 이 명령은 `app/Console/Commands` 디렉터리에 새 명령 클래스를 만듭니다. `make:command` Artisan 명령을 처음 실행할 때 생성되므로이 디렉토리가 애플리케이션에 존재하지 않아도 걱정하지 마십시오. 생성 된 명령에는 모든 명령에있는 기본 속성 및 메서드 집합이 포함됩니다.

```
php artisan make:command SendEmails
```

<a name="command-structure"></a>

### 명령 구조

명령을 생성 한 후 `list` 화면에 명령을 표시 할 때 사용할 클래스의 `signature` 및 `description` 속성을 입력해야합니다. `handle` 메서드는 명령이 실행될 때 호출됩니다. 이 메서드에 명령 논리를 배치 할 수 있습니다.

> {tip} 코드 재 사용률을 높이려면 콘솔 명령을 가볍게 유지하고 작업을 수행하기 위해 애플리케이션 서비스를 연기하도록하는 것이 좋습니다. 아래의 예에서 우리는 이메일을 보내는 "무거운 작업"을 수행하기 위해 서비스 클래스를 삽입합니다.

Let's take a look at an example command. Note that we are able to inject any dependencies we need into the command's constructor or `handle` method. The Laravel [service container](/docs/%7B%7Bversion%7D%7D/container) will automatically inject all dependencies type-hinted in the constructor or `handle` method:

```
<?php

namespace App\Console\Commands;

use App\User;
use App\DripEmailer;
use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send drip e-mails to a user';

    /**
     * The drip e-mail service.
     *
     * @var DripEmailer
     */
    protected $drip;

    /**
     * Create a new command instance.
     *
     * @param  DripEmailer  $drip
        * @return void
     */
    public function __construct(DripEmailer $drip)
    {
        parent::__construct();

        $this->drip = $drip;
    }

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->drip->send(User::find($this->argument('user')));
    }
}
```

<a name="closure-commands"></a>

### 폐쇄 명령

클로저 기반 명령은 콘솔 명령을 클래스로 정의하는 대안을 제공합니다. 라우트 클로저가 컨트롤러의 대안 인 것과 같은 방식으로 커맨드 클로저를 커맨드 클래스의 대안으로 생각하십시오. `app/Console/Kernel.php` 파일의 `commands` 메서드 내에서 Laravel은 `routes/console.php` 파일을로드합니다.

```
/**
 * Register the Closure based commands for the application.
 *
 * @return void
 */
protected function commands()
{
    require base_path('routes/console.php');
}
```

이 파일은 HTTP 경로를 정의하지 않지만 콘솔 기반 진입 점 (경로)을 애플리케이션에 정의합니다. 이 파일 내에서 `Artisan::command` 메소드를 사용하여 모든 Closure 기반 경로를 정의 할 수 있습니다. `command` 메소드는 두 개의 인수를받습니다 : [명령 서명](#defining-input-expectations) 과 명령 인수와 옵션을받는 Closure :

```
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
});
```

Closure는 기본 명령 인스턴스에 바인딩되어 있으므로 일반적으로 전체 명령 클래스에서 액세스 할 수있는 모든 도우미 메서드에 대한 전체 액세스 권한이 있습니다.

#### 유형 힌팅 종속성

명령의 인수와 옵션을받는 것 외에도 명령 클로저는 [서비스 컨테이너](/docs/%7B%7Bversion%7D%7D/container) 에서 해결하려는 추가 종속성을 입력 힌트 할 수도 있습니다.

```
use App\User;
use App\DripEmailer;

Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
    $drip->send(User::find($user));
});
```

#### 폐쇄 명령 설명

Closure 기반 명령을 정의 할 때 `describe` 메서드를 사용하여 명령에 설명을 추가 할 수 있습니다. 이 설명은 `php artisan list` 또는 `php artisan help` 명령을 실행할 때 표시됩니다.

```
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
})->describe('Build the project');
```

<a name="defining-input-expectations"></a>

## 입력 기대치 정의

콘솔 명령을 작성할 때 인수 또는 옵션을 통해 사용자로부터 입력을 수집하는 것이 일반적입니다. Laravel을 사용하면 명령의 `signature` 속성을 사용하여 사용자에게 기대하는 입력을 매우 편리하게 정의 할 수 있습니다. `signature` 속성을 사용하면 경로와 유사한 단일 표현 구문으로 명령의 이름, 인수 및 옵션을 정의 할 수 있습니다.

<a name="arguments"></a>

### 인수

모든 사용자 제공 인수와 옵션은 중괄호로 묶여 있습니다. 다음 예제에서 명령은 하나의 **필수** 인수 인 `user` 정의합니다.

```
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user}';
```

인수를 선택적으로 만들고 인수의 기본값을 정의 할 수도 있습니다.

```
// Optional argument...
email:send {user?}

// Optional argument with default value...
email:send {user=foo}
```

<a name="options"></a>

### 옵션

인수와 같은 옵션은 사용자 입력의 또 다른 형태입니다. 옵션이 명령 줄에 지정되면 두 개의 하이픈 ( `--` ) 접두사가 붙습니다. 두 가지 유형의 옵션이 있습니다. 값을받는 옵션과 그렇지 않은 옵션입니다. 값을받지 않는 옵션은 부울 "스위치"역할을합니다. 이 옵션 유형의 예를 살펴 보겠습니다.

```
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue}';
```

이 예에서 Artisan 명령을 호출 할 때 `--queue` 스위치를 지정할 수 있습니다. `--queue` 스위치가 전달되면 옵션 값은 `true` 됩니다. 그렇지 않으면 값은 `false` .

```
php artisan email:send 1 --queue
```

<a name="options-with-values"></a>

#### 값이있는 옵션

다음으로 값을 기대하는 옵션을 살펴 보겠습니다. 사용자가 옵션 값을 지정해야하는 경우 옵션 이름에 `=` 기호를 붙입니다.

```
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue=}';
```

이 예에서 사용자는 다음과 같이 옵션 값을 전달할 수 있습니다.

```
php artisan email:send 1 --queue=default
```

옵션 이름 뒤에 기본값을 지정하여 옵션에 기본값을 할당 할 수 있습니다. 사용자가 옵션 값을 전달하지 않으면 기본값이 사용됩니다.

```
email:send {user} {--queue=default}
```

<a name="option-shortcuts"></a>

#### 옵션 단축키

옵션을 정의 할 때 바로 가기를 지정하려면 옵션 이름 앞에 지정하고 | 전체 옵션 이름에서 바로 가기를 구분하는 구분 기호 :

```
email:send {user} {--Q|queue}
```

<a name="input-arrays"></a>

### 입력 배열

배열 입력을 기대하는 인수 또는 옵션을 정의하려면 `*` 문자를 사용할 수 있습니다. 먼저 배열 인수를 지정하는 예를 살펴 보겠습니다.

```
email:send {user*}
```

이 메서드를 호출 할 때 `user` 인수가 명령 줄에 순서대로 전달 될 수 있습니다. 예를 들어 다음 명령은 `user` 의 값을 `['foo', 'bar']` .

```
php artisan email:send foo bar
```

배열 입력을 예상하는 옵션을 정의 할 때 명령에 전달되는 각 옵션 값은 옵션 이름으로 시작해야합니다.

```
email:send {user} {--id=*}

php artisan email:send --id=1 --id=2
```

<a name="input-descriptions"></a>

### 입력 설명

콜론을 사용하여 설명에서 매개 변수를 분리하여 입력 인수 및 옵션에 설명을 지정할 수 있습니다. 명령을 정의하기 위해 약간의 추가 공간이 필요하면 정의를 여러 줄로 자유롭게 분산하십시오.

```
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'email:send
                        {user : The ID of the user}
                        {--queue= : Whether the job should be queued}';
```

<a name="command-io"></a>

## 명령 I / O

<a name="retrieving-input"></a>

### 입력 검색

명령이 실행되는 동안 명령에서 허용하는 인수 및 옵션의 값에 액세스해야합니다. 이를 위해 `argument` 및 `option` 메소드를 사용할 수 있습니다.

```
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $userId = $this->argument('user');

    //
}
```

모든 인수를 `array` 로 검색해야하는 경우 `arguments` 메서드를 호출합니다.

```
$arguments = $this->arguments();
```

옵션은 `option` 메소드를 사용하여 인수만큼 쉽게 검색 할 수 있습니다. 모든 옵션을 배열로 검색하려면 `options` 메소드를 호출하십시오.

```
// Retrieve a specific option...
$queueName = $this->option('queue');

// Retrieve all options...
$options = $this->options();
```

인수 또는 옵션이 없으면 `null` 이 반환됩니다.

<a name="prompting-for-input"></a>

### 입력 요청

출력을 표시하는 것 외에도 명령을 실행하는 동안 입력을 제공하도록 사용자에게 요청할 수도 있습니다. `ask` 메소드는 사용자에게 주어진 질문을 프롬프트하고 입력을 수락 한 다음 사용자의 입력을 명령으로 되돌립니다.

```
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $name = $this->ask('What is your name?');
}
```

`secret` 방법은 `ask` 와 유사하지만 사용자가 콘솔에 입력 할 때 사용자의 입력이 표시되지 않습니다. 이 방법은 비밀번호와 같은 민감한 정보를 요청할 때 유용합니다.

```
$password = $this->secret('What is the password?');
```

#### 확인 요청

만약 간단한 확인이 필요한 경우 `confirm` 메서드를 사용할 수 있다. 기본적으로 이 메서드는 `false`를 반환한다. 하지만 사용자가 프롬프트에서 `y` 또는 `yes` 를 입력하면 `true`가 반환된다.

```
if ($this->confirm('Do you wish to continue?')) {
    //
}
```

#### 자동 완성

`anticipate` 방법을 사용하여 가능한 선택에 대한 자동 완성을 제공 할 수 있습니다. 사용자는 자동 완성 힌트에 관계없이 모든 답변을 선택할 수 있습니다.

```
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```

#### 객관식 질문

사용자에게 미리 정의 된 선택 사항 집합을 제공해야하는 경우 `choice` 방법을 사용할 수 있습니다. 옵션을 선택하지 않은 경우 반환 될 기본값의 배열 인덱스를 설정할 수 있습니다.

```
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);
```

<a name="writing-output"></a>

### 출력 쓰기

콘솔에 출력을 보내려면 `line` , `info` , `comment` , `question` 및 `error` 메소드를 사용하십시오. 이러한 각 방법은 용도에 적합한 ANSI 색상을 사용합니다. 예를 들어 사용자에게 몇 가지 일반적인 정보를 표시해 보겠습니다. 일반적으로 `info` 메소드는 콘솔에 녹색 텍스트로 표시됩니다.

```
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $this->info('Display this on the screen');
}
```

오류 메시지를 표시하려면 `error` 방법을 사용하십시오. 오류 메시지 텍스트는 일반적으로 빨간색으로 표시됩니다.

```
$this->error('Something went wrong!');
```

색상이 지정되지 않은 일반 콘솔 출력을 표시하려면 `line` 메서드를 사용하십시오.

```
$this->line('Display this on the screen');
```

#### 테이블 레이아웃

`table` 방법을 사용하면 데이터의 여러 행 / 열을 올바르게 형식화 할 수 있습니다. 헤더와 행을 메소드에 전달하기 만하면됩니다. 너비와 높이는 주어진 데이터를 기반으로 동적으로 계산됩니다.

```
$headers = ['Name', 'Email'];

$users = App\User::all(['name', 'email'])->toArray();

$this->table($headers, $users);
```

#### 진행률 표시 줄

장기 실행 작업의 경우 진행률 표시기를 표시하는 것이 도움이 될 수 있습니다. 출력 개체를 사용하여 진행률 표시 줄을 시작, 진행 및 중지 할 수 있습니다. 먼저 프로세스가 반복 할 총 단계 수를 정의합니다. 그런 다음 각 항목을 처리 한 후 진행률 표시 줄을 진행합니다.

```
$users = App\User::all();

$bar = $this->output->createProgressBar(count($users));

foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```

고급 옵션에 대해서는 [Symfony Progress Bar 구성 요소 문서를](https://symfony.com/doc/current/components/console/helpers/progressbar.html) 확인하십시오.

<a name="registering-commands"></a>

## 명령 등록

콘솔 커널의 `commands` 메소드에서 `load` 메소드 호출로 인해 `app/Console/Commands` 디렉토리에있는 모든 명령이 자동으로 Artisan에 등록됩니다. 사실, 당신은 아티 즌 명령어를 위해 다른 디렉토리를 스캔하기 위해 `load` 메소드를 추가로 호출 할 수 있습니다 :

```
/**
 * Register the commands for the application.
 *
 * @return void
 */
protected function commands()
{
    $this->load(__DIR__.'/Commands');
    $this->load(__DIR__.'/MoreCommands');

    // ...
}
```

`app/Console/Kernel.php` 파일의 `$commands` 속성에 클래스 이름을 추가하여 수동으로 명령을 등록 할 수도 있습니다. Artisan이 부팅되면이 속성에 나열된 모든 명령이 [서비스 컨테이너에](/docs/%7B%7Bversion%7D%7D/container) 의해 해결되고 Artisan에 등록됩니다.

```
protected $commands = [
    Commands\SendEmails::class
];
```

<a name="programmatically-executing-commands"></a>

## 프로그래밍 방식으로 명령 실행

때로는 CLI 외부에서 Artisan 명령을 실행하고자 할 수 있습니다. 예를 들어, 경로 또는 컨트롤러에서 Artisan 명령을 실행하고자 할 수 있습니다. 이를 수행하기 위해 `Artisan` 파사드의 `call` 메소드를 사용할 수 있습니다. `call` 메소드는 명령의 이름 또는 클래스를 첫 번째 인수로, 명령 매개 변수 배열을 두 번째 인수로받습니다. 종료 코드가 반환됩니다.

```
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

`Artisan` 파사드의 `queue` 메서드를 사용하면 Artisan 명령을 [대기열 작업자](/docs/%7B%7Bversion%7D%7D/queues) 가 백그라운드에서 처리하도록 대기열에 넣을 수도 있습니다. 이 방법을 사용하기 전에 큐를 구성하고 큐 리스너를 실행 중인지 확인하십시오.

```
Route::get('/foo', function () {
    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

Artisan 명령이 전달되어야하는 연결 또는 대기열을 지정할 수도 있습니다.

```
Artisan::queue('email:send', [
    'user' => 1, '--queue' => 'default'
])->onConnection('redis')->onQueue('commands');
```

#### 배열 값 전달

명령이 배열을 허용하는 옵션을 정의하는 경우 해당 옵션에 값 배열을 전달할 수 있습니다.

```
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--id' => [5, 13]
    ]);
});
```

#### 부울 값 전달

`migrate:refresh` 명령의 `--force` 플래그와 같이 문자열 값을 허용하지 않는 옵션 값을 지정해야하는 경우 `true` 또는 `false` 전달해야합니다.

```
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

<a name="calling-commands-from-other-commands"></a>

### 다른 명령에서 명령 호출

때로는 기존 Artisan 명령에서 다른 명령을 호출하고 싶을 수 있습니다. `call` 메소드를 사용하면됩니다. 이 `call` 메소드는 명령 이름과 명령 매개 변수 배열을 허용합니다.

```
/**
 * Execute the console command.
 *
 * @return mixed
 */
public function handle()
{
    $this->call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
}
```

다른 콘솔 명령을 호출하고 모든 출력을 억제하려면 `callSilent` 메서드를 사용할 수 있습니다. `callSilent` 메소드는 `call` 메소드와 동일한 서명을 갖습니다.

```
$this->callSilent('email:send', [
    'user' => 1, '--queue' => 'default'
]);
```
