# WordFrequencyAnalysis
Compare Jieba and Droidtown ArticutAPI word segmentation and post tagging, and use the self-introduction of each company in the three industries as data to analyze the use of nouns and verbs in each industry.<br>
比較Jieba 與Droidtown ArticutAPI 中文文本的斷詞與詞性標註，並且選了三個產業（金融業、半導體產業與AI 新創）幾間公司，擷取其在官網中的自介，分別看名詞與動詞及TFIDF 特徵詞之詞頻分析與餘弦相似度計算。<br><br>

- [GoogleDrive 連結](https://drive.google.com/drive/folders/11dnCr1GPHcJDBMNvqC0WO_JCtTlhjSrX?usp=share_link)
- [Tableau Dashborad 連結](https://public.tableau.com/shared/RGDJGTFYS?:display_count=n&:origin=viz_share_link)
- [Doridtown Github 連結](https://github.com/Droidtown/ArticutAPI)
- [卓騰語言科技](https://www.droidtown.co/zh-tw/)


Abstract:<br>
在 [【人工智慧應用專題】語言學導向的NLP](https://www.youtube.com/watch?v=PbEQ3KE6WDE&t=7882s)，
請到[卓騰語言科技 總經理 王文傑老師](https://www.droidtown.co/zh-tw/) 介紹語言學導向的自然語言處理以及卓騰語言科技的產品。<br>
在這堂課中了解到在做中文自然語言處理與英文上的差別。中英文的結構不同導致一般常見處理技術如Word2Vec、BERT，這些基於英文為主流設計的演算法套用到中文時，雖然看起來很好，但效果仍有一段差距，例如BERT 不斷詞等等。課堂上有許多精彩的解釋與範例，這裡主要是延生老師於[第四堂課](https://www.youtube.com/watch?v=Z6KOlJ58Gfw)實作的衍生。<br>
透過這次小實作我有三個結論與心得，
1. Articut API 在中文文本不論是斷詞或是詞性標註得確與主流常用的Jieba 表現一樣好甚至有時會更好。
2. Articut API 我使用上覺得最明顯得缺點可能是其效率性，在斷詞時，雖然能夠得到的資訊較多，但花費時間也相對較久，若文本數量較大需要一段時間才能夠分析完成
3. 引入POS Tagging 的確在某些文本上能更好展現其特徵。<br><br>

Introduction:<br>
卓騰語言科技的Articut API，利用語言學的角度頗析中文句法的規則，斷詞Segmetation 的同時也做好POS taggint、Geo-Event Extraction、NER 等任務，以下我利用其API 與Jieba 做Segmetation 與POS Tagging，
1. 利用文字雲等方式評估Articut 與Jieba 誰在中文文本表現得較好。
2. 課堂上提到，TFIDF 只能代表特徵詞而非關鍵詞，所以另外引入POS Tagging，觀察特定詞性詞是否真得較能代表文本，以下利用名詞與動詞。

- [資料集](https://github.com/hsiehbocheng/WordFrequencyAnalysis/blob/main/data/公司自我介紹.xlsx)：三個產業各五間公司於其官網之自介擷取。<br>

- reference: [Droidtown/NLP_Training/Unit04](https://github.com/Droidtown/NLP_Training/tree/main/Unit04)

- Python code:<br>

1.  Articut v.s. Jieba<br>
使用Articut API 需要在官網申請金鑰，否則在 ``username`` 與 ``articut_key`` 輸入空字串，則使用每日的 2000 字額度。<br>
```py
config = { 
    "username":"", # use free 2,000 word per day without api key
    "articut_key":""}
    
articut = Articut(config['username'],config['articut_key'])
```
定義Articut 與Jieba 擷取斷詞與詞性標註結果，這邊參考老師的function，再自己加上一些變化，例如是否要去除停用詞等。<br>
Jieba 的詞性標記結果參考他們的[官網](https://github.com/fxsjy/jieba)<br>
    
```py
def wordExtractor(inputLIST, unify=True, StopWords=None):

    '''
    配合 Articut() 的 .getNounStemLIST() 和 .getVerbStemLIST() …等功能，拋棄位置資訊，只抽出詞彙。
    我另外又加上是否去除停用詞的選項，default 是 None，則和原本一樣，或著送入list 作為停用詞
    '''

    return output

def counterCosinSimilarity(RESULT1, RESULT2, terms=None, stopWords=None):
    '''
    微更改原本的demo function， 輸入變成吃 articut.parse 的 Result，wordExtractor 與Counter 我寫在function 裏面了
    '''
    dotprod = sum(counter01.get(k, 0) * counter02.get(k, 0) for k in terms)
    magA = math.sqrt(sum(counter01.get(k, 0) ** 2 for k in terms))
    magB = math.sqrt(sum(counter02.get(k, 0) ** 2 for k in terms))

    return dotprod / (magA * magB)

def jiebaPosExtractor(text, unify=True, StopWords=None):

    '''
    紀錄Jieba 斷詞語Pos tagging 的結果
    '''
    return result, verb_, num_, adj_, per_, 
```
與老師的demo 一樣，利用 ``articut.parse(text: str)`` 得到斷詞結果 ``result``，再利用 ``articut.getVerbStemLIST(result)`` 或是 ``articut.getNounStemLIST(result)`` 與 ``wordExtractor(inputLIST=VerbResultLIST, unify=True, StopWords=stopWords)``得到名詞與動詞。

```py
company = 'Appier'
text = rawData.loc[rawData.公司 == company]['簡介'].values[0]
result = articut.parse(text)

VerbResultLIST = articut.getVerbStemLIST(result)
NounResultLIST = articut.getNounStemLIST(result)

articutVerb = wordExtractor(inputLIST=VerbResultLIST, 
                            unify=True, 
                            StopWords=stopWords)
articutNoun = wordExtractor(inputLIST=NounResultLIST, 
                            unify=True, 
                            StopWords=stopWords)
result, verb_, num_, adj_, per_,  = jiebaPosExtractor(text=text, 
                                                    unify=True, 
                                                    StopWords=stopWords)
pd.DataFrame.from_dict({'articutVerb': articutVerb,
                'jiebaVerb': verb_,
                '': ['']* max(len(articutVerb), len(verb_), len(articutNoun), len(num_)),
                'articutNoun': articutNoun,
                'jiebaNoun':num_},
                orient='index').transpose().head(10)
```
<img src="https://github.com/hsiehbocheng/WordFrequencyAnalysis/blob/main/img/Appier_table.png" alt="Cover" width="50%"/>
<img src="https://github.com/hsiehbocheng/WordFrequencyAnalysis/blob/main/img/Appier.png" alt="Cover" width="50%"/>

<br>
可以發現在動詞的部分差異並不大，但名詞Articut 則可以吃到一些更代表的字詞，例如SaaS，因為該詞是英文，在Jieba 中會被分到 ``eng``，無法判斷詞性。<br>

接下來看看Awoo <br>
<img src="https://github.com/hsiehbocheng/WordFrequencyAnalysis/blob/main/img/Awoo_table.png" alt="Cover" width="50%"/>
<img src="https://github.com/hsiehbocheng/WordFrequencyAnalysis/blob/main/img/Awoo.png" alt="Cover" width="50%"/>
<br>
在Awoo 這邊可以看到，像這樣的新創或是Ai 公司一定會寫到英文如: AI、MarTech，``Articut API`` 顯現了其優勢，而``Jieba`` 在中文中會發生像：化、全，這樣的斷詞錯誤。<br>

2. 名詞、動詞與TFIDF 特徵詞詞頻餘弦相似度<br>

計算兩間公司在名詞、動詞與TFIDF 特徵詞的詞頻相似度，期望看到該公司的自介文本會與同產業其他公司相近。<br>

```py
company1 = '國泰金控'
company2 = '玉山金控'

text1 = rawData.loc[rawData.公司 == company1]['簡介'].values[0]
text2 = rawData.loc[rawData.公司 == company2]['簡介'].values[0]
result1 = articut.parse(text1)
result2 = articut.parse(text2)
resultLIST1 = articut.getVerbStemLIST(result1)
resultLIST2 = articut.getVerbStemLIST(result2)

similarity = counterCosinSimilarity(resultLIST1, resultLIST2)
print(f'{company1} vs {company2} 動詞相似度 {similarity}')

resultLIST1 = articut.getNounStemLIST(result1)
resultLIST2 = articut.getNounStemLIST(result2)

similarity = counterCosinSimilarity(resultLIST1, resultLIST2)
print(f'{company1} vs {company2} 名詞相似度 {similarity}')

resultLIST1 = articut.analyse.extract_tags(result1)
resultLIST2 = articut.analyse.extract_tags(result2)
similarity = counterCosinSimilarity(resultLIST1, resultLIST2)
print(f'{company1} vs {company2} TFIDF相似度 {similarity}')
```
```py
國泰金控 vs 玉山金控 動詞相似度 0.4822354489342862
國泰金控 vs 玉山金控 名詞相似度 0.22998675142327324
國泰金控 vs 玉山金控 TFIDF相似度 0.3424747597107866
```
<br>
上面可以發現動詞相似度最高，而TFIDF 贏過名詞，輸給動詞<br>

```py
company2 = 'iKala'
```
```py
國泰金控 vs iKala 動詞相似度 0.36103584630253305
國泰金控 vs iKala 名詞相似度 0.22857142857142856
```
<br>
而這裡可以發現國泰和玉山相似度都比國泰與iKala 高。<br>


```py
玉山金控 vs 國泰金控 動詞相似度 0.4822354489342862
玉山金控 vs 國泰金控 名詞相似度 0.22998675142327324
玉山金控 vs 國泰金控 TFIDF相似度 0.3424747597107866
```

```py
玉山金控 vs Appier 動詞相似度 0.11850314695868774
玉山金控 vs Appier 名詞相似度 0.03155092231950747
玉山金控 vs Appier TFIDF相似度 0.1851946331679053
```
<br>
這裡可以看出玉山金控與國泰金控之間的相似度高於玉山金控與Appier，符合認為同產業較相近的假設。

這邊有稍微用Tableau 製作簡單Dashborad，視覺化同產業之間的相似度<br><br>
<img src="https://github.com/hsiehbocheng/WordFrequencyAnalysis/blob/main/img/twb.png" alt="Cover" width="70%"/>
