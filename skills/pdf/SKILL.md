---
name: pdf
version: 1.0.0
description: "Используй этот скилл всякий раз, когда пользователь хочет что-либо сделать с PDF-файлами: чтение или извлечение текста/таблиц из PDF, объединение нескольких PDF в один, разделение PDF, поворот страниц, добавление водяных знаков, создание новых PDF, заполнение PDF-форм, шифрование/дешифрование PDF, извлечение изображений и OCR отсканированных PDF для превращения их в поисковые. Если пользователь упоминает .pdf файл или просит его создать — используй этот скилл."
triggers:
  - "pdf"
  - ".pdf"
  - "пдф"
  - "объедини pdf"
  - "заполни форму pdf"
tools:
  - files.read
  - files.write
  - files.edit
  - files.list
  - files.search
  - terminal.exec
permissions:
  - fs-write
  - exec
license: Proprietary. LICENSE.txt has complete terms
---

# Руководство по обработке PDF

## Обзор

Это руководство покрывает основные операции обработки PDF с помощью Python-библиотек и консольных инструментов. За продвинутыми возможностями, JavaScript-библиотеками и подробными примерами обращайся к REFERENCE.md. Если нужно заполнить PDF-форму, прочитай FORMS.md и следуй его инструкциям.

## Быстрый старт

```python
from pypdf import PdfReader, PdfWriter

# Read a PDF
reader = PdfReader("document.pdf")
print(f"Pages: {len(reader.pages)}")

# Extract text
text = ""
for page in reader.pages:
    text += page.extract_text()
```

## Python-библиотеки

### pypdf — базовые операции

#### Объединение PDF
```python
from pypdf import PdfWriter, PdfReader

writer = PdfWriter()
for pdf_file in ["doc1.pdf", "doc2.pdf", "doc3.pdf"]:
    reader = PdfReader(pdf_file)
    for page in reader.pages:
        writer.add_page(page)

with open("merged.pdf", "wb") as output:
    writer.write(output)
```

#### Разделение PDF
```python
reader = PdfReader("input.pdf")
for i, page in enumerate(reader.pages):
    writer = PdfWriter()
    writer.add_page(page)
    with open(f"page_{i+1}.pdf", "wb") as output:
        writer.write(output)
```

#### Извлечение метаданных
```python
reader = PdfReader("document.pdf")
meta = reader.metadata
print(f"Title: {meta.title}")
print(f"Author: {meta.author}")
print(f"Subject: {meta.subject}")
print(f"Creator: {meta.creator}")
```

#### Поворот страниц
```python
reader = PdfReader("input.pdf")
writer = PdfWriter()

page = reader.pages[0]
page.rotate(90)  # Rotate 90 degrees clockwise
writer.add_page(page)

with open("rotated.pdf", "wb") as output:
    writer.write(output)
```

### pdfplumber — извлечение текста и таблиц

#### Извлечение текста с раскладкой
```python
import pdfplumber

with pdfplumber.open("document.pdf") as pdf:
    for page in pdf.pages:
        text = page.extract_text()
        print(text)
```

#### Извлечение таблиц
```python
with pdfplumber.open("document.pdf") as pdf:
    for i, page in enumerate(pdf.pages):
        tables = page.extract_tables()
        for j, table in enumerate(tables):
            print(f"Table {j+1} on page {i+1}:")
            for row in table:
                print(row)
```

#### Продвинутое извлечение таблиц
```python
import pandas as pd

with pdfplumber.open("document.pdf") as pdf:
    all_tables = []
    for page in pdf.pages:
        tables = page.extract_tables()
        for table in tables:
            if table:  # Check if table is not empty
                df = pd.DataFrame(table[1:], columns=table[0])
                all_tables.append(df)

# Combine all tables
if all_tables:
    combined_df = pd.concat(all_tables, ignore_index=True)
    combined_df.to_excel("extracted_tables.xlsx", index=False)
```

### reportlab — создание PDF

#### Базовое создание PDF
```python
from reportlab.lib.pagesizes import letter
from reportlab.pdfgen import canvas

c = canvas.Canvas("hello.pdf", pagesize=letter)
width, height = letter

# Add text
c.drawString(100, height - 100, "Hello World!")
c.drawString(100, height - 120, "This is a PDF created with reportlab")

# Add a line
c.line(100, height - 140, 400, height - 140)

# Save
c.save()
```

#### Создание PDF из нескольких страниц
```python
from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, PageBreak
from reportlab.lib.styles import getSampleStyleSheet

doc = SimpleDocTemplate("report.pdf", pagesize=letter)
styles = getSampleStyleSheet()
story = []

# Add content
title = Paragraph("Report Title", styles['Title'])
story.append(title)
story.append(Spacer(1, 12))

body = Paragraph("This is the body of the report. " * 20, styles['Normal'])
story.append(body)
story.append(PageBreak())

# Page 2
story.append(Paragraph("Page 2", styles['Heading1']))
story.append(Paragraph("Content for page 2", styles['Normal']))

# Build PDF
doc.build(story)
```

