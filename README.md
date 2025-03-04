# Table Extraction using Unstructured and GenAI

## Overview
This project focuses on extracting tables from PDFs and images using the **Unstructured** library and Generative AI models like **Llama** and **Gemini**.

## Installation
To set up the environment, install the required dependencies:

```bash
pip install unstructured
pip install pi-heif
pip install unstructured-inference
pip install pdf2image
pip install "python-doctr[torch,viz,html,contrib]"
pip install onnx==1.16.1
sudo apt update -q
sudo apt install poppler-utils -q
pip install unstructured.pytesseract
sudo apt install tesseract-ocr
```

## Table Extraction from PDFs

```python
from unstructured.partition.pdf import partition_pdf

fname = "/content/Attention all you need.pdf"

# Extract elements from the PDF
elements = partition_pdf(filename=fname,
                         infer_table_structure=True,
                         strategy='hi_res',
           )

# Filter extracted tables
tables = [el for el in elements if el.category == "Table"]

print(tables[0].text)
print(tables[0].metadata.text_as_html)
```

## Converting Table to Pandas DataFrame

```python
from bs4 import BeautifulSoup
import pandas as pd

html_content = tables[3].metadata.text_as_html
soup = BeautifulSoup(html_content, "html.parser")

table = soup.find("table")

headers = [th.text.strip() for th in table.find_all("th")]

rows = []
for tr in table.find_all("tr")[1:]:  # Skip header row
    cells = [td.text.strip() for td in tr.find_all("td")]
    rows.append(cells)

df = pd.DataFrame(rows, columns=headers)
```

## Table Extraction using Generative AI

```python
import base64
import PIL.Image

def preprocess_image(image_path):
    with open(image_path, 'rb') as img:
        encoded_string = base64.b64encode(img.read())
    return encoded_string.decode('utf-8')

def create_json_from_response_from_image(img_path):
    img = PIL.Image.open(img_path)
    prompt = """
Your task is to **extract tables** from the given image with **precision** and return them in a structured JSON format with proper keys and values.
    """
    response = model.generate_content([prompt, img])
    return response.text.replace("```json","").replace("```","")
```

