# GUI на Golang: GTK+ 3

Решил я написать одно кроссплатформенное десктопное приложение на **Go**. Сделал **CLI**-версию, всё работает отлично. Да ещё и кросскомпиляция в **Go** поддерживается. Всё в общем отлично. Но понадобилась также и **GUI**-версия. И тут началось...

![Golang gotk3](https://raw.githubusercontent.com/jhekasoft/articles/master/01_golang_gtk3/images/go_gotk3.png)
<cut />

## Выбор библиотеки (биндинга) для GUI

Приложение должно было быть кроссплатформенным.
Поэтому должно компилироваться под **Windows**, **GNU/Linux** и **macOS**.
Выбор пал на такие библиотеки:

- [gotk3](https://github.com/gotk3/gotk3) (GTK+ 3)
- [therecipe/qt](https://github.com/therecipe/qt) (Qt)
- [zserge/webview](https://github.com/zserge/webview) (нативный вебвью)

**Electron** и прочие фреймворки, которые тянут с собой **Chromium** и **node.js**, я откинул так как они весят достаточно много, ещё и съедают много ресурсов операционной системы.

Теперь немного о каждой библиотеке.

### gotk3

Биндинг библиотеки **GTK+ 3**. Покрытие далеко не всех возможностей, но всё основное присутсвует.

Компилируется приложение с помощью стандартного `go build`. [Кроссплатформенная компиляция](https://github.com/gotk3/gotk3/wiki/Cross-Compiling) возможна, за исключением **macOS**. Только с **macOS** можно скомпилировать под эту ОС, ну и с **macOS** можно будет скомпилировать и под **Windows** + **GNU/Linux**.

Интерфейс будет выглядить нативно для **GNU/Linux**, **Windows** (нужно будет указать специальную тему). Для **macOS** будет выглядеть не нативно. Выкрутиться можно только разве что страшненькой темой, которая будет эмулирувать нативные элементы **macOS**.

### therecipe/qt

Биндинг библиотеки **Qt 5**. Поддержка QML, стандартных виджетов. Вообще этот биндинг многие советуют.

Компилируется с помощью специальной команды `qtdeploy`. Кроме десктопных платформ есть также и мобильные. Кросскомпиляция происходит с помощью [Docker](https://github.com/therecipe/qt/wiki/Deploying-Linux-to-Windows-64-bit-Static#using-docker-image). Под операционные системы **Apple** можно скомпилировать только с [macOS](https://github.com/therecipe/qt#deployment-targets).

При желании на **Qt** можно добиться чтобы интерфейс выглядел нативно на десктопных ОС.

### zserge/webview

Библиотека, которая написана изначально на **C**, автор прикрутил её ко многим языкам, в том числе и к **Go**. Использывается нативный [webview](https://github.com/zserge/webview#webview) для отображения: **Windows** — **MSHTML**, **GNU/Linux** — **gtk-webkit2**, **macOS** — **Cocoa/WebKit**. Кроме кода на **Go** нужно будет и на **JS** пописать, ну и **HTML** пригодится.

Компилируется при помощи `go build`, кросскомпиляция возможна с помощью [xgo](https://github.com/karalabe/xgo).

Выглядеть нативно может настолько насколько позволит стандартный браузер.

### Выбор

Почему же я выбрал именно **gotk3**?

В **therecipe/qt** мне не понравилась слишком сложная система сборки приложения, даже специальную команду сделали.

**zserge/webview** вроде бы не плох, весить будет не много, но всё-таки это **webview** и могут быть стандартные проблемы, которые бывают в таких приложениях: может что-то где-то поехать. И это не **Electron**, где всегда в комплекте продвинутый **Chromium**, а в какой-нибудь старой **Windows** может всё поехать. Да и к тому же придётся ещё и на **JS** писать.

**gotk3** я выбрал как что-то среднее. Можно собирать стандартным `go build`, выглядит приемлемо, да и вообще я **GTK+ 3** люблю!

В общем я думал, что всё будет просто. И что зря про **Go** говорят, что в нём проблема с **GUI**. Но как же я ошибался...

## Начинаем

Устанавливаем всё из **gotk3** (**gtk**, **gdk**, **glib**, **cairo**) себе:

```bash
go get github.com/gotk3/gotk3/...
```

Также у вас в системе должна быть установлена сама библиотека **GTK+ 3** для разработки.

### GNU/Linux

В **Ubuntu**:

```bash
sudo apt-get install libgtk-3-dev
```

В **Arch Linux**:

```bash
sudo pacman -S gtk3
```

### macOS

Через **Homebrew**:

```bash
 brew install gtk-mac-integration gtk+3
```

### Windows

Здесь всё не так просто. В [официальной инструкции](https://www.gtk.org/download/windows.php) предлагают использовать **MSYS2** и уже в ней всё делать. Лично я писал код на других операционных системах, а кросскомпиляцию для **Windows** делал в **Arch Linux**, о чём надеюсь скоро напишу.

## Простой пример

Теперь пишем небольшой файл с кодом `main.go`:

```go
package main

import (
    "log"

    "github.com/gotk3/gotk3/gtk"
)

func main() {
    // Инициализируем GTK.
    gtk.Init(nil)

    // Создаём окно верхнего уровня, устанавливаем заголовок
    // И соединяем с сигналом "destroy" чтобы можно было закрыть
    // приложение при закрытии окна
    win, err := gtk.WindowNew(gtk.WINDOW_TOPLEVEL)
    if err != nil {
        log.Fatal("Не удалось создать окно:", err)
    }
    win.SetTitle("Простой пример")
    win.Connect("destroy", func() {
        gtk.MainQuit()
    })

    // Создаём новую метку чтобы показать её в окне
    l, err := gtk.LabelNew("Привет, gotk3!")
    if err != nil {
        log.Fatal("Не удалось создать метку:", err)
    }

    // Добавляем метку в окно
    win.Add(l)

    // Устанавливаем размер окна по умолчанию
    win.SetDefaultSize(800, 600)

    // Отображаем все виджеты в окне
    win.ShowAll()

    // Выполняем главный цикл GTK (для отрисовки). Он остановится когда
    // выполнится gtk.MainQuit()
    gtk.Main()
}
```

Спомпилировать можно с помощью команды `go build`, а потом запустить бинарник. Но мы просто запустим его:

```bash
go run main.go
```

После запуска получим окно такого вида:

![Простой пример на Golang gotk3](https://raw.githubusercontent.com/jhekasoft/articles/master/01_golang_gtk3/images/go_simple.png)

Поздравляю! У вас получилось простое приложение из [README](https://github.com/gotk3/gotk3#sample-use) **gotk3**!

Больше примеров можно найти на [Github gotk3](https://github.com/gotk3/gotk3-examples/). Их разбирать я не буду. Давайте лучше займёмся тем, чего нет в примерах!

## Glade

Есть такая вещь для **Gtk+ 3** — **Glade**. Это конструктор графических интерфейсов для **GTK+**. Выглядит примерно так:

![Glade](https://raw.githubusercontent.com/jhekasoft/articles/master/01_golang_gtk3/images/glade.png)

Чтобы вручную не создавать каждый элемент и не помещать его потом где-то в окне с помощью программного кода, можно весь дизайн накидать в **Glade**. Потом сохранить всё в **XML-подобный** файл **\*.glade** и загрузить его уже через наше приложение.

### Установка Glade

#### GNU/Linux

В дистрибутивах **GNU/Linux** установить **glade** не составит труда. В какой-нибудь **Ubuntu** это будет:

```bash
sudo apt-get install glade
```

В **Arch Linux**:

```bash
sudo pacman -S glade
```

#### macOS

В загрузках с официального сайта очень старая сборка. Поэтому устанавливать лучше через **Homebrew**:

```bash
brew install glade
```

А запускать потом:

```bash
glade
```

#### Windows

Скачать не самую последнюю версию можно [здесь](http://ftp.gnome.org/pub/GNOME/binaries/win32/glade/). Я лично на **Windows** вообще не устанавливал, поэтому не знаю насчёт стабильность работы там **Glade**.

### Простое приложение с использованием Glade

В общем надизайнил я примерно такое окно:

![Glade](https://raw.githubusercontent.com/jhekasoft/articles/master/01_golang_gtk3/images/glade_simple_main.png)

Сохранил и получил файл `main.glade`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Generated with glade 3.22.1 -->
<interface>
  <requires lib="gtk+" version="3.20"/>
  <object class="GtkWindow" id="window_main">
    <property name="title" translatable="yes">Пример Glade</property>
    <property name="can_focus">False</property>
    <child>
      <placeholder/>
    </child>
    <child>
      <object class="GtkBox">
        <property name="visible">True</property>
        <property name="can_focus">False</property>
        <property name="margin_left">10</property>
        <property name="margin_right">10</property>
        <property name="margin_top">10</property>
        <property name="margin_bottom">10</property>
        <property name="orientation">vertical</property>
        <property name="spacing">10</property>
        <child>
          <object class="GtkEntry" id="entry_1">
            <property name="visible">True</property>
            <property name="can_focus">True</property>
          </object>
          <packing>
            <property name="expand">False</property>
            <property name="fill">True</property>
            <property name="position">0</property>
          </packing>
        </child>
        <child>
          <object class="GtkButton" id="button_1">
            <property name="label" translatable="yes">Go</property>
            <property name="visible">True</property>
            <property name="can_focus">True</property>
            <property name="receives_default">True</property>
          </object>
          <packing>
            <property name="expand">False</property>
            <property name="fill">True</property>
            <property name="position">1</property>
          </packing>
        </child>
        <child>
          <object class="GtkLabel" id="label_1">
            <property name="visible">True</property>
            <property name="can_focus">False</property>
            <property name="label" translatable="yes">This is label</property>
          </object>
          <packing>
            <property name="expand">False</property>
            <property name="fill">True</property>
            <property name="position">2</property>
          </packing>
        </child>
      </object>
    </child>
  </object>
</interface>
```

То есть у нас получилось окно `window_main` (`GtkWindow`), в котором внутри контейнер (`GtkBox`), который содержит поле ввода `entry_1` (`GtkEntry`), кнопку `button_1` (`GtkButton`) и метку `label_1` (`GtkLabel`). Кроме этого ещё имеются аттрибуты отсупов (я настроил немного), видимость и другие аттрибуты, которые **Glade** добавила автоматически.

Давайте теперь попробуем загрузить это представление в нашем `main.go`:

```go
package main

import (
    "log"

    "github.com/gotk3/gotk3/gtk"
)

func main() {
    // Инициализируем GTK.
    gtk.Init(nil)

    // Создаём билдер
    b, err := gtk.BuilderNew()
    if err != nil {
        log.Fatal("Ошибка:", err)
    }

    // Загружаем в билдер окно из файла Glade
    err = b.AddFromFile("main.glade")
    if err != nil {
        log.Fatal("Ошибка:", err)
    }

    // Получаем объект главного окна по ID
    obj, err := b.GetObject("window_main")
    if err != nil {
        log.Fatal("Ошибка:", err)
    }

    // Преобразуем из объекта именно окно типа gtk.Window
    // и соединяем с сигналом "destroy" чтобы можно было закрыть
    // приложение при закрытии окна
    win := obj.(*gtk.Window)
    win.Connect("destroy", func() {
        gtk.MainQuit()
    })

    // Отображаем все виджеты в окне
    win.ShowAll()

    // Выполняем главный цикл GTK (для отрисовки). Он остановится когда
    // выполнится gtk.MainQuit()
    gtk.Main()
}
```

Снова запускаем:

```bash
go run main.go
```

И получаем:

![Golang Glade gotk3](https://raw.githubusercontent.com/jhekasoft/articles/master/01_golang_gtk3/images/go_glade_main.png)

Ура! Теперь мы представление формы держим **XML**-подобном `main.glade` файле, а код в `main.go`!

## Сигналы

Окно запускается, но давайте добавим интерактивности. Пусть текст из поля ввода при нажатии на кнопку попадёт в метку.

Для этого для начала получим элементы поля ввода, кнопки и метке в коде:

```go
// Получаем поле ввода
obj, _ = b.GetObject("entry_1")
entry1 := obj.(*gtk.Entry)

// Получаем кнопку
obj, _ = b.GetObject("button_1")
button1 := obj.(*gtk.Button)

// Получаем метку
obj, _ = b.GetObject("label_1")
label1 := obj.(*gtk.Label)
```

Я не обрабатываю ошибки, которые возвращает функция `GetObject()`, для того, чтобы код был более простым. Но в реальном рабочем приложении их обязательно необходимо обработать.

Хорошо. С помощью кода выше мы получаем наши элементы формы. А теперь давайте обработаем сигнал `clicked` кнопку (когда кнопка нажата). Сигнал **GTK+** — это по сути реакция на событие. Добавим код:

```go
// Сигнал по нажатию на кнопку
button1.Connect("clicked", func() {
    text, err := entry1.GetText()
    if err == nil {
        // Устанавливаем текст из поля ввода метке
        label1.SetText(text)
    }
})
```

Теперь запускаем код:

```bash
go run main.go
```

После ввода какого-нибудь текста в поле и нажатию по кнопке **Go** мы увидем этот текст в метке:

![Golang Glade gotk3 signal](https://raw.githubusercontent.com/jhekasoft/articles/master/01_golang_gtk3/images/go_glade_signal.png)

Теперь у нас интерактивное приложение!

## Заключение

На данном этапе всё кажется простым и не вызывает трудностей. Но трудности у меня появились при кросскомпиляции (ведь **gotk3** компилируется с **CGO**), интеграции с операционными системами и с диалогом выбора файла. Я даже [добавил](https://github.com/gotk3/gotk3/pull/267) в проект **gotk** нативный диалог. Также в моём проекте нужна была интернационализация. Там тоже есть некоторые особенности. Если вам интересно увидеть это всё сейчас в коде, то можно подсмотреть [здесь](https://github.com/jhekasoft/insteadman3/tree/master/gtk).

Исходые коды примеров из статьи находятся [здесь](https://github.com/jhekasoft/articles/tree/master/01_golang_gtk3/example).

А если хотите почитать продолжение, то можете проголосовать. И в случае, если окажется это кому-нибудь интересным, я продолжу писать.
