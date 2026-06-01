# 文档智能识别输出字段说明

当 ocr_type = `DOC_PARSING` 时，任务成功后会返回一个 JSON 格式的结果文件（下载地址由 `results` 数组提供）。该 JSON 文件的结构遵循以下规范。

## 顶层结构

| 字段 | 类型 | 描述 |
|------|------|------|
| taskId  | string | 任务唯一标识 |
| documents | array | 文档列表（一个任务可包含多个文档，如 PDF 的每一页或压缩包中的多个文件） |


## documents  数组中每个元素的字段

| 字段 | 类型 | 描述 |
|------|------|------|
| documentId | string | 文档唯一标识 |
| fileName | string | 原始文件名 |
| datas | array | 文档页面数据列表（多页文档每页一个元素） |


### datas 数组中每个元素的字段

| 字段 | 类型 | 描述 |
|------|-----|------|
| rotate_angle | int | 页面旋转角度（0 表示未旋转） |
| height | int | 页面高度（像素） |
| width | int | 页面宽度（像素） |
| blocks | array | 该页上的版面块（block）列表，每个块代表一个独立的内容单元（文本、表格、图像、印章等） |
| md | object | 该页的 Markdown 格式内容及图片映射 |

### blocks 数组中每个元素的字段

| 字段 | 类型 | 描述 |
|------|------|------|
| id | int | 版面块唯一标识（注意：后处理可能整合部分块，id 为非连续递增） |
| bbox | array[int] | 边界框坐标 `[x1, y1, x2, y2]`（像素，左上角到右下角） |
| type | string | 一级分类，可选值：`text`, `chart`, `formula`, `footer`, `header`, `image`, `seal`, `table` |
| sub_type | string | 二级分类，详见下方“版面块二级分类体系” |
| sub_img | string | 切图的 Base64 编码（JPG 格式），仅当 `type` 为 `algorithm`、`chart`、`footer_image`、`header_image`、`image` 时存在 |
| lines | array | 文字识别内容（仅当 `type` 为 `text` 且 `sub_type` 为 `text`、`footer`、`footnote`、`header`、`formula_number` 时存在） |
| latex | string | 公式的 LaTeX 表示（仅当 `sub_type` 为 `display_formula` 时存在） |
| html | string | 表格的 HTML 格式（仅当 `type` 为 `table` 时存在） |
| cells | array | 表格单元格信息（仅当 `type` 为 `table` 时存在） |
| blocks | array | 印章集合（仅当 `type` 为 `seal` 时存在，内含多个印章块） |

### lines 数组中每个元素的字段

| 字段 | 类型 | 描述 |
|------|------|------|
| bbox | array[int] | 文本行边界框 `[x1, y1, x2, y2]` |
| polygon | array[array[int]] | 文本行多边形坐标（每个点 `[x, y]`） |
| text | string | 识别的文字内容 |
| score | float | 置信度（0~1） |

### cells 数组中每个元素的字段（表格单元格）

| 字段 | 类型 | 描述 |
|------|------|------|
| id | int | 单元格序号 |
| bbox | array[int] | 单元格边界框 `[x1, y1, x2, y2]` |
| polygon | array[array[int]] | 单元格多边形坐标 |
| rows | array[int] | 所在行的逻辑位置（从 0 开始） |
| cols | array[int] | 所在列的逻辑位置（从 0 开始） |

### 印章（seal）专用字段

当 `type` 为 `seal` 时，`blocks` 数组中的每个元素代表一个印章，结构如下：

| 字段 | 类型 | 描述 |
|------|------|------|
| class_id | int | 印章类型 ID，映射：`0: 'circle_stamp'`, `1: 'oval_stamp'`, `2: 'rectangle_stamp'`, `3: 'other'`, `4: 'personal_stamp'`, `5: 'triangle_stamp'`, `6: 'rhombus_stamp'` |
| class_name | string | 印章类型名称 |
| bbox | array[int] | 印章边界框 `[x1, y1, x2, y2]` |
| score | float | 置信度 |
| is_rotated | bool | 是否旋转 |
| id | int | 印章序号 |
| color_type | string | 颜色分类：`红色`、`蓝色`、`黑色`、`其他` |
| sub_img | string | 印章切图 Base64（JPG） |
| lines | array | 印章中的文本行（结构同 `lines`） |

## md 对象字段

| 字段 | 类型 | 描述 |
|------|------|------|
| markdown_content | string | 整页的 Markdown 内容，其中图片和印章切图会被映射为 `images/image_id_sid.jpg` 形式的占位符 |
| images | array | 图片映射信息列表，每个元素描述一个嵌入图片 |

