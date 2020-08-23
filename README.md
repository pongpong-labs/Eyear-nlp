# Eyear-nlp
- 부산대학교 제3회 창의융합 소프트웨어 해커톤 참여작품

## Introduction
- 이 프로젝트의 목적은 수업 중 교수자의 음성을 자막처리한 텍스트의 오류를 찾아내어 수정하는 것입니다.
- 또한, 오류를 수정한 텍스트를 자막으로 띄워 더 높은 정확도를 가지는 자막을 만드는 것입니다.

## Dependency
    
    python >= 3.6
    tensorflow == 2.3.0
    keras == 2.4.3

## How to install
    
    git clone https://github.com/pongpong-labs/Eyear-nlp.git

## Quick Start

#### 학습해둔 word2vec 불러오기
    
    from gensim.models import Word2Vec
    from eunjeon import Mecab
    from korean_romanizer import *
    import jellyfish
    from pykospacing import spacing

    tagger = Mecab()

    model = Word2Vec.load('psychology.model')

#### 수업과 관련된 단어 불러오기
    
    text_list=[]
    with open('analysis_token.txt', 'r',encoding='utf-8') as f:
        for line in f:
            text_list.append(line.rstrip())
    
    except_words=[]
    with open('except_word.txt', 'r',encoding='utf-8') as f:
        except_words = []
        for line in f:
            except_words.append(line.rstrip())
            
#### 자막 분석에 필요한 조사, 어미, 띄어쓰기 불러오기
    
    josa_eomi = []
    with open("josaeomi.txt",'r', encoding = 'utf-8') as f:
        for line in f:
            josa_eomi.append(line.rstrip())


#### 자막 중 오류가 있는지 확인
    
    def find_error_word(): #오류가 있는지 확인
        error_word=[] #오류단어를 저장
        for i in jamak_nouns:
            if i not in text_list:
                error_word.append(i)
        return error_word
        
    def comb_error_word():
    global err_comb
    global err_word
    err_word = []
    err_comb = []
    for i in range(len(error_word)):
        for j in range(len(jamak)):
            if error_word[i] == jamak[j]:
                comb1 = []
                comb2 = []
                
                k = 0
                while jamak[j + k] not in josa_eomi:
                    real_err_word1 = jamak[j + k]
                    comb1.append(real_err_word1)
                    k += 1
                    
                k = 1
                while jamak[j - k] not in josa_eomi:
                    real_err_word2 = jamak[j - k]
                    comb2.insert(0,real_err_word2)
                    k += 1
                    
                err_comb.append(comb2 + comb1)
                
    
    for i in range(len(err_comb)):
        err_comb[i] = ''.join(err_comb[i])
    for v in err_comb:
        if v not in err_word:
            err_word.append(v)
    return err_word


#### 오류 단어 근처의 단어(명사) 뽑기 (근처의 단어가 오류인 경우 무시)

    def nearby_error_word(): #오류 단어 앞뒤의 3단어 뽑기
    global check_nouns
    check_nouns = []#오류단어 앞뒤의 단어 저장
    if err_word == []:
        return []
    else:
        for i in range(len(err_word)):
            for j in range(len(line_space)):
                if err_word[i] == line_space[j] or err_word[i] in tagger.nouns(line_space[j]) or error_word[i] in tagger.nouns(line_space[j]):
                    check_nouns_list = []
                    if j == len(line_space)-1:
                        try:
                            li1 = tagger.nouns(line_space[j - 1])
                            li2 = tagger.nouns(line_space[j - 2])
                            li3 = tagger.nouns(line_space[j - 3])
                            li4 = tagger.nouns(line_space[j - 4])
                            li5 = tagger.nouns(line_space[j - 5])
                            li6 = tagger.nouns(line_space[j - 6])
                            check_nouns_list.extend(li1+li2+li3+li4+li5+li6)
                        except:
                            pass
                        
                    elif j == len(line_space) - 2:
                        try:
                            li1 = tagger.nouns(line_space[j - 1])
                            li2 = tagger.nouns(line_space[j + 1])
                            li3 = tagger.nouns(line_space[j - 2])
                            li4 = tagger.nouns(line_space[j - 3])
                            li5 = tagger.nouns(line_space[j - 4])
                            li6 = tagger.nouns(line_space[j - 5])
                            check_nouns_list.extend(li1+li2+li3+li4+li5+li6)
                        except:
                            pass
                        
                    else:
                        try:
                            li1 = tagger.nouns(line_space[j - 1])
                            li2 = tagger.nouns(line_space[j + 1])
                            li3 = tagger.nouns(line_space[j - 2])
                            li4 = tagger.nouns(line_space[j + 2])
                            li5 = tagger.nouns(line_space[j - 3])
                            li6 = tagger.nouns(line_space[j - 4])
                            check_nouns_list.extend(li1+li2+li3+li4+li5+li6)
                        except:
                            pass
                        
                    check_nouns.append(check_nouns_list)    
                    if check_nouns[0] == []:
                        return []
    return check_nouns


