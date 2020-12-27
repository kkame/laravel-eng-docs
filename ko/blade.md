# 블레이드 템플릿

- [소개](#introduction)
- [템플릿 상속](#template-inheritance)
    - [레이아웃 정의](#defining-a-layout)
    - [레이아웃 확장](#extending-a-layout)
- [부품 및 슬롯](#components-and-slots)
- [데이터 표시](#displaying-data)
    - [블레이드 및 JavaScript 프레임 워크](#blade-and-javascript-frameworks)
- [제어 구조](#control-structures)
    - [If 문](#if-statements)
    - [Switch 문](#switch-statements)
    - [루프](#loops)
    - [루프 변수](#the-loop-variable)
    - [코멘트](#comments)
    - [PHP](#php)
- [하위보기 포함](#including-sub-views)
    - [컬렉션에 대한 렌더링 뷰](#rendering-views-for-collections)
- [스택](#stacks)
- [서비스 주입](#service-injection)
- [확장 블레이드](#extending-blade)
    - [사용자 정의 If 문](#custom-if-statements)

<a name="introduction"></a>

## <a>소개</a>

Blade는 Laravel과 함께 제공되는 간단하면서도 강력한 템플릿 엔진입니다. 다른 인기있는 PHP 템플릿 엔진과 달리 Blade는 뷰에서 일반 PHP 코드를 사용하는 것을 제한하지 않습니다. 실제로 모든 블레이드 뷰는 일반 PHP 코드로 컴파일되고 수정 될 때까지 캐시됩니다. 즉, 블레이드는 기본적으로 애플리케이션에 오버 헤드를 전혀 추가하지 않습니다. 블레이드보기 파일은 `.blade.php` 파일 확장자를 사용하며 일반적으로 `resources/views` 디렉토리에 저장됩니다.

<a name="template-inheritance"></a>

## 템플릿 상속

<a name="defining-a-layout"></a>

### 레이아웃 정의

Blade 사용의 두 가지 주요 이점은 *템플릿 상속* 과 *섹션* 입니다. 시작하려면 간단한 예를 살펴 보겠습니다. 먼저 "마스터"페이지 레이아웃을 살펴 보겠습니다. 대부분의 웹 애플리케이션은 다양한 페이지에서 동일한 일반 레이아웃을 유지하므로이 레이아웃을 단일 블레이드보기로 정의하는 것이 편리합니다.

```
<!-- Stored in resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

보시다시피이 파일에는 일반적인 HTML 마크 업이 포함되어 있습니다. 그러나 `@section` 및 `@yield` 지시문에 유의하십시오. `@section` 이름이 의미하는 것처럼하면서 지시는, 내용의 절을 정의 `@yield` 지시가 주어진 섹션의 내용을 표시하는 데 사용됩니다.

이제 애플리케이션의 레이아웃을 정의 했으므로 레이아웃을 상속하는 자식 페이지를 정의하겠습니다.

<a name="extending-a-layout"></a>

### 레이아웃 확장

자식보기를 정의 할 때 Blade `@extends` 지시문을 사용하여 자식보기가 "상속"해야하는 레이아웃을 지정합니다. 블레이드 레이아웃을 확장하는 뷰는 `@section` 지시문을 사용하여 레이아웃의 섹션에 콘텐츠를 삽입 할 수 있습니다. 위의 예에서 볼 수 있듯이 이러한 섹션의 내용은 `@yield` 사용하여 레이아웃에 표시됩니다.

```
<!-- Stored in resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @@parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

이 예에서 `sidebar` 섹션은 `@@parent` 지시문을 사용하여 레이아웃의 사이드 바에 콘텐츠를 덮어 쓰지 않고 추가합니다. `@@parent` 지시문은 뷰가 렌더링 될 때 레이아웃의 내용으로 대체됩니다.

> 앞의 예는 달리 팁 {}이 `sidebar` 와 부 단부 `@endsection` 대신 `@show` . `@endsection` 지시문은 섹션 만 정의하는 반면 `@show` 는 섹션을 정의하고 **즉시 생성** 합니다.

전역 `view` 도우미를 사용하여 경로에서 블레이드보기를 반환 할 수 있습니다.

```
Route::get('blade', function () {
    return view('child');
});
```

<a name="components-and-slots"></a>

## 부품 및 슬롯

구성 요소 및 슬롯은 섹션 및 레이아웃에 유사한 이점을 제공합니다. 그러나 일부는 구성 요소 및 슬롯의 정신적 모델을 이해하기 더 쉽게 찾을 수 있습니다. 먼저 애플리케이션 전체에서 재사용하고 싶은 재사용 가능한 "경고"구성 요소를 상상해 보겠습니다.

```
<!-- /resources/views/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

`{{ $slot }}` 변수에는 컴포넌트에 삽입하려는 콘텐츠가 포함됩니다. 이제이 구성 요소를 구성하기 위해 `@component` Blade 지시문을 사용할 수 있습니다.

```
@component('alert')
    <strong>Whoops!</strong> Something went wrong!
@endcomponent
```

구성 요소에 대해 여러 슬롯을 정의하는 것이 도움이되는 경우가 있습니다. "제목"을 삽입 할 수 있도록 경고 구성 요소를 수정 해 보겠습니다. 이름이 지정된 슬롯은 이름과 일치하는 변수를 "반향"하여 표시 할 수 있습니다.

```
<!-- /resources/views/alert.blade.php -->

<div class="alert alert-danger">
    <div class="alert-title">{{ $title }}</div>

    {{ $slot }}
</div>
```

이제 `@slot` 지시문을 사용하여 명명 된 슬롯에 콘텐츠를 삽입 할 수 있습니다. `@slot` 지시문에 포함되지 않은 콘텐츠는 `$slot` 변수의 구성 요소로 전달됩니다.

```
@component('alert')
    @slot('title')
        Forbidden
    @endslot

    You are not allowed to access this resource!
@endcomponent
```

#### 추가 데이터를 구성 요소에 전달

때로는 추가 데이터를 구성 요소에 전달해야 할 수도 있습니다. 이러한 이유로 데이터 배열을 `@component` 지시문의 두 번째 인수로 전달할 수 있습니다. 모든 데이터는 구성 요소 템플릿에서 변수로 사용할 수 있습니다.

```
@component('alert', ['foo' => 'bar'])
    ...
@endcomponent
```

#### 별칭 구성 요소

블레이드 구성 요소가 하위 디렉토리에 저장되어있는 경우 쉽게 액세스 할 수 있도록 별칭을 지정할 수 있습니다. 예를 들어 `resources/views/components/alert.blade.php` 저장된 블레이드 구성 요소를 상상해보십시오. `component` 메서드를 사용하여 components.alert에서 `alert` 로 `components.alert` 별칭을 지정할 수 있습니다. 일반적으로이 작업은 `AppServiceProvider` 의 `boot` 메서드에서 수행해야합니다.

```
use Illuminate\Support\Facades\Blade;

Blade::component('components.alert', 'alert');
```

구성 요소에 별칭이 지정되면 지시문을 사용하여 렌더링 할 수 있습니다.

```
@alert(['type' => 'danger'])
    You are not allowed to access this resource!
@endalert
```

추가 슬롯이없는 경우 구성 요소 매개 변수를 생략 할 수 있습니다.

```
@alert
    You are not allowed to access this resource!
@endalert
```

<a name="displaying-data"></a>

## 데이터 표시

변수를 중괄호로 묶어 블레이드보기에 전달 된 데이터를 표시 할 수 있습니다. 예를 들어, 다음 경로가 주어진 경우 :

```
Route::get('greeting', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

다음과 같이 `name` 변수의 내용을 표시 할 수 있습니다.

```
Hello, {{ $name }}.
```

물론 뷰에 전달 된 변수의 내용을 표시하는 데 국한되지 않습니다. PHP 함수의 결과를 에코 할 수도 있습니다. 실제로 Blade echo 문 안에 원하는 PHP 코드를 넣을 수 있습니다.

```
The current UNIX timestamp is {{ time() }}.
```

> {tip} Blade `{{ }}` 문은 XSS 공격을 방지하기 위해 PHP의 `htmlspecialchars` 함수를 통해 자동으로 전송됩니다.

#### 이스케이프되지 않은 데이터 표시

기본적으로 Blade `{{ }}` 문은 XSS 공격을 방지하기 위해 PHP의 `htmlspecialchars` 함수를 통해 자동으로 전송됩니다. 데이터를 이스케이프하지 않으려면 다음 구문을 사용할 수 있습니다.

```
Hello, {!! $name !!}.
```

> {note} 애플리케이션 사용자가 제공하는 콘텐츠를 에코 할 때 매우주의하세요. 사용자 제공 데이터를 표시 할 때 XSS 공격을 방지하려면 항상 이스케이프 처리 된 이중 중괄호 구문을 사용하십시오.

#### JSON 렌더링

때때로 자바 스크립트 변수를 초기화하기 위해 배열을 JSON으로 렌더링 할 의도로 뷰에 전달할 수 있습니다. 예를 들면 :

```
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

그러나 수동으로 `json_encode` 호출하는 대신 `@json` Blade 지시문을 사용할 수 있습니다.

```
<script>
    var app = @json($array);
</script>
```

#### HTML 엔티티 인코딩

기본적으로 Blade (및 Laravel `e` 도우미)는 HTML 엔티티를 이중 인코딩합니다. 이중 인코딩을 비활성화하려면 `AppServiceProvider` 의 `boot` 메서드에서 `Blade::withoutDoubleEncoding` 메서드를 호출합니다.

```
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::withoutDoubleEncoding();
    }
}
```

<a name="blade-and-javascript-frameworks"></a>

### 블레이드 및 JavaScript 프레임 워크

많은 JavaScript 프레임 워크도 "중괄호"를 사용하여 주어진 표현식이 브라우저에 표시되어야 함을 나타 내기 때문에 `@` 기호를 사용하여 Blade 렌더링 엔진에 표현식이 그대로 유지되어야 함을 알릴 수 있습니다. 예를 들면 :

```
<h1>Laravel</h1>

Hello, @{{ name }}.
```

이 예에서 `@` 기호는 Blade에 의해 제거됩니다. 그러나 `{{ name }}` 표현식은 Blade 엔진에 의해 변경되지 않고 대신 JavaScript 프레임 워크에 의해 렌더링 될 수 있습니다.

#### `@verbatim` 지시어

템플릿의 많은 부분에 자바 스크립트 변수를 표시하는 경우 ` @verbatim ` 지시문에서 HTML을 래핑하여 각 Blade echo 문 앞에  기호 `@` 을 붙일 필요가 없습니다.

```
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

<a name="control-structures"></a>

## 제어 구조

템플릿 상속 및 데이터 표시 외에도 Blade는 조건문 및 루프와 같은 일반적인 PHP 제어 구조에 대한 편리한 단축키를 제공합니다. 이 단축키는 PHP 제어 구조를 사용하는 매우 깔끔하고 간결한 방법을 제공하는 동시에 PHP 대응에 익숙합니다.

<a name="if-statements"></a>

### If 문

<code> @if </code>, <code> @elseif </code>, <code> @else </code> 및 <code> @endif</code> 지시문을 사용하여 <code> if </code> 문을 생성 할 수 있습니다. 이 지시문은 PHP 대응 지시문과 동일하게 작동합니다.

```
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

편의를 위해 Blade는 `@unless` 지시문도 제공합니다.

```
@unless (Auth::check())
    You are not signed in.
@endunless
```

이미 논의 된 조건부 지시문 외에도 `@isset` 및 `@empty` 지시문은 각각의 PHP 함수에 대한 편리한 바로 가기로 사용할 수 있습니다.

```
@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

#### 인증 지시문

`@auth` 및 `@guest` 지시문을 사용하여 현재 사용자가 인증되었는지 또는 게스트인지 빠르게 확인할 수 있습니다.

```
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

필요한 경우 `@auth` 및 `@guest` 지시문을 사용할 때 확인해야하는 [인증 가드](/docs/%7B%7Bversion%7D%7D/authentication) 를 지정할 수 있습니다.

```
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

#### 섹션 지시문

`@hasSection` 지시문을 사용하여 섹션에 콘텐츠가 있는지 확인할 수 있습니다.

```
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

<a name="switch-statements"></a>

### Switch 문

Switch 문은 `@switch` , `@case` , `@break` , `@default` 및 `@endswitch` 지시문을 사용하여 구성 할 수 있습니다.

```
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

<a name="loops"></a>

### 루프

조건문 외에도 Blade는 PHP의 루프 구조 작업을위한 간단한 지시문을 제공합니다. 다시 말하지만, 이러한 각 지시문은 PHP 대응 지시문과 동일하게 작동합니다.

```
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

> {tip} 루핑 할 때 [ 루프 변수 ](#the-loop-variable)를 사용하여 루프의 첫 번째 또는 마지막 반복에 있는지 여부와 같은 루프에 대한 중요한 정보를 얻을 수 있습니다.

루프를 사용할 때 루프를 종료하거나 현재 반복을 건너 뛸 수도 있습니다.

```
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

지시문 선언과 함께 조건을 한 줄에 포함 할 수도 있습니다.

```
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

<a name="the-loop-variable"></a>

### 루프 변수

루핑 할 때 루프 내에서 `$loop` 변수를 사용할 수 있습니다. 이 변수는 현재 루프 인덱스 및 이것이 루프를 통한 첫 번째 또는 마지막 반복인지 여부와 같은 유용한 정보 비트에 대한 액세스를 제공합니다.

```
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

중첩 루프에있는 경우 `parent` 속성을 통해 부모 루프의 `$loop` 변수에 액세스 할 수 있습니다.

```
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

`$loop` 변수는 또한 다양한 기타 유용한 속성을 포함합니다.

특성 | 기술
--- | ---
`$loop->index` | 현재 루프 반복의 인덱스입니다 (0에서 시작).
`$loop->iteration` | 현재 루프 반복 (1에서 시작).
`$loop->remaining` | 루프에 남아있는 반복입니다.
`$loop->count` | 반복되는 배열의 총 항목 수입니다.
`$loop->first` | 이것이 루프를 통한 첫 번째 반복인지 여부.
`$loop->last` | 이것이 루프를 통한 마지막 반복인지 여부.
`$loop->depth` | 현재 루프의 중첩 수준입니다.
`$loop->parent` | 중첩 루프에있을 때 부모의 루프 변수입니다.

<a name="comments"></a>

### 코멘트

Blade를 사용하면 뷰에서 주석을 정의 할 수도 있습니다. 그러나 HTML 주석과 달리 Blade 주석은 애플리케이션에서 반환하는 HTML에 포함되지 않습니다.

```
{{-- This comment will not be present in the rendered HTML --}}
```

<a name="php"></a>

### PHP

어떤 상황에서는 뷰에 PHP 코드를 포함하는 것이 유용합니다. Blade `@php` 지시문을 사용하여 템플릿 내에서 일반 PHP 블록을 실행할 수 있습니다.

```
@php
    //
@endphp
```

> {tip} Blade는이 기능을 제공하지만 자주 사용하면 템플릿 내에 너무 많은 로직이 포함되어 있다는 신호일 수 있습니다.

<a name="including-sub-views"></a>

## 하위보기 포함

Blade의 `@include` 지시문을 사용하면 다른보기 내에서 Blade보기를 포함 할 수 있습니다. 상위 뷰에서 사용할 수있는 모든 변수는 포함 된 뷰에서 사용할 수 있습니다.

```
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

포함 된보기가 상위보기에서 사용 가능한 모든 데이터를 상속하더라도 포함 된보기에 추가 데이터 배열을 전달할 수도 있습니다.

```
@include('view.name', ['some' => 'data'])
```

물론 존재하지 않는 뷰를 `@include` 하려고하면 Laravel은 오류를 발생시킵니다. 존재하거나 존재하지 않는 뷰를 포함하려면 `@includeIf` 지시문을 사용해야합니다.

```
@includeIf('view.name', ['some' => 'data'])
```

주어진 부울 조건에 따라 뷰를 `@include` 하려면 `@includeWhen` 지시문을 사용할 수 있습니다.

```
@includeWhen($boolean, 'view.name', ['some' => 'data'])
```

주어진 뷰 배열에서 존재하는 첫 번째 뷰를 포함하려면 `includeFirst` 지시문을 사용할 수 있습니다.

```
@includeFirst(['custom.admin', 'admin'], ['some' => 'data'])
```

> {note} 블레이드 뷰에서 `__DIR__` 및 `__FILE__` 상수는 캐시되고 컴파일 된 뷰의 위치를 참조하므로 사용하지 않아야합니다.

<a name="rendering-views-for-collections"></a>

### 컬렉션에 대한 렌더링 뷰

Blade의 `@each` 지시문을 사용하여 루프와 포함을 한 줄로 결합 할 수 있습니다.

```
@each('view.name', $jobs, 'job')
```

첫 번째 인수는 배열 또는 컬렉션의 각 요소에 대해 렌더링 할 뷰 부분입니다. 두 번째 인수는 반복하려는 배열 또는 컬렉션이고 세 번째 인수는 뷰 내에서 현재 반복에 할당 될 변수 이름입니다. 예를 들어 `jobs` 배열을 반복하는 경우 일반적으로 각 작업을 부분 뷰 내에서 `job` 변수로 액세스하려고합니다. 현재 반복의 키는 뷰 부분 내에서 `key` 변수로 사용할 수 있습니다.

`@each` 지시문에 네 번째 인수를 전달할 수도 있습니다. 이 인수는 주어진 배열이 비어있는 경우 렌더링 될 뷰를 결정합니다.

```
@each('view.name', $jobs, 'job', 'view.empty')
```

> {note} `@each` 를 통해 렌더링 된 뷰는 상위 뷰의 변수를 상속하지 않습니다. 자식보기에 이러한 변수가 필요한 경우 `@foreach` 및 `@include` 대신 사용해야합니다.

<a name="stacks"></a>

## 스택

Blade를 사용하면 다른 뷰 또는 레이아웃의 다른 곳에서 렌더링 할 수있는 명명 된 스택으로 푸시 할 수 있습니다. 이는 하위 뷰에 필요한 JavaScript 라이브러리를 지정하는 데 특히 유용 할 수 있습니다.

```
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

필요한만큼 여러 번 스택으로 푸시 할 수 있습니다. 전체 스택 콘텐츠를 렌더링하려면 스택 이름을 ` @stack ` 지시문에 전달합니다.

```
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

스택 시작 부분에 콘텐츠를 추가하려면 `@prepend` 지시문을 사용해야합니다.

```
@push('scripts')
    This will be second...
@endpush

// Later...

@prepend('scripts')
    This will be first...
@endprepend
```

<a name="service-injection"></a>

## 서비스 주입

`@inject` 지시문은 Laravel [서비스 컨테이너](/docs/%7B%7Bversion%7D%7D/container) 에서 서비스를 검색하는 데 사용할 수 있습니다. `@inject` 전달 된 첫 번째 인수는 서비스가 배치 될 변수의 이름이고 두 번째 인수는 해결하려는 서비스의 클래스 또는 인터페이스 이름입니다.

```
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

<a name="extending-blade"></a>

## 확장 블레이드

Blade를 사용하면 `directive` 방법을 사용하여 사용자 정의 지시문을 정의 할 수 있습니다. 블레이드 컴파일러가 사용자 지정 지시문을 발견하면 지시문에 포함 된 표현식과 함께 제공된 콜백을 호출합니다.

다음 예제는 `DateTime` 의 인스턴스 여야하는 주어진 `$var` 형식을 지정하는 `@datetime($var)` 지시문을 만듭니다.

```
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Perform post-registration booting of services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::directive('datetime', function ($expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }

    /**
     * Register bindings in the container.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

보시다시피, 우리는 지시문에 전달되는 표현식에 `format` 메서드를 연결합니다. 따라서이 예에서이 지시문에 의해 생성 된 최종 PHP는 다음과 같습니다.

```
<?php echo ($var)->format('m/d/Y H:i'); ?>
```

> {note} Blade 지시문의 로직을 업데이트 한 후에는 캐시 된 모든 Blade 뷰를 삭제해야합니다. 캐시 된 블레이드 뷰는 `view:clear` Artisan 명령을 사용하여 제거 할 수 있습니다.

<a name="custom-if-statements"></a>

### 사용자 정의 If 문

사용자 지정 지시문을 프로그래밍하는 것은 간단한 사용자 지정 조건문을 정의 할 때 필요 이상으로 복잡한 경우가 있습니다. 이러한 이유로 Blade는 Closure를 사용하여 사용자 지정 조건 지시문을 빠르게 정의 할 수있는 `Blade::if` 메서드를 제공합니다. 예를 들어 현재 애플리케이션 환경을 확인하는 사용자 지정 조건을 정의 해 보겠습니다. `AppServiceProvider` 의 `boot` 메서드에서이 작업을 수행 할 수 있습니다.

```
use Illuminate\Support\Facades\Blade;

/**
 * Perform post-registration booting of services.
 *
 * @return void
 */
public function boot()
{
    Blade::if('env', function ($environment) {
        return app()->environment($environment);
    });
}
```

사용자 지정 조건이 정의되면 템플릿에서 쉽게 사용할 수 있습니다.

```
@env('local')
    // The application is in the local environment...
@elseenv('testing')
    // The application is in the testing environment...
@else
    // The application is not in the local or testing environment...
@endenv
```
