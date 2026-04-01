# Rhymix Template Reference

Rhymix supports two template engines:
- `.html` files: v1 by default; opt into v2 with `@version(2)` or `<config version="2" />`
- `.blade.php` files: v2 automatically

## Template v1 (XE-compatible, .html files)

### Variable Output

```
{$variable}
{$object->property}
{$object->method()}
{$array[$key]}
{$var|filter}
{$var|date:'Y-m-d'}
{$content|noescape}
```

### PHP Code Execution

```
{@ $myvar = 'value' }
{@ $count = count($list) }
```

### Conditionals (HTML comment style)

```html
<!--@if($condition)-->
    content
<!--@elseif($other)-->
    other content
<!--@else-->
    fallback
<!--@end-->
```

### Loops

```html
<!--@foreach($list as $key => $item)-->
    {$item->title}
<!--@endforeach-->
```

### Attribute Conditionals

```html
<li class="active"|cond="$val['selected']">
<input checked="checked"|cond="$is_checked">
```

### Tag loop/cond Attributes

```html
<tr loop="$document_list=>$no,$document">
    <td>{$document->getTitle()}</td>
</tr>

<div cond="$show_content">content</div>
```

### Include

```html
<include target="header.html" />
<include target="^/common/tpl/popup_layout.html" />
<include target="sub_template" cond="$condition" />
```

### Resource Loading

```html
<load target="style.css" />
<load target="style.scss" vars="$scss_vars" />
<load target="script.js" type="body" index="10" />
<load target="./lang" />
<unload target="old_script.js" />
```

### Template Comments (stripped from output)

```html
<!--// This is a template comment -->
```

### Auto-escape

```html
<config autoescape="on" />
```

### Form Auto-processing

Forms automatically receive `mid`, `act`, `error_return_url` hidden fields and ruleset processing.
Disable with `rx-autoform="false"` attribute.

## Template v2 (Blade-style)

### Version Declaration (in .html files)

```
@version(2)
```

### Variable Output

```
{{ $var }}              // Auto-escaped
{!! $var !!}            // Unescaped (raw HTML)
{{ $var|lower }}        // With filter
{{ $date|date:'n/j' }} // Filter with option
{$var}                  // v1-compatible (restricted in script/style blocks)
```

### Filter List

| Filter | Description |
|--------|-------------|
| `autoescape` | Auto-escape |
| `autolang` | Auto-language |
| `escape` | HTML escape |
| `escapejs` | JS escape |
| `noescape` | Disable escaping |
| `json` | JSON encode |
| `strip` / `strip_tags` | Strip tags |
| `trim` | Trim whitespace |
| `urlencode` | URL encode |
| `lower` | Lowercase |
| `upper` | Uppercase |
| `nl2br` | Newline to `<br>` |
| `join` | Join array (separator, default `,`) |
| `date` | Date format (default `Y-m-d H:i:s`) |
| `format` / `number_format` | Number format (decimals, default `0`) |
| `shorten` / `number_shorten` | Number shorten (decimals, default `2`) |
| `link` | Generate link |

### Conditionals

```
@if ($condition)
    content
@elseif ($other)
    other
@else
    fallback
@endif
```

### Loops

```
@foreach ($items as $key => $item)
    {{ $item->name }}
@endforeach

@forelse ($items as $item)
    {{ $item->name }}
@empty
    No items found.
@endforelse

@for ($i = 0; $i < 10; $i++)
@endfor

@while ($condition)
@endwhile

@switch ($value)
    @case('a') ... @break
    @default ...
@endswitch
```

### $loop Variable (inside foreach/forelse)

| Property | Description |
|----------|-------------|
| `$loop->index` | 0-based index |
| `$loop->iteration` | 1-based index |
| `$loop->remaining` | Remaining count |
| `$loop->count` | Total count |
| `$loop->first` | Is first item |
| `$loop->last` | Is last item |
| `$loop->even` | Is even iteration |
| `$loop->odd` | Is odd iteration |
| `$loop->depth` | Nesting depth |
| `$loop->parent` | Parent loop variable |

### Auth/Permission Directives

```
@admin ... @endadmin
@auth ... @endauth
@auth('admin') ... @endauth
@auth('manager') ... @endauth
@guest ... @endguest
@can('permission') ... @endcan
@cannot('permission') ... @endcannot
@canany(['view', 'write_comment']) ... @endcanany
@desktop ... @enddesktop
@mobile ... @endmobile
```

### Convenience Conditionals

```
@isset ($foo) ... @endisset
@unset ($foo) ... @endunset
@empty ($foo) ... @endempty
@env ('foo') ... @endenv
```

### Include

```
@include ('dir/filename')
@include ('^/common/tpl/default_layout')
@includeIf ('path')
@includeWhen ($condition, 'path')
@includeUnless ($condition, 'path')
@include ('component', ['key' => $value])
```

### Resource Loading

```
@load ('style.css')
@load ('style.scss', $vars)
@load ('style.css', 'print', 20)
@load ('script.js', 'body', 10)
@load ('^/common/js/plugins/ckeditor/')
@load ('../lang/')
@unload ('foo/bar.js')
```

### HTML Attribute Helpers

```html
<div @class(['base', 'active' => $isActive])>
<div @style(['color: red', 'bold' => $isBold])>
<input @checked($val)>
<option @selected($val == 1)>
<input @disabled($val)>
<input @readonly($val)>
<input @required($val)>
```

### Utility Directives

```
@csrf                               // CSRF token
@json($data)                        // JSON encode
@lang('lang.key')                   // Translation
@url('key', 'value')                // URL generation
@url(['mid' => $mid, 'act' => 'dispBoardWrite'])
@widget('widget_name', $args)       // Widget insertion
@dump($var)                         // Debug dump
@dd($var)                           // Dump and die
@use('Namespace\Class', 'Alias')    // Class alias
@php ... @endphp                    // Raw PHP
@verbatim ... @endverbatim          // Skip parsing
@once ... @endonce                  // Execute once
@fragment('name') ... @endfragment  // Named fragment
@push('scripts') ... @endpush      // Push to stack
@pushOnce('scripts') ... @endPushOnce
@prepend('scripts') ... @endprepend
@stack('scripts')                   // Render stack
@error('path') ... @enderror        // Error handling
```

### Unsupported Blade Features

`@extends`, `@yield`, `@section`, `@show`, `@inject`, `@slot` are NOT supported — they conflict with Rhymix's modular architecture. Use variable-passing includes or widgets instead.
