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
    
    git clone 

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

#### 자막 중 오류가 있는지 확인
    
    def find_error_word(): #오류가 있는지 확인
        error_word=[] #오류단어를 저장
        for i in jamak_nouns:
            if i not in text_list:
                error_word.append(i)
        return error_word

#### 오류 단어 근처의 단어(명사) 뽑기 (근처의 단어가 오류인 경우 무시)

    def nearby_error_word(): 
        check_nouns=[]
        if len(jamak_nouns)<4:
            return []
        else:
            for i in range(len(error_word)):
                for j in range(len(jamak_nouns)):
                    if error_word[i] == jamak_nouns[j]:
                        check_nouns_list=[]
                        if j == len(jamak_nouns)-1:
                            check_nouns_list.append(jamak_nouns[j-1])
                            check_nouns_list.append(jamak_nouns[j-2])
                            check_nouns_list.append(jamak_nouns[j-3])
                        else:
                            check_nouns_list.append(jamak_nouns[j-1])
                            check_nouns_list.append(jamak_nouns[j+1])
                            check_nouns_list.append(jamak_nouns[j-2])
                        check_nouns.append(check_nouns_list)    
        return check_nouns


#### 선택된 단어들 중 word2vec으로 학습된 가장 연관성 높은 단어의 리스트 만들기
    
    def check_word_list(): 
        word_list=[]
        global model_result
        if check_nouns==[]:
            return []
    
        else:
            for i in range(len(check_nouns)):
                list_result=[]
                for j in check_nouns[i]:
                    try:
                        model_result=model.wv.most_similar(j)
                    except:
                        pass
                    for k in model_result:
                        if k[0] not in except_words:
                            list_result.append(k[0])
                word_list.append(list_result)
       
        return word_list

#### 단어 리스트들의 발음을 로마자로 변환 
    
    def romanizing(): #word_list의 단어들의 한글 발음을 로마자로 변환
        pronounce=[] # word_list의 발음을 저장.
        if word_list == []:
            return []
        else:
            for i in range(len(word_list)):
                pronounce_list=[]
                for j in range(len(word_list[i])):
                    a=Romanizer(word_list[i][j])
                    pronounce_list.append(a.romanize())
                pronounce.append(pronounce_list)
        return pronounce


#### 로마자로 변환한 단어들의 발음 유사도 측정
    
    def similarity(): #error word와 word list 단어의 발음 유사도 측정
        probability=[]
        if check_nouns == []:
            return []
        else:
            for e in range(len(error_word)):
                a=Romanizer(error_word[e]).romanize()
                prob=[]
                prob1=[]
                prob2=[]
                prob3=[]
                for j in range(len(pronounce[e])):
                    prob1.append(jellyfish.jaro_winkler_similarity(a, pronounce[e][j]))
                    prob2.append(jellyfish.jaro_similarity(a, pronounce[e][j]))
                    prob3.append(1-(jellyfish.levenshtein_distance(a,pronounce[e][j]))/10)
                    prob.append(prob3[j]+(prob1[j]+prob2[j])/2)
                probability.append(prob)
        return probability

#### 오류 단어를 가장 높은 발음 유사도를 가지는 단어로 교체

    def word_change(): #오류단어를 교체
        global correct_word
        correct_word=[]
        if check_nouns==[]:
            return jamak
        else:
            change_word=[]
            for i in range(len(probability)):
                err_word_index=probability[i].index(max(probability[i]))
                correct_word.append(word_list[i][err_word_index])
                
            for a in range(len(jamak)):
                for b in range(len(error_word)):
                    if jamak[a] == error_word[b]:
                        jamak[a]=correct_word[b]
        return jamak

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
        check_nouns=nearby_error_word()
        word_list=check_word_list()
        pronounce=romanizing()
        probability=similarity()
        change_word=word_change()
      
    
#### 자막으로 변환  (리스트 형태를 str 형태로)  
    
        jamak=''.join(jamak)
        print(spacing(jamak),'\n')    
    f.close()


참조
1. https://github.com/koshort/pyeunjeon 
2. https://github.com/jamesturk/jellyfish
3. https://pypi.org/project/mykoreanromanizer/
