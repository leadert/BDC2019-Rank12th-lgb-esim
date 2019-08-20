# 2019-lightgbm-esim-pytorch
2019中国高校计算机大赛--大数据挑战赛Rank 12（福建大三本）-Solutions

比赛链接：[2019中国高校计算机大赛--大数据挑战赛](https://www.kesci.com/home/competition/5cc51043f71088002c5b8840  "悬停显示文字")
## 正式赛题--文本点击率预估（5月26日开赛）
搜索中一个重要的任务是根据query和title预测query下doc点击率，本次大赛参赛队伍需要根据脱敏后的数据预测指定doc的点击率，结果按照指定的评价指标使用在线评测数据进行评测和排名，得分最优者获胜。

### 比赛数据
#### sample 样本格式
| 列名      | 类型     | 示例     |
| ---------- | :-----------:  | :-----------: |
| query_id     | int，一个query的唯一标识     | 1     |
| query     | 字符string，term空格分割     | "字节跳动"     |
| title     | 字符string，term空格分割     | "字节跳动 - 百科"     |
| label     | int，取值{0, 1}，有点击为1，无点击为0   | 1     |

#### training 样本
脱敏后的query和网页文本数据，并已经分词为term并脱敏，term之间空格分割。

| 列名      | 类型     | 示例     |
| ---------- | :-----------:  | :-----------: |
| query_id     | int     | 3     |
| query     | hash string，term空格分割    | 1 9 117     |
| query_title_id    | title在query下的唯一标识    | 2    |
| title     | hash string，term空格分割    | 3 9 120     |
| label     | int，取值{0, 1}   | 0     |

### 评估标准
qAUC, qAUC为不同query下的AUC的平均值，计算如下：

<a href="https://www.codecogs.com/eqnedit.php?latex=qAUC&space;=&space;\frac{\sum&space;AUC_{i}}{query\_num}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?qAUC&space;=&space;\frac{\sum&space;AUC_{i}}{query\_num}" title="qAUC = \frac{\sum AUC_{i}}{query\_num}" /></a>

其中AUCi为同一个query_id下的AUC（Area Under Curve）。

### 最终成绩
初赛Rank 34(lgb)，复赛A榜Rank 17，B榜Rank 12，score=0.63834。

（第一次参加NLP比赛，能拿到这个名次已经很开心了。希望以后能再接再厉，更进一步）



## 项目说明
训练lgb所采用的数据为10亿训练集中的后5kw，训练ESIM所采用的数据为10亿训练集中随机抽样的8kw。

#### lgb_word2vec.ipynb
训练用于lgb的word2vec词向量，5kw训练集与1亿最终测试集作为数据集，min_count=1, size=150，未对句子进行去重。

#### nn_word2vec.ipynb
训练用于ESIM的word2vec词向量，8kw训练集与1亿最终测试集作为数据集，min_count=4, size=150，未对句子进行去重。

#### nn_fasttext.ipynb
训练用于ESIM的fasttext词向量，8kw训练集与1亿最终测试集作为数据集，min_count=4, size=150，未对句子进行去重。

#### lgb_model.ipynb
训练lgb，A榜0.595266采用了40维向量，线下loss为0.435457左右（后两位不记得了）。包括统计特征，频率特征，nlp特征，jaccard、levenshtein、fuzzywuzzy距离度量特征等，特征都比较普通。单进程构造特征特别慢，在程序中使用多进程处理。nlp特征对句中词向量加权平均得到句向量，计算[scipy distance computations](https://docs.scipy.org/doc/scipy/reference/spatial.distance.html  "悬停显示文字")。lightgbm使用的sklearn中的接口，num_leaves=127。最终成绩中的lgb采用了约55维特征，线下的loss为0.435312。

| 特征名      | 含义     |
| ---------- | :-----------:  |
| query_length     | query中词的总数     |
| title_length     | title中词的总数     |
| query_length_sub_title_length     | query与title词数的差     |
| query_length_ratio_title_length   | query与title词数的比     |
| mean_title_length     | 一个query下所有title包含词的平均数   |
| query_length_sub_mean_title_length    | query与mean_title词数的差   |
| title_length_sub_mean_title_length    | title与mean_title词数的差   |
| query_length_ratio_mean_title_length    | query与mean_title词数的比   |
| title_length_ratio_mean_title_length    | title与mean_title词数的比   |
| query_nunique_title     | 一个query对应的title的数量   |
| title_nunique_query     | 一个title对应的query的数量   |
| longest_common_subsequence     | 最长公共子序列长度(词级别)   |
| longest_common_subsequence_query_ratio     | 最长公共子序列跨度(start_loc - end_loc)与query_length 的比   |
| longest_common_subsequence_title_ratio     | 最长公共子序列跨度与title_length 的比   |
| lcsubsequence_ratio_t_length     | 最长公共子序列长度与title_length 的长度比   |
| longest_common_substring     | 最长公共子串长度(词级别)   |
| lcs_start_location_ratio     | 最长公共子串中第一个词位置(start_loc)与title_length的比   |
| lcs_mean_location_ratio     | 最长公共子串中词的平均位置与title_length的比   |
| lcs_total_loc_ratio     | 所有子串的位置和   |
| lcs_dense_ratio     | 所有子串的位置和与(length_query * length_title)的比   |
| lcstring_ratio_q_length     | 最长公共子串长度与query_length的长度比   |
| lcstring_ratio_t_length     | 最长公共子串长度与title_length的长度比   |
| unique_ratio     | title中去重后公共词数与query和tite中所有词数的比   |
| share_words_qt_length_ratio     | (query中公共词在title中的数量 + title中公共词在query中的数量) 与 (query_length + title_length)的比|
| query_common_words_len_ratio     | 公共词与query_length的长度比   |
| title_common_words_len_ratio     | 公共词与title_length的长度比   |
| query_common_words_loc_ratio     | query中公共词平均位置与query_length的比   |
| title_common_words_loc_ratio     | title中公共词平均位置与title_length的比   |
| query_common_set_ratio     | 去重后公共词数量与去重后query_length的比   |
| title_common_set_ratio     | 去重后公共词数量与去重后title_length的比   |
| query_title_set_ratio     | 2* 去重后公共词数量 与 (query_length + title_length) 的比(一种简单的相似度度量函数) |
| query_common_set_loc_ratio     | 去重query中公共词平均位置与query_length的比   |
| title_common_set_loc_ratio     | 去重title中公共词平均位置与title_length的比   |
| common_set_ratio     | 去重后公共词数量与未去重公共词数量的比 |
| fuzz_token_sort_ratio     | fuzzywuzzy.token_sort_ratio |
| fuzz_token_set_ratio     | fuzzywuzzy.token_set_ratio |
| levenshtein_jaro     | Levenshtein.jaro |
| levenshtein_distance     | Levenshtein.distance |
| levenshtein_distance     | Levenshtein.ratio |
| jaccard_sim     | 杰卡德相似度 |
| words_movement_distance     | Word2vec.wv.wmdistance(list_query, list_title) |
| dot_product     | query句向量与title句向量的点积 |
| braycurtis     | query句向量与title句向量的braycurtis距离 |
| cityblock     | query句向量与title句向量的cityblock距离 |
| correlation     | query句向量与title句向量的correlation距离 |
| cosine     | query句向量与title句向量的cosine距离 |
| euclidean     | query句向量与title句向量的euclidean距离 |
| minkowski     | query句向量与title句向量的minkowski距离, p=3 |
| dis_dot_product     | query与title去除公共词后，句向量的点积 |
| dis_braycurtis     | query与title去除公共词后，句向量的braycurtis距离  |
| dis_cityblock     | query与title去除公共词后，句向量的cityblock距离 |
| dis_correlation     | query与title去除公共词后，句向量的correlation距离 |
| dis_cosine     | query与title去除公共词后，句向量的cosine距离 |
| dis_euclidean     | query与title去除公共词后，句向量的euclidean距离 |
| dis_minkowski     | query与title去除公共词后，句向量的minkowski距离 |


#### esim_pytorch.ipynb
训练ESIM，纯nn模型，8kw训练集在A榜上能够取得0.588336的成绩，在最终测试集的效果应该更好，因为A榜训练集并没有作为我们训练词向量的数据集来源。Embedding层使用word2vec+fasttext拼接而成的词向量，即每个词对应300维，在训练过程中进行微调。

#### 模型融合
简单地对lgb和esim的结果加权融合，0.595266 lgb与0.586439 esim平均融合能取得0.612357的成绩，0.55*lgb + 0.45*esim 能取得0.612852的成绩。0.595266 lgb与0.587132 esim 平均融合能取得0.613485的成绩，0.55*lgb + 0.45*esim 能取得0.613587的成绩。


## 个人总结
 1.因为时间关系，没有尝试在esim中加入手工特征，听比赛群里的大佬说效果应该还蛮不错的。\
 2.个人也尝试了textcnn、abcnn等以cnn为基础的nn模型，但效果甚微，loss在几个batch后和esim的差距已经非常大。\
 3.我的老师给我介绍了一种方法，受自身水平所限未能实现。思路分享如下：构图，图中有两类节点，一类是词节点一类是句节点。句节点与句中词节点相连，在句节点之间设置一种相似度度量方法作为loss function，然后用GNN的方法训练，不断更新每个节点的隐藏状态。这是一种区别于lgb和esim的新方法，应该会有不错的效果。但是如何对训练集中未出现的词进行表示也一直是我在思考的问题。
