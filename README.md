Здесь  я буду представлять решения задач,  которые я поставил перед собой.

## Проблема 1. 
Я хочу, чтобы таблица не имела внешних отступов или полей на втором слайде при включенных опциях (autoPageRepeatHeader : true, autoPage: true), а также положения таблицы (x: 0, y: 0). Установление опции *newSlideStartY* в 0 не дает результата ([**причина тут**](#пояснение)).

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

# пояснение

f dvfdfvd

## Проблема 2. 
 Стоит задача в том, чтобы логика работы определенной функции при включенной опции **autopage:true** работала немного иначе, а именно: переносила всю строку на следующий слайд.
 Для этого, необходимо ввести дополнительную переменную, которая будет хранить высоту текущей строки. Это нужно для переменной **emuTabCurrH**, которая для нового слайда будет равна в начале нулю, а затем нужно будет прибавить высоту текущей строки.

 Ниже исправленный фрагмент части функции **getSlidesForTableRows**:
```js
function getSlidesForTableRows(tableRows, tableProps, presLayout, masterSlide) {
...
// Максимальная высота строки таблицы
 var maxCurTableRowHeight = 0;
...
 while (!isDone) {
            var srcCell = rowCellLines[currCellIdx];
            var tgtCell = currTableRow[currCellIdx]; // NOTE: may be redefined below (a new row may be created, thus changing this value)
            // 1: calc emuLineMaxH
            rowCellLines.forEach(function (cell) {
                if (cell._lineHeight >= emuLineMaxH)
                    emuLineMaxH = cell._lineHeight;
            });
            // 3: set array of words that comprise this line
            var currLine = srcCell._lines.shift();
            // 4: create new line by adding all words from curr line (or add empty if there are no words to avoid "needs repair" issue triggered when cells have null content)
            if (Array.isArray(tgtCell.text)) {
                if (currLine)
                    tgtCell.text = tgtCell.text.concat(currLine);
                else if (tgtCell.text.length === 0)
                    tgtCell.text = tgtCell.text.concat({ _type: SLIDE_OBJECT_TYPES.tablecell, text: '' });
                // IMPORTANT: ^^^ add empty if there are no words to avoid "needs repair" issue triggered when cells have null content
            }
            // 5: increase table height by the curr line height (if we're on the last column)
            if (currCellIdx === rowCellLines.length - 1) {
                emuTabCurrH += emuLineMaxH;
                maxCurTableRowHeight += emuLineMaxH;
            }
            // 6: advance column/cell index (or circle back to first one to continue adding lines)
            currCellIdx = currCellIdx < rowCellLines.length - 1 ? currCellIdx + 1 : 0;
            // 7: done?
            var brent = rowCellLines.map(function (cell) { return cell._lines.length; }).reduce(function (prev, next) { return prev + next; });
            if (brent === 0)
                isDone = true;
        }

        // 2: create a new slide if there is insufficient room for the current row
        if (emuTabCurrH + emuLineMaxH > emuSlideTabH) {

            // B: add current slide to Slides array
            tableRowSlides.push(newTableRowSlide);
            // E: Calc usable vertical space/table height now as we may still be in the same row and code above ("C: Calc usable vertical space/table height.") calc may now be invalid
            calcSlideTabH();
            emuTabCurrH += maxCellMarTopEmu + maxCellMarBtmEmu; // Start row height with margins
            // F: reset current table height for this new Slide
            emuTabCurrH = 0;
            // Добавляю высоту строки таблицы к значению emuTabCurrH
            emuTabCurrH += maxCurTableRowHeight;
            maxCurTableRowHeight = 0;
            // C: reset working/curr slide to hold rows as they're created
            var newRows = [];
            newTableRowSlide = { rows: newRows };

            // G: handle repeat headers option /or/ Add new empty row to continue current lines into
            if ((tableProps.addHeaderToEach || tableProps.autoPageRepeatHeader) && tableProps._arrObjTabHeadRows) {
                tableProps._arrObjTabHeadRows.forEach(function (row) {
                    var newHeadRow = [];
                    var maxLineHeight = 0;
                    row.forEach(function (cell) {
                        newHeadRow.push(cell);
                        if (cell._lineHeight > maxLineHeight)
                            maxLineHeight = cell._lineHeight;
                    });
                    newTableRowSlide.rows.push(newHeadRow);
                    emuTabCurrH += maxLineHeight;
                });
            }


            // A: add current row slide or it will be lost (only if it has rows and text)
            if (currTableRow.length > 0 && currTableRow.map(function (cell) { return cell.text.length; }).reduce(function (p, n) { return p + n; }) > 0) {
                newTableRowSlide.rows.push(currTableRow);
            }
            // D: reset working/curr row
            currTableRow = [];
            row.forEach(function (cell) { return currTableRow.push({ _type: SLIDE_OBJECT_TYPES.tablecell, text: [], options: cell.options }); });
        } else if (currTableRow.length > 0) {  // F: Flush/capture row buffer before it resets at the top of this loop
            maxCurTableRowHeight = 0;
            newTableRowSlide.rows.push(currTableRow);
        }
...
```
где три точки(...) - неизменяемый код
