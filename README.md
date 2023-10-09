Здесь  я буду представлять решения задач,  которые я поставил перед собой.

## Проблема 1. 
Я хочу, чтобы таблица не имела внешних отступов или полей на втором слайде при включенных опциях (autoPageRepeatHeader : true, autoPage: true), а также положения таблицы (x: 0, y: 0). Установление опции *newSlideStartY* в 0 не дает результата.

***Решение:***
Здесь я убираю отступы (**margin**) со всех четырех сторон
```js
const pptx = new pptxgen()

pptx.defineSlideMaster({
    title: "TEST",
    margin: [0, 0, 0, 0]
})

let slide = pptx.addSlide({masterName: "TEST"})
```
 
