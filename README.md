# Document Parse OCR Skill

基于 Scnet OCR API 的文档智能解析技能，支持异步识别 PDF、图片等文件中的文本、表格、标题、公式等结构，并输出结构化 JSON 和 Markdown 内容。

## 功能
- 异步提交，适合大批量文档
- 自动识别版面元素（段落、标题、表格、图表、页眉页脚等）
- 返回 Markdown 格式全文
- 结果文件临时下载（有效期12小时）

## 前置要求
- Python 3.6+
- requests 库 (`pip install requests`)
- Scnet API Key（[申请地址](https://www.scnet.cn)）

## 配置
在 `config/.env` 中设置：
```ini
SCNET_API_KEY=你的密钥
SCNET_API_BASE=https://api.scnet.cn/api/llm/v1
```

## 使用
```bash
python scripts/main.py [ocrType] <fileUrl>
# 示例
python scripts/main.py DOC_PARSING "https://example.com/doc.pdf"
```

## 输出
标准输出为识别结果的 JSON 对象，错误信息输出到 stderr。

## 数据隐私与安全
本技能仅将文件 URL 转发至 Scnet API，不存储任何文件内容。

结果由 Scnet 生成，下载地址有效期 12 小时，请及时保存。

用户须确保拥有文档处理权限，并遵守相关法律法规。

## 免责声明
本技能按“现状”提供，不承担因使用本技能产生的任何直接或间接责任。用户应自行判断并承担风险。

## 使用条款
1. 用户须保证所提交文档内容的合法性，不得含有违反法律法规或侵犯他人权益的信息。
2. 用户知悉并同意，文档内容将通过本技能发送至 Scnet 第三方 API 进行处理，相关数据按 Scnet 的隐私政策处理。
3. 本技能开发者不对识别结果的准确性、完整性或可用性作任何保证，也不对用户因使用结果而遭受的任何损失承担责任。
4. 若您不同意以上条款，请勿使用本技能

## 许可证
MIT

