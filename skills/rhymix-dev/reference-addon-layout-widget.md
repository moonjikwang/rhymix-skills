# Rhymix Addon, Layout, Widget, Widgetstyle, and Skin Reference

## Addon

### Directory Structure

```
addons/{addon_name}/
  {addon_name}.addon.php    # Main script - REQUIRED
  conf/
    info.xml                # Metadata - REQUIRED
  {addon_name}.js           # JavaScript (optional)
  {addon_name}.lib.php      # Library (optional)
```

### conf/info.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<addon version="0.2">
    <title xml:lang="ko">내 애드온</title>
    <title xml:lang="en">My Addon</title>
    <description xml:lang="ko">설명</description>
    <description xml:lang="en">Description</description>
    <version>1.0.0</version>
    <date>2024-01-01</date>
    <author email_address="dev@example.com" link="https://example.com/">
        <name xml:lang="ko">Author Name</name>
        <name xml:lang="en">Author Name</name>
    </author>
    <extra_vars>
        <var name="my_option" type="select">
            <title xml:lang="ko">옵션</title>
            <title xml:lang="en">Option</title>
            <options value="Y">
                <title xml:lang="ko">사용함</title>
                <title xml:lang="en">Enabled</title>
            </options>
            <options value="N">
                <title xml:lang="ko">사용안함</title>
                <title xml:lang="en">Disabled</title>
            </options>
        </var>
    </extra_vars>
</addon>
```

### Main Script ({addon_name}.addon.php)

```php
<?php
if (!defined('__XE__')) exit();

/**
 * @file my_addon.addon.php
 * @brief My addon description.
 */

// $called_position - current hook position
// $addon_info      - admin settings (extra_vars)
// $output          - HTML output (only in before_display_content)

if ($called_position == 'before_module_init')
{
    // Before module initialization
    // Good for: setting context variables, loading resources
}

if ($called_position == 'after_module_proc')
{
    // After module processing
    // Good for: loading JS/CSS, post-processing
    if (\Context::getResponseMethod() == 'HTML')
    {
        \Context::loadFile(array('./addons/my_addon/my_addon.js', 'body', '', null), true);
    }
}

if ($called_position == 'before_display_content')
{
    // Before HTML output — modify $output variable directly
    $output = str_replace('find', 'replace', $output);
}
```

### called_position Values

| Position | Description | $output Available |
|----------|-------------|:-----------------:|
| `before_module_init` | Before module initialization | No |
| `after_module_proc` | After module processing | No |
| `before_display_content` | Before HTML output | Yes |

---

## Layout (Theme)

### Directory Structure

```
layouts/{layout_name}/
  conf/
    info.xml              # Metadata - REQUIRED
  layout.html             # Main layout template - REQUIRED
  *.css, *.js             # Resources
  thumbnail.png           # Preview image
```

Mobile layouts: `m.layouts/{layout_name}/`

### conf/info.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<layout version="0.2">
    <title xml:lang="ko">레이아웃 이름</title>
    <title xml:lang="en">Layout Name</title>
    <description xml:lang="ko">설명</description>
    <description xml:lang="en">Description</description>
    <version>1.0.0</version>
    <date>2024-01-01</date>
    <author email_address="dev@example.com" link="https://example.com/">
        <name xml:lang="ko">Author Name</name>
        <name xml:lang="en">Author Name</name>
    </author>
    <menus>
        <menu name="GNB" maxdepth="2" default="true">
            <title xml:lang="ko">전역 네비게이션 바</title>
            <title xml:lang="en">Global Navigation Bar</title>
        </menu>
    </menus>
    <extra_vars>
        <var name="LOGO_IMG" type="image">
            <title xml:lang="ko">로고 이미지</title>
            <title xml:lang="en">Logo image</title>
        </var>
        <var name="LOGO_TEXT" type="text">
            <title xml:lang="ko">로고 텍스트</title>
            <title xml:lang="en">Logo text</title>
        </var>
        <var name="COLOR_SCHEME" type="select">
            <title xml:lang="ko">색상 테마</title>
            <title xml:lang="en">Color scheme</title>
            <options value="light">
                <title xml:lang="ko">밝은 색</title>
                <title xml:lang="en">Light</title>
            </options>
            <options value="dark">
                <title xml:lang="ko">어두운 색</title>
                <title xml:lang="en">Dark</title>
            </options>
        </var>
        <var name="FOOTER" type="textarea">
            <title xml:lang="ko">푸터 텍스트</title>
            <title xml:lang="en">Footer text</title>
        </var>
    </extra_vars>
</layout>
```

extra_var types: `text`, `textarea`, `image`, `image_url`, `select`, `checkbox`, `color`

### layout.html

```html
<load target="style.css" />
<load target="script.js" type="body" />

<div class="layout-wrapper">
    <header>
        <h1><a href="{getUrl('')}">{Context::getSiteTitle()}</a></h1>
        <nav>
            <ul>
                <li loop="$GNB->list=>$key,$val"
                    class="active"|cond="$val['selected']">
                    <a href="{$val['href']}"
                       target="_blank"|cond="$val['open_window']=='Y'">
                        {$val['link']}
                    </a>
                    <ul cond="$val['list']">
                        <li loop="$val['list']=>$key2,$val2">
                            <a href="{$val2['href']}">{$val2['link']}</a>
                        </li>
                    </ul>
                </li>
            </ul>
        </nav>
    </header>
    <main id="content">
        {$content|noescape}
    </main>
    <footer>
        <p cond="$layout_info->FOOTER">{$layout_info->FOOTER}</p>
    </footer>
</div>
```

