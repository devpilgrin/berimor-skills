---
name: docx
version: 1.0.0
description: "Используй этот скилл всякий раз, когда пользователь хочет создать, прочитать, отредактировать или обработать документы Word (файлы .docx) или шаблоны Word (файлы .dotx): создание профессиональных документов с оглавлением, заголовками, номерами страниц, фирменными бланками; извлечение и реорганизация содержимого; вставка и замена изображений; поиск и замена в Word; работа с исправлениями (tracked changes) и комментариями; отчёты, служебные записки, письма, шаблоны как .docx. НЕ использовать для PDF, электронных таблиц, Google Docs и общих задач кодирования."
triggers:
  - "word"
  - "docx"
  - "документ word"
  - ".docx"
  - "отчёт в word"
tools:
  - files.read
  - files.write
  - files.edit
  - files.list
  - files.search
  - terminal.exec
license: Proprietary. LICENSE.txt has complete terms
---

# Создание, редактирование и анализ DOCX

`.docx` — это ZIP-архив из XML-файлов. Выбирай подход по задаче:

| Задача | Подход |
|---|---|
| **Создать** новый документ | Написать скрипт на `docx` (npm) — см. подводные камни ниже |
| **Редактировать** существующий документ | `unzip` → правка `word/document.xml` → `zip` (docx-js не умеет открывать существующие файлы) |
| **Прочитать** содержимое | `pandoc -t markdown file.docx` |

> Пути к скриптам ниже указаны относительно каталога этого скилла.

## Создание через docx-js — подводные камни

`docx` предустановлен — не запускай `npm install` заранее; пиши скрипт и вызывай `require('docx')` напрямую. Только если require падает: `npm install docx`. Модель знает API; вот типичные ловушки:

- **Размер страницы по умолчанию — A4.** Для US Letter задай `page: { size: { width: 12240, height: 15840 } }` (DXA; 1440 = 1″).
- **Альбомная ориентация:** передавай книжные размеры и `orientation: PageOrientation.LANDSCAPE` — docx-js сам меняет ширину/высоту местами.
- **Таблицам нужны двойные ширины:** задавай `columnWidths` у таблицы И `width` у каждой ячейки, оба в `WidthType.DXA` (PERCENTAGE ломается в Google Docs). Сумма ширин колонок должна равняться ширине таблицы.
- **Заливка таблицы:** используй `ShadingType.CLEAR`, никогда `SOLID` (отрисуется чёрным).
- **Списки:** никогда не вставляй `•` буквально; используй конфиг `numbering` с `LevelFormat.BULLET`.
- **`ImageRun` требует `type:`** (`"png"`, `"jpg"`, …).
- **`PageBreak` должен быть внутри `Paragraph`.**
- **Никогда не используй `\n`** — используй отдельные элементы `Paragraph`.
- **Оглавление (TOC):** заголовки должны использовать встроенные `HeadingLevel.*`; у пользовательских стилей заголовков нужно задать `outlineLevel`, иначе они не попадут в оглавление.
- **Не используй таблицу как горизонтальную линию** — вместо этого используй нижнюю границу абзаца.
- **Точечный лидер / выравнивание вправо на той же строке:** используй `PositionalTab` (`alignment: PositionalTabAlignment.RIGHT`, `leader: PositionalTabLeader.DOT`) внутри `TextRun`, а не буквальные `.` или отбивку пробелами.

## Проверка результата

После записи `.docx` отрендерь его и посмотри на него:

```bash
python scripts/office/soffice.py --headless --convert-to pdf output.docx
pdftoppm -jpeg -r 100 output.pdf page
ls page-*.jpg   # then Read the images
```

`pdftoppm` дополняет номера страниц нулями до ширины общего числа страниц (`page-01.jpg`…`page-12.jpg`).

## Редактирование существующих документов

Устаревшие файлы `.doc` сначала нужно конвертировать: `python scripts/office/soffice.py --headless --convert-to docx file.doc`.

