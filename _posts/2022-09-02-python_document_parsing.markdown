---
layout: custom
title: python으로 문서파일에서 텍스트 추출하기
date: 2022-09-02 10:46:00 +0900
last_modified_at: 2022-09-02 10:46:00 +0900
category: python
tags: ["python"]
published: false

---
> 문서(hwp, pdf, pptx, docx)에서 텍스트 추출하기

## 1. hwp 파일에서 텍스트 추출하기

- install
```bash
$ pip install olefile
```

- 추출 코드
    - 참고: https://blog.naver.com/PostView.nhn?blogId=shino1025&logNo=222131312454&categoryNo=0&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView)
    - PrvText를 이용한 추출은 미리보기 영역이므로 일정 길이를 초과하면 잘림
    - BodyText/Section에서 추출해야함
```python
import olefile
import zlib
import struct

def get_hwp_text(filename):
    f = olefile.OleFileIO(filename)
    dirs = f.listdir()

    # HWP 파일 검증
    if ["FileHeader"] not in dirs or \
       ["\x05HwpSummaryInformation"] not in dirs:
        raise Exception("Not Valid HWP.")

    # 문서 포맷 압축 여부 확인
    header = f.openstream("FileHeader")
    header_data = header.read()
    is_compressed = (header_data[36] & 1) == 1

    # Body Sections 불러오기
    nums = []
    for d in dirs:
        if d[0] == "BodyText":
            nums.append(int(d[1][len("Section"):]))
    sections = ["BodyText/Section"+str(x) for x in sorted(nums)]

    # 전체 text 추출
    text = ""
    for section in sections:
        bodytext = f.openstream(section)
        data = bodytext.read()
        if is_compressed:
            unpacked_data = zlib.decompress(data, -15)
        else:
            unpacked_data = data
    
        # 각 Section 내 text 추출    
        section_text = ""
        i = 0
        size = len(unpacked_data)
        while i < size:
            header = struct.unpack_from("<I", unpacked_data, i)[0]
            rec_type = header & 0x3ff
            rec_len = (header >> 20) & 0xfff

            if rec_type in [67]:
                rec_data = unpacked_data[i+4:i+4+rec_len]
                section_text += rec_data.decode('utf-16')
                section_text += "\n"

            i += 4 + rec_len

        text += section_text
        text += "\n"

    return text
```

## 2. pdf 파일에서 텍스트 추출하기

- install
```bash
$ pip install pdfminer
```

- 추출 코드
```python
from io import StringIO
from pdfminer.converter import TextConverter
from pdfminer.layout import LAParams
from pdfminer.pdfdocument import PDFDocument
from pdfminer.pdfinterp import PDFResourceManager, PDFPageInterpreter
from pdfminer.pdfpage import PDFPage
from pdfminer.pdfparser import PDFParser

def get_pdf_to_str(path):
    f = open(path, 'rb')
    
    output_string = StringIO()
    parser = PDFParser(f)
    doc = PDFDocument(parser)

    rsrcmgr = PDFResourceManager()
    device = TextConverter(rsrcmgr, output_string, laparams=LAParams())
    interpreter = PDFPageInterpreter(rsrcmgr, device)
    for page in PDFPage.create_pages(doc):
        interpreter.process_page(page)

    return str(output_string.getvalue())
```

## 3. docx 파일에서 텍스트 추출하기

- install
```bash
$ pip install python-docx
```

- 추출 코드
```python
import docx

def get_docx_to_str(path):
    doc = docx.Document(path)
    
    results = []
    results += [para.text for para in doc.paragraphs]
    results += [para.text for table in doc.tables for row in table.rows for cell in row.cells for para in cell.paragraphs]
    results += [content.text for content in doc.paragraphs if content.style.name == 'Heading 1' or content.style.name == 'Heading 2' or content.style.name == 'Heading 3']

    return '\n'.join(results)
```

## 4. pptx 파일에서 텍스트 추출하기

- install
```bash
$ pip install python-pptx
```

- 추출 코드
```python
from pptx import Presentation

def get_pptx_to_str(path):
    f = Presentation(path)
    
    result = []
    # 슬라이드 순회하며 텍스트 파싱
    for slide in f.slides:
        for shape in slide.shapes:

            # 슬라이드 text 존재 확인
            if not shape.has_text_frame:
                continue
            for paragraph in shape.text_frame.paragraphs:
                if paragraph.text == '':
                    continue
                result.append(paragraph.text)

    return '\n'.join(result)
```

