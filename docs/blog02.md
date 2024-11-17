# 使用 OpenAI 的 JSON Schema 進行結構化數據提取：升級版 AI 驅動解決方案

在前一篇文章中，我們介紹了如何利用 OpenAI API 從 PDF 文件中提取結構化的食譜數據。在本篇文章中，我們將進一步深入探討如何使用 **OpenAI 的 JSON Schema** 技術，以更精確和結構化的方式提取數據。這次的改進將結合 Python 的 `pydantic` 數據模型，並使用 OpenAI 的 `response_format` 功能來確保返回的數據符合我們預期的結構。

---

## 使用 JSON Schema 的優勢

透過 OpenAI 的 JSON Schema 功能，我們可以確保 AI 模型生成的輸出完全符合預期的數據格式，從而大幅降低數據清理和後期處理的工作量。在自然語言文本處理任務中，使用預定義的 Schema 可以避免格式錯誤，確保數據的一致性和準確性。

### 預期輸出格式

我們希望從 PDF 文件中提取的數據結構如下：

```json
{
  "id": 1,
  "title": "Tomato Soup",
  "page": 11,
  "author": "Mrs. Wilhelmina Albrecht",
  "ingredients": [
    { "item": "tomatoes", "quantity": 12, "unit": null },
    { "item": "soda", "quantity": 1, "unit": "teaspoon" },
    { "item": "butter", "quantity": 1, "unit": "tablespoon" },
    { "item": "flour", "quantity": 1, "unit": "teaspoon" }
  ],
  "instructions": [
    "Boil 12 tomatoes until they are soft.",
    "Add a teaspoon of soda to a quart of pulp.",
    "Put a tablespoon of butter in a sauce pan and add a teaspoon of flour.",
    "Add hot milk, salt, cayenne pepper, and cracker crumbs.",
    "Serve at once."
  ]
}
```

---

## Python 代碼解析

### 1. 數據模型定義

首先，我們使用 `pydantic` 來定義我們的數據結構：

```python
from typing import List, Optional, Union
from pydantic import BaseModel

class Ingredient(BaseModel):
    item: Optional[str] = None
    quantity: Optional[Union[float, str]] = None
    unit: Optional[str] = None

class Recipe(BaseModel):
    id: int
    title: Optional[str] = None
    page: Optional[int] = None
    author: Optional[str] = None
    ingredients: Optional[List[Ingredient]] = None
    instructions: Optional[List[str]] = None
```

這些模型幫助我們確保所有提取到的數據符合定義的格式，避免因輸出不一致而導致的問題。

### 2. 使用 OpenAI API 進行數據提取

我們利用 OpenAI 的 `response_format` 功能來確保返回的數據符合我們的 JSON Schema：

```python
def find_structured_info_with_ai(prompt, system_content, response_format):
    completion = openai_client.beta.chat.completions.parse(
        model="gpt-4o-2024-08-06",
        messages=[
            {"role": "system", "content": system_content},
            {"role": "user", "content": prompt}
        ],
        response_format=response_format,
    )
    message = completion.choices[0].message
    if message.parsed:
        return message.parsed
    else:
        return message.refusal
```

### 3. 查詢提示生成

我們為每一頁生成提示，以幫助 AI 從文本中準確識別食譜內容：

```python
def generate_prompt(page_number, text):
    return (f"Extract all recipes from the following text starting from ---Page {page_number}--- "
            f"Each recipe should include: - title - Ingredients (with item, quantity, and unit) - "
            f"Instructions (use the original text as much as possible) - Author - Page number\n"
            f"There could be multiple recipes in a page. "
            f"---Page {page_number}---\n{text}")
```

### 4. 主程序運行邏輯

以下是主程序，從 PDF 文件中提取文本，並使用 AI 進行結構化數據提取：

```python
def main():
    content = read_pdf(FILE_PATH)
    recipes = []
    no_recipes_warnings = []
    
    for page_number, text in content.items():
        prompt = generate_prompt(page_number, text)
        response = find_structured_info_with_ai(prompt, SYSTEM_CONTENT, CookBook)
        
        if isinstance(response, CookBook) and response.recipes:
            recipes.extend(response.recipes)
            logger.info(f"Found {len(response.recipes)} recipes on page {page_number}")
        else:
            no_recipes_warnings.append({"page_number": page_number, "warning": response})
    
    # 保存提取的食譜數據
    recipes_dict = [recipe.model_dump() for recipe in recipes]
    dump_json(recipes_dict, STRUCTURED_RECIPES_FILEPATH)
    dump_json(no_recipes_warnings, STRUCTURED_NO_RECIPES_WARNINGS_FILEPATH)

if __name__ == "__main__":
    main()
```

---

## 完整的 Python 代碼

以下是完整的代碼，您可以直接複製並運行：

```python
import os
from dotenv import load_dotenv
from icecream import ic
from loguru import logger
from openai import OpenAI
from pdf_text import read_pdf
from settings import FILE_PATH, SYSTEM_CONTENT, STRUCTURED_RECIPES_FILEPATH, STRUCTURED_NO_RECIPES_WARNINGS_FILEPATH
from utils import dump_json
from typing import List, Optional, Union
from pydantic import BaseModel

class Ingredient(BaseModel):
    item: Optional[str] = None
    quantity: Optional[Union[float, str]] = None
    unit: Optional[str] = None

class Recipe(BaseModel):
    id: int
    title: Optional[str] = None
    page: Optional[int] = None
    author: Optional[str] = None
    ingredients: Optional[List[Ingredient]] = None
    instructions: Optional[List[str]] = None

class CookBook(BaseModel):
    recipes: List[Recipe]

load_dotenv()
openai_client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

def main():
    content = read_pdf(FILE_PATH)
    recipes = []
    
    for page_number, text in content.items():
        prompt = generate_prompt(page_number, text)
        response = find_structured_info_with_ai(prompt, SYSTEM_CONTENT, CookBook)
        if isinstance(response, CookBook) and response.recipes:
            recipes.extend(response.recipes)
    
    recipes_dict = [recipe.model_dump() for recipe in recipes]
    dump_json(recipes_dict, STRUCTURED_RECIPES_FILEPATH)

if __name__ == "__main__":
    main()
```

---

## 結論

透過本次升級，我們展示了如何結合 OpenAI 的 JSON Schema 和 Python 的 `pydantic` 庫，以更精確的方式從 PDF 中提取結構化數據。這不僅提高了數據提取的準確性，還減少了後期數據清理的工作量。

未來，我們可以將這種方法應用於其他類型的文檔數據提取，如合同、技術手冊和報告等，為企業和數據科學家提供更高效的數據處理工具。

希望本文能幫助您更好地理解 OpenAI 的 JSON Schema 功能及其應用場景。