#### 선택된 단어들 중 word2vec으로 학습된 가장 연관성 높은 단어의 리스트 만들기
    
    def check_word_list(): #오류 단어 근처의 단어에 대해 word2vec으로 학습한 연관성 높은 단어의 리스트를 출력
    global word_list
    word_list = []
    global model_result
    if check_nouns == []:
        return []

    else:
        for i in range(len(check_nouns)):
            list_result = []
            for j in check_nouns[i]:
                try:
                    model_result = model.wv.most_similar(j, topn=100)
                    for k in model_result:
                        if k[0] not in except_words:
                            list_result.append(k[0])
                except:
                    pass
            word_list.append(list_result)
            
            
            if list_result == []:
                return []
            
    return word_list

#### 단어 리스트들의 발음을 로마자로 변환 
    
    def romanizing(): #word_list의 한글 발음을 로마자로 변환
    global pronounce
    pronounce = [] # word_list의 발음을 저장.
    if word_list == []:
        return []
    else:
        for i in range(len(word_list)):
            pronounce_list = []
            for j in range(len(word_list[i])):
                try:
                    a = Romanizer(word_list[i][j])
                    pronounce_list.append(a.romanize())
                except:
                    pass
            pronounce.append(pronounce_list)
    return pronounce


#### 로마자로 변환한 단어들의 발음 유사도 측정
    
    def similarity(): #error word와 word list 단어의 발음 유사도 측정
    global probability
    probability = []
    
    if pronounce == []:
        return []
    
    else:
        for e in range(len(err_word)):
            a = Romanizer(err_word[e]).romanize()
            prob = []
            prob1 = []
            prob2 = []
            prob3 = []
            try:
                for j in range(len(pronounce[e])):
                    prob1.append(jellyfish.jaro_winkler_similarity(a, pronounce[e][j]))
                    prob2.append(jellyfish.jaro_similarity(a, pronounce[e][j]))
                    prob3.append(1-(jellyfish.levenshtein_distance(a,pronounce[e][j]))/7)
                    prob.append(prob3[j] + (prob1[j] + prob2[j]))
                probability.append(prob)
            except:
                pass
        if pronounce[0] == []:
            return []
    return probability

#### 오류 단어를 가장 높은 발음 유사도를 가지는 단어로 교체

    def word_change(): #오류단어를 교체
    global correct_word
    global line_space
    correct_word = []
    
    if probability == []:
        return line_space
    
    else:
        change_word = []
        try:
            for i in range(len(probability)):
                err_word_index = probability[i].index(max(probability[i]))
                correct_word.append(word_list[i][err_word_index])

            for a in range(len(err_word)):
                for b in range(len(line_space)):
                    if err_word[a] == line_space[b]:
                        line_space[b] = correct_word[a]
        except:
            pass
    return line_space
    
    
    def word_change2():
    global line_space
    
    if probability == []:
        return line_space
    
    else:
        try:
            line_space = word_change()
            line_nnn = spacing(''.join(line_space))
            line_nnn = tagger.morphs(line_nnn)
            for a in range(len(err_word)):
                for b in range(len(line_nnn)):
                    if err_word[a] == line_nnn[b]:
                        line_nnn[b] = correct_word[a]
                        line_space = line_nnn
        except:
            pass
                    
    return line_space

#### 자막 불러오기
    f=open(r'자막이 들어올 위치',encoding="utf8") #자막을 한 줄씩 받을 예정

#### 자막 처리 (리스트형태)
    
    while True:
        line=f.readline()
        jamak_nn=tagger.nouns(line) #자막에 나오는 명사를 찾기
    
        if not line:
            break
    
        jamak_nouns=[]
        for w in jamak_nn:
            if w not in except_words:
                jamak_nouns.append(w)
    
        if tagger.morphs(line)!=[]: #자막문장을 품사별로 끊기
            jamak=tagger.morphs(line)

        error_word=find_error_word()
        err_word=comb_error_word()
        check_nouns=nearby_error_word()
        word_list=check_word_list()
        pronounce=romanizing()
        probability=similarity()
        change_word=word_change()
        change_word2=word_change2()
      
    
#### 자막으로 변환  (리스트 형태를 str 형태로)  
    
        final_line=''.join(change_word2)
        print(spacing(final_line),'\n')    
    f.close()


## Training Word2Vec
    from eunjeon import Mecab
    tagger = Mecab()
    
    f= open(r'학습할 데이터.txt',encoding="utf-8")

    text=[]
    while True:
        line=f.readline()
        if tagger.nouns(line)!=[]:
            text.append(tagger.nouns(line))
        if not line:
            break
    f.close()
    
    from gensim.models import Word2Vec
    model = Word2Vec(text,size=500,window=7,min_count=5,workers=6,sg=1)
    
    model.save('psychology.model')
    
## 정답 단어 정의 (여기에 포함되지 않는 단어를 오류 단어 취급)
    
    text_list=sum(text,[])
    text_list=set(text_list)
    text_list=list(text_list)
    
    f=open(r'파일위치\파일이름.txt','w',encoding='utf-8')
    for i in text_list:
        f.write(i+'\n')
    f.close()


## 참조
1. https://github.com/koshort/pyeunjeon 
2. https://github.com/jamesturk/jellyfish
3. https://pypi.org/project/mykoreanromanizer/
4. https://github.com/haven-jeon/PyKoSpacing.git
