# WordFrequencyAnalysis
Compare Jieba and Droidtown ArticutAPI word segmentation and post tagging, and use the self-introduction of each company in the three industries as data to analyze the use of nouns and verbs in each industry
比較Jieba 與Droidtown ArticutAPI 中文文本的斷詞與詞性標註，並且選了三個產業（金融業、半導體產業與AI 新創）幾間公司，擷取其在官網中的自介，分別看名詞與動詞及TFIDF 特徵詞之詞頻分析與餘弦相似度計算。<br><br>

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

- reference: [Droidtown/NLP_Training/Unit04])https://github.com/Droidtown/NLP_Training/tree/main/Unit04)

- Python code:<br>

使用Articut API 需要在官網申請金鑰，否則在 ``username`` 與 ``articut_key`` 輸入空字串，則使用每日的 2000 字額度。
```py
config = { 
    "username":"", # use free 2,000 word per day without api key
    "articut_key":""}
    
articut = Articut(config['username'],config['articut_key'])
```
定義Articut 與Jieba 擷取斷詞與詞性標註結果，這邊參考老師的function，再自己加上一些變化，例如是否要去除停用詞等。
    
```py
def wordExtractor(inputLIST, unify=True, StopWords=None):

    '''
    配合 Articut() 的 .getNounStemLIST() 和 .getVerbStemLIST() …等功能，拋棄位置資訊，只抽出詞彙。
    我另外又加上是否去除停用詞的選項，default 是 None，則和原本一樣，或著送入list 作為停用詞
    '''
    resultLIST = []

    for item in inputLIST:
        if item == []:
            pass
        else:
            for word in item:
                resultLIST.append(word[-1])
    if StopWords == None:
        pass
    else:
        resultLIST = [word for word in resultLIST if word not in StopWords]
        
    if unify == True:
        output = sorted(list(set(resultLIST)))
    else:
        output =  sorted(resultLIST)

    return output

def counterCosinSimilarity(RESULT1, RESULT2, terms=None, stopWords=None):
    '''
    微更改原本的demo function， 輸入變成吃 articut.parse 的 Result，wordExtractor 與Counter 我寫在function 裏面了
    '''

    counter01 = Counter(wordExtractor(inputLIST=RESULT1,
                                    unify=False, 
                                    StopWords=stopWords))
    counter02 = Counter(wordExtractor(inputLIST=RESULT2,
                                    unify=False, 
                                    StopWords=stopWords))
    
    if terms == None:
        terms = set(counter01).union(counter02)
    dotprod = sum(counter01.get(k, 0) * counter02.get(k, 0) for k in terms)
    magA = math.sqrt(sum(counter01.get(k, 0) ** 2 for k in terms))
    magB = math.sqrt(sum(counter02.get(k, 0) ** 2 for k in terms))

    return dotprod / (magA * magB)

def jiebaPosExtractor(text, unify=True, StopWords=None):

    '''
    紀錄Jieba 斷詞語Pos tagging 的結果
    '''

    words = pseg.cut(text)

    result = []
    verb_ = []
    num_ = []
    adj_ = []
    per_ = []
    for word, flag in words:
        if StopWords != None:
            if word in StopWords:
                continue

        result.append((flag, word))
        if flag in ['v', 'vd','vn']:
            verb_.append(word)
        elif flag in ['n', 'nr', 'nz']:
            num_.append(word)
        elif flag in ['nr', 'PER']:
            per_.append(word)
        elif flag in ['a', 'ad', 'an', 'd']:
            adj_.append(word)

    
    if unify == True:
        verb_ = sorted(list(set(verb_)))
        num_ = sorted(list(set(num_)))
        adj_ = sorted(list(set(adj_)))
        per_ = sorted(list(set(per_)))

    return result, verb_, num_, adj_, per_, 
```
