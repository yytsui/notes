# 使用 AI 技術從 PDF 中提取結構化數據

在現代數據分析與處理中，從非結構化數據中提取有價值的信息是一項挑戰。尤其是當數據以自然語言文本存在於 PDF 文件中時，如何有效地提取並轉換成結構化數據（如 JSON 格式）以便於進一步分析，顯得尤為重要。本文將介紹如何使用 Python 和 AI 技術從一本出版於 1919 年的食譜書《Original Recipes of Good Things to Eat》中提取食譜數據。

## 背景

這本書中包含許多用自然語言撰寫的食譜段落，例如：

**Tomato Soup**
```
Boil 12 tomatoes until they are soft, run through a sieve and add a teaspoon of soda to a quart of pulp.
Put a tablespoon of butter in a sauce pan; when it melts add a teaspoon of flour. Add a pint of hot milk, salt,
cayenne pepper, and cracker crumbs. When it boils, add the tomatoes. Do not let it boil after the tomatoes
have been added. Serve at once. Mrs. Wilhelmina Albrecht.
```

我們的任務是從這些自然語言描述中提取出結構化數據，以 JSON 格式表示，如下所示：

```json
{
  "title": "Tomato Soup",
  "page": 11,
  "author": "Mrs. Wilhelmina Albrecht",
  "ingredients": [
    { "item": "tomatoes", "quantity": 12.0, "unit": null },
    { "item": "soda", "quantity": 1.0, "unit": "teaspoon" },
    { "item": "butter", "quantity": 1.0, "unit": "tablespoon" },
    { "item": "flour", "quantity": 1.0, "unit": "teaspoon" },
    { "item": "milk", "quantity": 1.0, "unit": "pint" },
    { "item": "salt", "quantity": null, "unit": null },
    { "item": "cayenne pepper", "quantity": null, "unit": null },
    { "item": "cracker crumbs", "quantity": null, "unit": null }
  ],
  "instructions": [
    "Boil 12 tomatoes until they are soft, run through a sieve and add a teaspoon of soda to a quart of pulp.",
    "Put a tablespoon of butter in a sauce pan; when it melts add a teaspoon of flour.",
    "Add a pint of hot milk, salt, cayenne pepper, and cracker crumbs.",
    "When it boils, add the tomatoes.",
    "Do not let it boil after the tomatoes have been added.",
    "Serve at once."
  ],
  "id": 1
}
```

此外，書中還包含一些廣告頁，我們需要忽略這些頁面，僅提取食譜內容。

---

## 使用的技術與工具

我們將使用以下工具來完成這個任務：

- **Python**：作為主要開發語言。
- **PyPDF2**：用於從 PDF 文件中提取文本。
- **OpenAI API**：利用 AI 模型從提取的文本中識別並結構化食譜。
- **Loguru**：用於日誌記錄，方便我們調試和監控程序。
- **dotenv**：用於管理 API 金鑰和其他環境變量。

---

## 解決方案概述

我們的 AI 解決方案包含以下步驟：

1. **PDF 文件讀取**：
   - 使用 PyPDF2 讀取 PDF 文件並提取每一頁的文本內容。
   
2. **構建 AI 提示**：
   - 根據提取的文本構建提示，要求 AI 從文本中提取食譜數據（包括標題、作者、食材、步驟等）。
   
3. **調用 OpenAI API**：
   - 將構建的提示發送給 OpenAI API，並將返回的數據解析為 JSON 格式。
   
4. **數據存儲**：
   - 將提取的食譜數據存儲到 JSON 文件中，並記錄未提取到食譜的頁面。

---

## 主要代碼解析

以下是我們的 Python 代碼：

```python
import json
import os
from string import Template
from typing import Dict

import PyPDF2
from dotenv import load_dotenv
from loguru import logger
from openai import OpenAI

from ai import ask_ai

FILE_PATH = "data/originalrecipeso00orde.pdf"
load_dotenv()
openai_client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

prompt_template = Template("""
    Extract all recipes from the following text starting from ---Page $page_number---. Each recipe should include:
    - Recipe name
    - Ingredients (with item, quantity, and unit)
    - Instructions (use the original text as much as possible)
    - Author
    - Page number

    Format each recipe in JSON format like this:

    {{
        "recipe": "Recipe Name",
        "page": {page_number},
        "author": "Author Name",
        "ingredients": [
            {{"item": "ingredient 1", "quantity": 2, "unit": "cups"}},
            {{"item": "ingredient 2", "quantity": 1, "unit": "tablespoon"}}
        ],
        "instructions": [
            "Step 1",
            "Step 2",
            "Step 3"
        ]
    }}
""")

SYSTEM_CONTENT = "You are a helpful information extraction assistant."

def read_pdf(pdf_path: str) -> Dict:
    """Extract text from PDF file."""
    page_text = {}
    with open(pdf_path, 'rb') as file:
        pdf_reader = PyPDF2.PdfReader(file)
        for i, page in enumerate(pdf_reader.pages):
            page_text[i+1] = page.extract_text()
    return page_text

def load_openai_json(response):
    """Parse the JSON response from OpenAI API."""
    try:
        return json.loads(response.split("```json")[1].split("```")[0])
    except Exception as e:
        logger.error(f"Error loading json: {e}")
        return None

def run():
    content = read_pdf(FILE_PATH)
    results = []
    for page_number, text in content.items():
        prompt = prompt_template.substitute(page_number=page_number, text=text)
        logger.info(f"Processing Page {page_number}")
        text_response = ask_ai(prompt, system_content=SYSTEM_CONTENT)
        response = load_openai_json(text_response)
        if response:
            results.append(response)
    
    with open("recipes.json", "w") as f:
        json.dump(results, f, indent=4)

if __name__ == "__main__":
    run()
```

---

## 結論

透過本案例，我們展示了如何結合 Python 和 AI 技術，從 PDF 文檔中提取結構化數據，並轉化為 JSON 格式。這不僅僅適用於食譜提取，還可以應用於其他類型的文檔數據提取，如報告、手冊和合同。

此解決方案展示了利用 AI 技術處理非結構化數據的潛力，為企業和研究機構提供了一種高效的數據處理方式。希望本文能幫助您理解 AI 在自然語言處理和數據提取方面的應用。