```bash
unzip -q doc.docx -d unpacked/
find unpacked -type l -delete   # strip symlink entries — docx from external parties is untrusted
python scripts/merge_runs.py unpacked/   # coalesce fragmented runs so text is findable
# edit unpacked/word/document.xml in place — do NOT reformat or pretty-print
(cd unpacked && rm -f ../out.docx && zip -Xr ../out.docx .)
python scripts/office/validate.py out.docx --original doc.docx   # XSD checks; --auto-repair fixes common issues
# redlining? add --author "<the name you redlined under>" to check every edit is tracked
```

Word разбивает текст на множество run'ов `<w:r>` (идентификаторы ревизий, метки проверки орфографии), поэтому фраза, которую ты видишь в документе, часто не существует в XML как непрерывная строка. `merge_runs.py` объединяет соседние одинаково отформатированные run'ы в `word/document.xml`, не меняя содержимое и отрисовку; он также принимает `.docx` напрямую (`python scripts/merge_runs.py doc.docx -o merged.docx`).

**Отслеживаемые изменения (tracked changes):** при redlining валидируй с `--author "<имя, под которым правил>"` (требует `--original`) — скрипт сообщит о любом тексте, который ты изменил без обёртки `<w:ins>`/`<w:del>`, что легко сделать случайно и невидимо в принятом виде. Оборачивай run'ы в `<w:ins>`/`<w:del>` с атрибутами `w:id`, `w:author`, `w:date`. Внутри `<w:del>` текстовый элемент — `<w:delText>`, а не `<w:t>`. Удалённый знак абзаца (`<w:pPr><w:rPr><w:del w:id=".." w:author=".." w:date=".."/></w:rPr></w:pPr>`) означает «слить этот абзац со следующим» — поэтому полное удаление абзаца — это плюс `<w:del>` вокруг каждого run'а. `<w:del/>` должен идти перед остальными дочерними элементами rPr; их порядок закреплён схемой.

Чтобы получить чистую копию со всеми принятыми правками: `python scripts/accept_changes.py in.docx out.docx`.

Принятие удалённого знака абзаца должно присоединять этот абзац к нижележащему, поэтому абзац, чьи run'ы удалены *все*, исчезает. Word так делает; `accept_changes.py` и `pandoc --track-changes=accept` — не всегда. Оба падают одинаково — они вырезают удалённый текст, но оставляют опустевший абзац, который читается как лишний пустой пункт, если абзац был автонумерованным:

- `pandoc --track-changes=accept` никогда не объединяет абзацы.
- `accept_changes.py` (LibreOffice) объединяет их корректно, кроме случая, когда за удалённым абзацем следует пустой абзац-разделитель.

Пустой пункт в любом из видов — артефакт этого вида, а не дефект документа. Проверяй удаления абзацев в XML.

## Комментарии

Комментарии требуют шести перекрёстно связанных файлов. Используй хелпер — режим каталога, если ты также будешь редактировать `document.xml` (экономит цикл unzip/rezip), и режим `.docx`-напрямую в остальных случаях:

```bash
# Against an already-unpacked directory (preferred when also placing markers)
python scripts/comment.py unpacked/ "Fees & expenses cap is too low"
python scripts/comment.py unpacked/ "Agreed" --parent 0

# Against a .docx directly
python scripts/comment.py contract.docx "This cap is too low" -o annotated.docx
```

Скрипт записывает `comments.xml`, `commentsExtended.xml`, `commentsIds.xml`, `commentsExtensible.xml`, связи (relationships) и переопределения content-type. ID комментариев назначаются автоматически. Затем он печатает фрагмент `<w:commentRangeStart>`/`<w:commentRangeEnd>`/`<w:commentReference>`, который нужно добавить в `word/document.xml`, чтобы комментарий привязался к конкретному тексту — пока ты не расставишь эти маркеры, комментарий существует, но не виден.

## Зависимости

`docx` (npm, предустановлен — ставь, только если `require('docx')` падает) · `pandoc` · LibreOffice (`soffice`) · `pdftoppm` (Poppler)
