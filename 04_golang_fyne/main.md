# GUI на Golang: Fyne

## Как работает

## Установка

```bash
go get fyne.io/fyne
```

### Запуск примеров

```bash
go get fyne.io/fyne/cmd/fyne_demo
```

```bash
fyne_demo
```

## Пишем простое десктопное приложение

```go
package main

import (
  "fyne.io/fyne/app"
  "fyne.io/fyne/widget"
)

func main() {
  a := app.New()

  w := a.NewWindow("Приветули")
  w.SetContent(widget.NewVBox(
    widget.NewLabel("Приветули, Fyne!"),
    widget.NewButton("Выйти", func() {
      a.Quit()
    }),
  ))

  w.ShowAndRun()
}
```

```bash
go run hello.go
```

Тема
`FYNE_THEME=light`

```go
app.Settings().SetTheme(theme.LightTheme())
```

## Виджеты

Список виджетов (https://fyne.io/develop/widgets.html). Есть диалоги уведомления/ошибки/подтверждения. Есть скролбары.

Нет таблиц (https://github.com/fyne-io/fyne/issues/157), нет файлового диалога (https://github.com/fyne-io/fyne/issues/225).

### Расширение виджетов

Можно расширять существующие виджеты. Например, если необходимо добавить возможно нажимать на иконку, то достаточно этого:

https://fyne.io/develop/extending-widgets.html

```go
//...
type tappableIcon struct {
  widget.Icon
}

func newTappableIcon(res fyne.Resource) *tappableIcon {
  icon := &tappableIcon{}
  icon.ExtendBaseWidget(icon)
  icon.SetResource(res)

  return icon
}
```

### Нативные меню

## Кросскомпиляция

## Сборка и дистрибьюция

## Тестирование