#### Нижние и верхние индексы

**ВАЖНО**: Никогда не используй Unicode-символы нижних/верхних индексов (₀₁₂₃₄₅₆₇₈₉, ⁰¹²³⁴⁵⁶⁷⁸⁹) в PDF на ReportLab. Встроенные шрифты не содержат этих глифов, из-за чего они рендерятся сплошными чёрными квадратами.

Вместо этого используй XML-разметку ReportLab в объектах Paragraph:
```python
from reportlab.platypus import Paragraph
from reportlab.lib.styles import getSampleStyleSheet

styles = getSampleStyleSheet()

# Subscripts: use <sub> tag
chemical = Paragraph("H<sub>2</sub>O", styles['Normal'])

# Superscripts: use <super> tag
squared = Paragraph("x<super>2</super> + y<super>2</super>", styles['Normal'])
```

Для текста, рисуемого через canvas (не объекты Paragraph), вручную подбирай размер шрифта и позицию вместо Unicode-индексов.

## Консольные инструменты

### pdftotext (poppler-utils)
```bash
# Extract text
pdftotext input.pdf output.txt

# Extract text preserving layout
pdftotext -layout input.pdf output.txt

# Extract specific pages
pdftotext -f 1 -l 5 input.pdf output.txt  # Pages 1-5
```

### qpdf
```bash
# Merge PDFs
qpdf --empty --pages file1.pdf file2.pdf -- merged.pdf

# Split pages
qpdf input.pdf --pages . 1-5 -- pages1-5.pdf
qpdf input.pdf --pages . 6-10 -- pages6-10.pdf

# Rotate pages
qpdf input.pdf output.pdf --rotate=+90:1  # Rotate page 1 by 90 degrees

# Remove password
qpdf --password=mypassword --decrypt encrypted.pdf decrypted.pdf
```

### pdftk (если доступен)
```bash
# Merge
pdftk file1.pdf file2.pdf cat output merged.pdf

# Split
pdftk input.pdf burst

# Rotate
pdftk input.pdf rotate 1east output rotated.pdf
```

## Типовые задачи

### Извлечение текста из отсканированных PDF
```python
# Requires: pip install pytesseract pdf2image
import pytesseract
from pdf2image import convert_from_path

# Convert PDF to images
images = convert_from_path('scanned.pdf')

# OCR each page
text = ""
for i, image in enumerate(images):
    text += f"Page {i+1}:\n"
    text += pytesseract.image_to_string(image)
    text += "\n\n"

print(text)
```

### Добавление водяного знака
```python
from pypdf import PdfReader, PdfWriter

# Create watermark (or load existing)
watermark = PdfReader("watermark.pdf").pages[0]

# Apply to all pages
reader = PdfReader("document.pdf")
writer = PdfWriter()

for page in reader.pages:
    page.merge_page(watermark)
    writer.add_page(page)

with open("watermarked.pdf", "wb") as output:
    writer.write(output)
```

### Извлечение изображений
```bash
# Using pdfimages (poppler-utils)
pdfimages -j input.pdf output_prefix

# This extracts all images as output_prefix-000.jpg, output_prefix-001.jpg, etc.
```

### Защита паролем
```python
from pypdf import PdfReader, PdfWriter

reader = PdfReader("input.pdf")
writer = PdfWriter()

for page in reader.pages:
    writer.add_page(page)

# Add password
writer.encrypt("userpassword", "ownerpassword")

with open("encrypted.pdf", "wb") as output:
    writer.write(output)
```

## Краткая справка

| Задача | Лучший инструмент | Команда/код |
|------|-----------|--------------|
| Объединить PDF | pypdf | `writer.add_page(page)` |
| Разделить PDF | pypdf | По странице на файл |
| Извлечь текст | pdfplumber | `page.extract_text()` |
| Извлечь таблицы | pdfplumber | `page.extract_tables()` |
| Создать PDF | reportlab | Canvas или Platypus |
| Объединение из командной строки | qpdf | `qpdf --empty --pages ...` |
| OCR отсканированных PDF | pytesseract | Сначала конвертировать в изображения |
| Заполнить PDF-формы | pdf-lib или pypdf (см. FORMS.md) | См. FORMS.md |

## Дальнейшие шаги

- Продвинутое использование pypdfium2 — см. REFERENCE.md
- JavaScript-библиотеки (pdf-lib) — см. REFERENCE.md
- Если нужно заполнить PDF-форму, следуй инструкциям в FORMS.md
- Руководства по устранению неполадок — см. REFERENCE.md
