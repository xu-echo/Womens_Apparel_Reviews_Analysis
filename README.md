# 女性服装评论数据分析项目

## 项目概述
本项目对某在线零售商的女装产品评论进行多维度分析，使用`numpy`/`pandas`/`matplotlib`技术栈，主要研究内容包括：
- 客户年龄分布特征
- 基于Minqing Hu and Bing Liu. "Mining and summarizing customer reviews."的评论情感分析 
- 评分与情感关联性研究
- 产品推荐率与评分关联性研究
- 改进情感分析函数（考虑否定词）

## 数据集
包含三个文件：
- `WomensApparelReviews.csv` (23,000+条评论)
  - 字段说明：
    | 列名               | 类型     | 描述                          |
    |--------------------|----------|-----------------------------|
    | Product ID         | 整型     | 商品唯一标识                  |
    | Age                | 整型     | 评论者年龄                    |
    | Title              | 文本     | 评论标题（部分为空）           |
    | Review Text        | 文本     | 评论文本内容                  |
    | Rating             | 1-5分    | 用户评分（1代表最差，5代表最好）  |
    | Is it Recommended? | 0/1      | 0：不推荐；1：推荐             |
    | Department         | 类别型   | 商品类别（连衣裙、上衣等）     |

- `positive-words.txt`: 2006个积极情感词汇
- `negative-words.txt`: 4783个消极情感词汇

## 分析方法
1. 情感评分模型
- 情感得分 = 积极单词数 - 消极单词数
-  阈值划分：
   - 情感得分 > 0 → 积极
   - 情感得分 < 0 → 消极
   - 情感得分 = 0 → 中性
```python
# 情感分析函数
def analyze_sentiment(text):
    words = preprocess_text(text)
    positive_count = sum(1 for word in words if word in positive_words)
    negative_count = sum(1 for word in words if word in negative_words)
    # 计算情感得分（积极单词数 - 消极单词数）
    sentiment_score = positive_count - negative_count
    return sentiment_score

# 情感分类函数
def classify_sentiment(score):
    if score > 0:
        return 'Positive'
    elif score < 0:
        return 'Negative'
    else:
        return 'Neutral'
```
2. 核心分析维度
- 年龄分布分析
- 情感-评分/产品推荐率-评分相关性分析（皮尔逊系数）
- 否定句式处理（如评论文本"not good"的语义反转）
```python
# 定义否定词列表
negation_words = {'not', 'no', 'never', 'none', 'neither', 'nor', 'cannot'}

# 情感分析函数（改进版，考虑否定词）
def analyze_sentiment2(text):
    sentences = re.split(r'[.!?]', text)  # 分句处理
    positive_count = 0
    negative_count = 0
    
    for sentence in sentences:
        words = preprocess_text(sentence)
        negation_flag = False  # 当前句子内的否定状态
        
        for word in words:
            if word in negation_words:
                negation_flag = not negation_flag  # 遇到否定词时切换状态
            else:
                # 判断词汇极性并应用否定逻辑
                if word in positive_words:
                    adjusted_polarity = -1 if negation_flag else 1
                    if adjusted_polarity == 1:
                        positive_count += 1
                    else:
                        negative_count += 1
                elif word in negative_words:
                    adjusted_polarity = 1 if negation_flag else -1
                    if adjusted_polarity == 1:
                        positive_count += 1
                    else:
                        negative_count += 1
    
    sentiment_score = positive_count - negative_count
    return sentiment_score
```

## 关键发现
1. 年龄分布特征
- 大部分顾客年龄集中在 34 到 52 岁（占比50%）
- 将年龄分段（如 18-30 岁、31-45 岁、46-60 岁、60+ 岁），各年龄段最喜爱的商品id是1078，购买频率最高的是上衣（Tops）
2. 相关性分析
- 标题情感得分和评论文本情感得分正相关性较弱
- 评论文本情感得分和评分正相关性较弱
- 高评分通常对应高推荐率
3. 消费者负面评价主要聚焦四大产品维度：
- 尺码标注系统与消费者预期存在显著偏差（偏大或偏小）
- 面料质感未达消费者心理预期（主要表现为易皱、透气性差等问题）
- 版型设计与人体工程学原理存在偏差（肩线位置不当、腰围比例失调等）
- 工艺质量控制缺陷（车缝走线不齐整、线头处理粗糙等）