### Key Layout Variables

| Variable | Description |
|----------|-------------|
| `{$content}` | Module rendered output (use `\|noescape`) |
| `$layout_info->VARNAME` | Extra vars defined in info.xml |
| `$GNB` (or menu name) | Menu tree object with `.list` property |

Menu item properties: `href`, `link` (text), `selected`, `open_window`, `class`, `list` (children)

---

## Widget

### Directory Structure

```
widgets/{widget_name}/
  {widget_name}.class.php    # Widget class - REQUIRED
  conf/
    info.xml                  # Metadata + extra_vars - REQUIRED
  queries/                    # Query XML files
  skins/
    {skin_name}/
      list.html               # Skin template
      skin.xml                # Skin metadata
```

### Widget Class (extends WidgetHandler)

```php
<?php

class my_widget extends WidgetHandler
{
    /**
     * Widget handler.
     *
     * @param object $args extra_vars from admin
     * @return string compiled template HTML
     */
    function proc($args)
    {
        $list_count = (int)($args->list_count ?: 5);

        $obj = new stdClass;
        $obj->module_srl = $args->module_srls;
        $obj->list_count = $list_count;
        $output = executeQueryArray('widgets.my_widget.getItems', $obj);

        $widget_info = new stdClass;
        $widget_info->item_list = $output->data;
        \Context::set('widget_info', $widget_info);

        $tpl_path = sprintf('%sskins/%s', $this->widget_path, $args->skin);
        $tpl_file = 'list';
        $oTemplate = \TemplateHandler::getInstance();
        return $oTemplate->compile($tpl_path, $tpl_file);
    }
}
```

### conf/info.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<widget version="0.2">
    <title xml:lang="ko">내 위젯</title>
    <title xml:lang="en">My Widget</title>
    <description xml:lang="ko">설명</description>
    <description xml:lang="en">Description</description>
    <version>1.0.0</version>
    <date>2024-01-01</date>
    <author email_address="dev@example.com" link="https://example.com/">
        <name xml:lang="en">Author</name>
    </author>
    <extra_vars>
        <group>
            <title xml:lang="ko">기본 설정</title>
            <title xml:lang="en">Basic Settings</title>
            <var id="list_count" type="text">
                <name xml:lang="ko">목록 수</name>
                <name xml:lang="en">List count</name>
            </var>
            <var id="content_type" type="select">
                <name xml:lang="ko">추출 대상</name>
                <name xml:lang="en">Content type</name>
                <options value="document">
                    <name xml:lang="ko">게시물</name>
                    <name xml:lang="en">Documents</name>
                </options>
                <options value="comment">
                    <name xml:lang="ko">댓글</name>
                    <name xml:lang="en">Comments</name>
                </options>
            </var>
        </group>
    </extra_vars>
</widget>
```

---

## Widgetstyle

### Directory Structure

```
widgetstyles/{widgetstyle_name}/
  skin.xml              # Metadata - REQUIRED
  widgetstyle.html      # Template - REQUIRED
  style.css             # Styles
  preview.gif           # Preview image
```

### skin.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<widgetstyle>
    <title xml:lang="ko">위젯 스타일</title>
    <title xml:lang="en">Widget Style</title>
    <description xml:lang="ko">설명</description>
    <description xml:lang="en">Description</description>
    <version>1.0.0</version>
    <date>2024-01-01</date>
    <preview>preview.gif</preview>
    <author email_address="dev@example.com" link="https://example.com/">
        <name xml:lang="en">Author</name>
    </author>
    <extra_vars>
        <var name="ws_title" type="text">
            <name xml:lang="ko">제목</name>
            <name xml:lang="en">Title</name>
        </var>
    </extra_vars>
</widgetstyle>
```

### widgetstyle.html

```html
<div class="myWidgetStyle">
    <h2>{$widgetstyle_extra_var->ws_title}</h2>
    {$widget_content}
</div>
```

---

## Skin (for Modules)

### Directory Structure

```
modules/{module}/skins/{skin_name}/
  skin.xml               # Skin metadata + extra_vars
  list.html              # List template
  view.html              # Detail view template
  write_form.html        # Write form
  comment.html           # Comment template
  delete_form.html       # Delete confirmation
  *.css, *.js            # Resources
  thumbnail.png          # Preview
```

### skin.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<skin version="0.2">
    <title xml:lang="ko">스킨 이름</title>
    <title xml:lang="en">Skin Name</title>
    <description xml:lang="ko">설명</description>
    <description xml:lang="en">Description</description>
    <version>1.0</version>
    <date>2024-01-01</date>
    <author email_address="dev@example.com" link="https://example.com/">
        <name xml:lang="en">Author</name>
    </author>
    <license>GPL v2</license>
    <extra_vars>
        <var name="title_image" type="image">
            <title xml:lang="ko">제목 이미지</title>
            <title xml:lang="en">Title image</title>
        </var>
    </extra_vars>
</skin>
```
