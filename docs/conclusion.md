# 結論

這一系列的文章展示了如何用AI從自然語言中自動提取結構化訊息,
並且使用OpenAI 的 DALL-E 3模型根據提取到的文字來生成圖片,
達到了將一段文字轉換成表列加圖案使可讀性大大的增加

但仍有一些可改進之處

1. 步驟圖的風格不一致:這可能可以藉由同一份食譜生成圖的prompt指定風格來改進
2. 正確性的驗證:文字部分可以藉由不同LLM (ex. OpenAI vs Claude AI)的提取結果交叉比對來做為驗證
3. 中文化:[資料翻譯部分已用AI完成](https://github.com/yytsui/AIInfoExtractionExample/blob/master/recipe_extractor/translation.py)