### images 数组中每个元素的字段

| 字段 | 类型 | 描述 |
|------|------|------|
| id | int | 图片序号 |
| block_index | int | 在 `blocks` 数组中的索引 |
| seal_block_index | int | 若为印章，表示在印章 blocks 数组中的索引 |
| format | string | 图片格式，固定为 `JPG` |
| path | string | 映射路径，固定为 `images/` |
| name | string | 文件名，如 `image_1.jpg` 或 `image_1_seal_0.jpg` |
| type | string | 一级分类（与 block 的 type 相同） |
| sub_type | string | 二级分类（与 block 的 sub_type 相同） |

## 版面块二级分类体系

### text 类型

| sub_type | 说明 |
|----------|------|
| abstract | 论文摘要 |
| content | 目录内容（仅在大目录块中出现） |
| doc_title | 文档主标题 |
| text | 普通文本 |
| vertical_text | 竖排文本 |
| paragraph_title | 段落标题 |
| reference | 参考文献列表外框 |
| reference_content | 参考文献条目 |
| aside_text | 页边注文本（位于页面边缘的补充信息） |
| figure_title | 图片/图表/表格的标题（caption） |
| vision_footnote | 图片/图表/表格的脚注 |
| number | 页码 |

### chart 类型

| sub_type | 说明 |
|----------|------|
| chart | 图表（柱状图、折线图、饼图等数据可视化元素） |

### formula 类型

| sub_type | 说明 |
|----------|------|
| display_formula | 独立展示的公式（整行或多行） |
| formula_number | 公式编号 |
| inline_formula | 行内公式 |

### footer 类型

| sub_type | 说明 |
|----------|------|
| footer | 页脚文本 |
| footer_image | 页脚图片 |
| footnote | 脚注（页面底部） |

### header 类型

| sub_type | 说明 |
|----------|------|
| header | 页眉文本 |
| header_image | 页眉图片 |

### image 类型

| sub_type | 说明 |
|----------|------|
| algorithm | 算法图 |
| image | 普通图片 |

### seal 类型

| sub_type | 说明 |
|----------|------|
| seal | 印章 |

### table 类型

| sub_type | 说明 |
|----------|------|
| table | 表格 |



## 示例输出片段

```json
{
  "taskId": "task-20250506-7643",
  "documents": [
    {
      "documentId": "task-20250506-7643_0",
      "fileName": "招标公告.pdf",
      "datas": [
        {
          "rotate_angle": 0,
          "height": 3507,
          "width": 2478,
          "blocks": [
            {
              "id": 1,
              "bbox": [446, 329, 2084, 489],
              "type": "text",
              "sub_type": "text",
              "lines": [
                {
                  "bbox": [455, 335, 2072, 382],
                  "polygon": [[455,335], [2072,333], [2072,382], [455,385]],
                  "text": "中国工商银行股份有限公司 2025 年度海光芯片服务器采购项目公开招标公告",
                  "score": 0.996
                }
              ]
            },
            {
              "id": 24,
              "type": "seal",
              "sub_type": "seal",
              "blocks": [
                {
                  "class_id": 0,
                  "class_name": "circle_stamp",
                  "bbox": [1553, 519, 2063, 1019],
                  "score": 0.967,
                  "color_type": "红色",
                  "lines": [{"text": "中信国际招标有限公司", "score": 1.0, "polygon": [...]}],
                  "sub_img": "/9j/4AAQSkZJRgABAQAAAQABAAD/..."
                }
              ]
            }
          ],
          "md": {
            "images": [
              {
                "id": 0,
                "block_index": 2,
                "seal_block_index": 0,
                "format": "JPG",
                "path": "images",
                "name": "image_1_seal_0.jpg",
                "type": "seal",
                "sub_type": "seal"
              }
            ],
            "markdown_content": "尚的。(签名)招标人或其招标代理机构主要负责人 (项目负责人)\n\n招标人或其招标代理机构:\n\n![](images/image_1_seal_0.jpg)"
          }
        }
      ]
    }
  ]
}
 ```
## 注意事项
- `所有坐标以像素为单位，原点为页面左上角，x 轴向右，y 轴向下。`

- `置信度 score 取值范围 0~1，越接近 1 表示识别结果越可信。`

- `sub_img 字段为 Base64 编码的 JPG 图片，可直接解码保存。`

- `Markdown 中的图片映射路径为 images/xxx.jpg，需结合 md.images 数组中的信息进行替换或提取。`

- `印章识别结果中的 lines 数组可能包含多个文本行，每个文本行包含多边形坐标和文本内容。`