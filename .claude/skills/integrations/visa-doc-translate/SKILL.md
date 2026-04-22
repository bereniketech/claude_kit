---
name: visa-doc-translate
description: OCR, translate, and generate bilingual PDFs from visa application document images.
---

# Visa Document Translator

Translate visa application document images to English and produce a bilingual PDF — original image on page 1, professional English translation on page 2.

---

## 1. Workflow

When given an image file path, execute all steps automatically without asking for confirmation:

1. **Convert** HEIC to PNG if needed: `sips -s format png <input> --out <output>`
2. **Rotate** based on EXIF orientation (orientation 6 → rotate 90° CCW; apply 180° if upside-down)
3. **OCR** the document (try methods in order: macOS Vision, EasyOCR, Tesseract)
4. **Translate** all text to professional English, preserving document structure
5. **Generate PDF** with PIL + reportlab: page 1 = rotated original image, page 2 = formatted translation
6. **Output** `<original_filename>_Translated.pdf` in the same directory

**Rule:** Complete the entire pipeline and report the final PDF location. Do not stop at intermediate steps.

---

## 2. OCR Methods (tried in order)

**macOS Vision** (preferred on macOS):
```python
import Vision
from Foundation import NSURL
```
Install: `pip install pyobjc-framework-Vision pyobjc-framework-Quartz`

**EasyOCR** (cross-platform, no Tesseract required):
```bash
pip install easyocr
```

**Tesseract** (fallback):
```bash
brew install tesseract tesseract-lang
pip install pytesseract
```

---

## 3. Translation Rules

- Translate all text content to professional English
- Maintain original document structure and format
- Use professional terminology appropriate for visa applications
- Keep proper names in original language with English in parentheses
- For Chinese names, use pinyin format (e.g., WU Zhengye)
- Preserve all numbers, dates, and amounts exactly

---

## 4. PDF Generation

```bash
pip install pillow reportlab
```

Structure:
- **Page 1:** Rotated original image, centered and scaled to fit A4
- **Page 2:** English translation — title centered and bold, content left-aligned with appropriate spacing
- **Footer note:** "This is a certified English translation of the original document"

---

## 5. Supported Document Types

- Bank deposit certificates (存款证明)
- Income and employment certificates (收入证明 / 在职证明)
- Retirement certificates (退休证明)
- Property certificates (房产证明)
- Business licenses (营业执照)
- ID cards, passports, and other official documents

---

## 6. Usage

```bash
/visa-doc-translate RetirementCertificate.PNG
/visa-doc-translate BankStatement.HEIC
/visa-doc-translate EmploymentLetter.jpg
```

Suitable for visa applications to Australia, USA, Canada, UK, and other countries requiring translated documents.
