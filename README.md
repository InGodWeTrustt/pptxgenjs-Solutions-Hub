# pptxgenjs-Solutions-Hub
Здесь  я буду представлять решения задач,  которые я поставил перед собой.

## Проблема 1. 
Я хочу, чтобы таблица не имела отступа на втором слайде при включенной опции (autoPageRepeatHeader : true, autoPage: true). Установление опции *newSlideStartY* в 0 не дает результата.

***Решение:***
Здесь я убираю отступы со всех четырех сторон
```js
const pptx = new pptxgen()

pptx.defineSlideMaster({
    title: "TEST",
    margin: [0, 0, 0, 0]
})

let slide = pptx.addSlide({masterName: "TEST"})
```
 
