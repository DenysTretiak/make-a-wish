# Привет!

Немного магии перед стартом. 

## Prerequisites

Сделай, пожалуйста, **приватный** форк этого репо.

## Setup

```
cd make-a-wish
npm install
npm run dev
```

## Show time

В этом репо всего одна страничка, похожая на детали вакансии. К кнопкам “Откликнуться” и “Добавить в избранное” надо добавить третью кнопку “Ридикулус”, которая по аналогии с “Откликнуться” открывает magic form:


![magic form](https://i.ibb.co/0FWjj1M/IMG-7913.jpg)


Magic Form  состоит из autosuggest (1), текстарии с плейсхолдером “Type a spell…” (2), свитчером (3) с текстом «Торжественно клянусь, что замышляю шалость, и только шалость!» и кнопок Отмена и Сохранить (4)

### Autosuggest (1)

Для autosuggest использовать опции(tags) в `src/data/categories.js`
* даже если инпут пуст, всегда показываются 7 опций под ним (1.2)
* можно нажать на опцию с тремя точками […] (1.3 на скетче), чтобы показать еще 7 опций
* при клике на опцию она становится первой в списке и меняет цвет (цвет фона: $red, цвет текста: white) - (1.1)
* можно выбрать несколько опций
* при повторном клике опция сбрасывается (меняет цвет на дефолтный и становится в очереди за выбранными опциями)
В качестве референса можно смотреть на фильтры в [Вакансиях](https://djinni.co/jobs2)

👆 **Хозяйке на заметку:** большинство компонентов есть в `/src/components`. Но можно создавать свои. 

### Валидация формы

Кнопка Сохранить(4.) активна при условии, что текстариа (2.) не пуста и включен свитчер (3.)

### Кнопка "Сохранить"

По нажатию на кнопку Сохранить отправить POST реквест https://djinni.co/api/domagic c данными:

```
{
	options: ["some-option-value", "java", "android", ...],
	spell: "some spell"
}
```

### Great success!

В случае ответа со статусом 200 показать зеленую плашку “Шалость удалась!” (есть в темплейте) и скрыть форму.


Расшарьте нам ваш форк (контакты вышлем). 


Спасибо.


![magic](https://media.giphy.com/media/4xqGxx0rED4ze/giphy.gif)



