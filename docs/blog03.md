# 使用 AI 生成食譜主圖與步驟圖片：提升食譜呈現的視覺效果

在前幾篇文章中，我們介紹了如何使用 AI 從 PDF 中提取結構化數據，並將食譜轉換為 JSON 格式。然而，僅僅有結構化的數據還不夠，如果能為食譜生成精美的圖片，將使其更具吸引力和實用性。本文將介紹如何使用 OpenAI 的 DALL-E 3 模型來生成食譜主圖和分步驟圖片，以提升食譜的視覺效果。

---

## 為什麼要生成食譜圖片？

在食譜展示中，圖片是吸引讀者的關鍵。研究表明，具有精美圖片的食譜更能引起讀者的興趣，並提高嘗試製作的意願。傳統上，為每道菜拍攝專業照片需要時間和費用，而 AI 生成的圖片提供了一種高效且經濟的替代方案。

### 本文將涵蓋以下內容

1. **如何生成食譜主圖**：根據食譜的標題和食材生成主圖。
2. **如何生成步驟圖片**：為每一步驟生成對應的圖片，幫助讀者更好地理解製作流程。
3. **如何將生成的圖片存儲到本地**：將生成的圖片下載並保存到本地，以便於網站或應用程序展示。

---

## 使用 OpenAI API 生成圖片

### Python 程式碼概述

以下程式碼展示了如何使用 OpenAI 的 DALL-E 3 模型來生成食譜主圖和步驟圖片。

### 1. 初始化 OpenAI 客戶端

首先，我們需要從環境變量中讀取 OpenAI 的 API 密鑰並初始化客戶端：

```python
from openai import OpenAI
from dotenv import load_dotenv
import os
from loguru import logger
import urllib.request
from pathlib import Path

load_dotenv()
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
```

### 2. 定義圖片生成函數

我們使用 `generate_image` 函數來生成圖片。該函數接受一個描述性文字（prompt），並調用 OpenAI 的 DALL-E 3 模型來生成圖片：

```python
def generate_image(prompt):
    logger.info(f"Generating image with prompt: {prompt}")
    response = client.images.generate(
        model="dall-e-3",
        prompt=prompt,
        size="1024x1024",
        quality="standard",
        n=1,
    )
    image_url = response.data[0].url
    return image_url
```

### 3. 生成食譜主圖

使用食譜的標題和食材列表來生成一張主圖，以展示完成的料理成品：

```python
def generate_main_image_from_recipe(recipe):
    prompt = f"A {recipe.title} which was made from ingredients: {recipe.ingredient_items_text}."
    image_url = generate_image(prompt)
    return image_url
```

### 4. 為每個步驟生成圖片

針對食譜中的每一個步驟生成對應的圖片，幫助讀者更直觀地理解製作過程：

```python
def generate_image_from_step(step):
    image_url = generate_image(step)
    return image_url
```

### 5. 下載並存儲圖片

為了方便展示和存檔，我們需要將生成的圖片下載並保存到本地：

```python
def persist_image(image_url, filepath):
    logger.info(f"Downloading image from {image_url} to {filepath}")
    Path(filepath).parent.mkdir(parents=True, exist_ok=True)
    urllib.request.urlretrieve(image_url, filepath)
```

---

## 綜合應用：為食譜生成主圖與步驟圖片

我們可以使用以下函數自動生成食譜的主圖和步驟圖片，並保存到指定目錄：

```python
def generate_recipe_images_and_persist(recipe):
    logger.info(f"Generating images for recipe: {recipe.title} (ID: {recipe.id})")
    
    # 生成主圖
    main_image_url = generate_main_image_from_recipe(recipe)
    main_image_filepath = os.path.join(IMAGE_STORAGE_PATH, recipe.main_image_filename)
    persist_image(main_image_url, main_image_filepath)
    
    # 生成步驟圖片
    for i, step in enumerate(recipe.instructions):
        step_image_url = generate_image_from_step(step)
        step_image_filepath = os.path.join(IMAGE_STORAGE_PATH, recipe.get_step_image_filename(i))
        persist_image(step_image_url, step_image_filepath)
```

---

## 主程序執行流程

最後，我們整合以上功能，批量生成食譜圖片：

```python
def main():
    from load_recipes import fetch_recipe_samples
    recipes = fetch_recipe_samples(5)
    for recipe in recipes:
        generate_recipe_images_and_persist(recipe)

if __name__ == "__main__":
    main()
```

### 說明

1. **`fetch_recipe_samples`**：用於加載示例食譜數據。
2. **`generate_recipe_images_and_persist`**：為每道食譜生成主圖和步驟圖片並保存。

---

## 結論

透過使用 OpenAI 的 DALL-E 3 模型，我們可以自動為食譜生成高質量的圖片，無論是成品展示還是步驟說明。這不僅節省了人工拍攝的時間和成本，還能提升食譜的可讀性和吸引力。

這種技術不僅限於食譜應用，還可以廣泛應用於產品展示、教學材料等多種場景。未來，我們可以進一步優化生成的圖片質量，並探索更多的 AI 驅動應用場景。

希望這篇文章能幫助您理解如何使用 AI 技術來提升食譜和其他內容的視覺呈現效果！
