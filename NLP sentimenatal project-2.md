```python
pip install Keras
```

    Requirement already satisfied: Keras in c:\users\javee\anaconda3\lib\site-packages (2.9.0)
    Note: you may need to restart the kernel to use updated packages.
    


```python
pip install gensim
```

    Requirement already satisfied: gensim in c:\users\javee\anaconda3\lib\site-packages (4.1.2)
    Requirement already satisfied: scipy>=0.18.1 in c:\users\javee\anaconda3\lib\site-packages (from gensim) (1.7.3)
    Requirement already satisfied: smart-open>=1.8.1 in c:\users\javee\anaconda3\lib\site-packages (from gensim) (5.1.0)
    Requirement already satisfied: numpy>=1.17.0 in c:\users\javee\anaconda3\lib\site-packages (from gensim) (1.21.5)
    Note: you may need to restart the kernel to use updated packages.
    


```python
pip install nltk
```

    Requirement already satisfied: nltk in c:\users\javee\anaconda3\lib\site-packages (3.7)
    Requirement already satisfied: joblib in c:\users\javee\anaconda3\lib\site-packages (from nltk) (1.1.0)
    Requirement already satisfied: tqdm in c:\users\javee\anaconda3\lib\site-packages (from nltk) (4.64.0)
    Requirement already satisfied: click in c:\users\javee\anaconda3\lib\site-packages (from nltk) (8.0.4)
    Requirement already satisfied: regex>=2021.8.3 in c:\users\javee\anaconda3\lib\site-packages (from nltk) (2022.3.15)
    Requirement already satisfied: colorama in c:\users\javee\anaconda3\lib\site-packages (from click->nltk) (0.4.4)
    Note: you may need to restart the kernel to use updated packages.
    


```python

import re
import numpy as np 
import pandas as pd 
import warnings
warnings.filterwarnings('ignore')
from nltk.stem import WordNetLemmatizer
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras.models import Sequential
from tensorflow.keras.preprocessing import text, sequence
from tensorflow.keras.preprocessing.sequence import pad_sequences
from tensorflow.keras import regularizers
from sklearn.model_selection import train_test_split
import nltk
from nltk.corpus import stopwords
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn import naive_bayes
from sklearn.metrics import roc_auc_score

```


```python
df=pd.read_csv('train.csv.zip')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Labels</th>
      <th>Text</th>
      <th>Text_Tag</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Says the Annies List political group supports ...</td>
      <td>abortion</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>When did the decline of coal start? It started...</td>
      <td>energy,history,job-accomplishments</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Hillary Clinton agrees with John McCain "by vo...</td>
      <td>foreign-policy</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>Health care reform legislation is likely to ma...</td>
      <td>health-care</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>The economic turnaround started at the end of ...</td>
      <td>economy,jobs</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.info
```




    <bound method DataFrame.info of        Labels                                               Text  \
    0           1  Says the Annies List political group supports ...   
    1           2  When did the decline of coal start? It started...   
    2           3  Hillary Clinton agrees with John McCain "by vo...   
    3           1  Health care reform legislation is likely to ma...   
    4           2  The economic turnaround started at the end of ...   
    ...       ...                                                ...   
    10235       3  There are a larger number of shark attacks in ...   
    10236       3  Democrats have now become the party of the [At...   
    10237       2  Says an alternative to Social Security that op...   
    10238       1  On lifting the U.S. Cuban embargo and allowing...   
    10239       4  The Department of Veterans Affairs has a manua...   
    
                                     Text_Tag  
    0                                abortion  
    1      energy,history,job-accomplishments  
    2                          foreign-policy  
    3                             health-care  
    4                            economy,jobs  
    ...                                   ...  
    10235                   animals,elections  
    10236                           elections  
    10237          retirement,social-security  
    10238              florida,foreign-policy  
    10239                health-care,veterans  
    
    [10240 rows x 3 columns]>




```python
df.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Labels</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>10240.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.328613</td>
    </tr>
    <tr>
      <th>std</th>
      <td>1.650933</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>3.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>5.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
df[df.duplicated(['Text'])]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Labels</th>
      <th>Text</th>
      <th>Text_Tag</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1014</th>
      <td>2</td>
      <td>On abortion</td>
      <td>abortion,candidates-biography</td>
    </tr>
    <tr>
      <th>1814</th>
      <td>1</td>
      <td>On support for gay marriage.</td>
      <td>civil-rights,families,gays-and-lesbians,marriage</td>
    </tr>
    <tr>
      <th>1846</th>
      <td>1</td>
      <td>Obama says Iran is a 'tiny' country, 'doesn't ...</td>
      <td>foreign-policy</td>
    </tr>
    <tr>
      <th>2697</th>
      <td>1</td>
      <td>On repealing the 17th Amendment</td>
      <td>debates,elections,states</td>
    </tr>
    <tr>
      <th>2846</th>
      <td>3</td>
      <td>Four balanced budgets in a row, with no new ta...</td>
      <td>job-accomplishments,jobs,state-budget,state-fi...</td>
    </tr>
    <tr>
      <th>3256</th>
      <td>1</td>
      <td>On a cap-and-trade plan.</td>
      <td>cap-and-trade,climate-change,environment</td>
    </tr>
    <tr>
      <th>4386</th>
      <td>1</td>
      <td>On the Trans-Pacific Partnership.</td>
      <td>trade</td>
    </tr>
    <tr>
      <th>4839</th>
      <td>2</td>
      <td>During Sherrod Browns past decade as a D.C. po...</td>
      <td>economy,job-accomplishments,jobs</td>
    </tr>
    <tr>
      <th>4940</th>
      <td>1</td>
      <td>On changing the rules for filibusters on presi...</td>
      <td>congressional-rules</td>
    </tr>
    <tr>
      <th>6759</th>
      <td>2</td>
      <td>On torture.</td>
      <td>human-rights,terrorism</td>
    </tr>
    <tr>
      <th>6784</th>
      <td>1</td>
      <td>On support for the Export-Import Bank</td>
      <td>trade</td>
    </tr>
    <tr>
      <th>7248</th>
      <td>2</td>
      <td>On the status of illegal immigrants</td>
      <td>immigration</td>
    </tr>
    <tr>
      <th>7647</th>
      <td>5</td>
      <td>Six justices on the U.S. Supreme Court have be...</td>
      <td>congress,legal-issues,supreme-court</td>
    </tr>
    <tr>
      <th>8906</th>
      <td>5</td>
      <td>Says Mitt Romney flip-flopped on abortion.</td>
      <td>abortion,message-machine-2012</td>
    </tr>
    <tr>
      <th>9400</th>
      <td>2</td>
      <td>Twenty million Americans are out of work.</td>
      <td>jobs</td>
    </tr>
    <tr>
      <th>9642</th>
      <td>1</td>
      <td>On changing the rules for filibusters on presi...</td>
      <td>congressional-rules</td>
    </tr>
    <tr>
      <th>9750</th>
      <td>0</td>
      <td>Some 20,000 Delphi salaried retirees lost up t...</td>
      <td>corporations,economy</td>
    </tr>
  </tbody>
</table>
</div>




```python
df[df.duplicated(['Text_Tag'])]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Labels</th>
      <th>Text</th>
      <th>Text_Tag</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14</th>
      <td>0</td>
      <td>Most of the (Affordable Care Act) has already ...</td>
      <td>health-care</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2</td>
      <td>In this last election in November, ... 63 perc...</td>
      <td>elections</td>
    </tr>
    <tr>
      <th>17</th>
      <td>0</td>
      <td>U.S. Rep. Ron Kind, D-Wis., and his fellow Dem...</td>
      <td>federal-budget</td>
    </tr>
    <tr>
      <th>25</th>
      <td>1</td>
      <td>I dont know who (Jonathan Gruber) is.</td>
      <td>health-care</td>
    </tr>
    <tr>
      <th>27</th>
      <td>2</td>
      <td>Rick Perry has never lost an election and rema...</td>
      <td>candidates-biography</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>10228</th>
      <td>0</td>
      <td>Stopped by Smiley Cookie to pick up some great...</td>
      <td>food</td>
    </tr>
    <tr>
      <th>10231</th>
      <td>2</td>
      <td>When it comes to the state deficit, Wisconsin ...</td>
      <td>state-budget</td>
    </tr>
    <tr>
      <th>10232</th>
      <td>2</td>
      <td>Eighty percent of the net new jobs created in ...</td>
      <td>economy,jobs</td>
    </tr>
    <tr>
      <th>10236</th>
      <td>3</td>
      <td>Democrats have now become the party of the [At...</td>
      <td>elections</td>
    </tr>
    <tr>
      <th>10239</th>
      <td>4</td>
      <td>The Department of Veterans Affairs has a manua...</td>
      <td>health-care,veterans</td>
    </tr>
  </tbody>
</table>
<p>6412 rows ?? 3 columns</p>
</div>




```python
df=df.drop_duplicates(['Text'])
```


```python
df=df.drop_duplicates(['Text_Tag'])
```


```python
df.sample(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Labels</th>
      <th>Text</th>
      <th>Text_Tag</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5043</th>
      <td>2</td>
      <td>Says Anthropology is a STEM (science, technolo...</td>
      <td>education,science</td>
    </tr>
    <tr>
      <th>3909</th>
      <td>2</td>
      <td>Although we have twice the population of Greec...</td>
      <td>economy,state-finances</td>
    </tr>
    <tr>
      <th>4372</th>
      <td>1</td>
      <td>This Congressadjourned earliest of any time in...</td>
      <td>congress,elections,history</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.isnull().sum()
```




    Labels      0
    Text        0
    Text_Tag    1
    dtype: int64




```python
df.dropna(inplace=True)
```


```python

```

-----------------------------------------------------------------------------


```python
import string
regular_punct = list(string.punctuation)
def remove_punctuation(text,punct_list):
    for punc in punct_list:
        if punc in text:
            text = text.replace(punc, ' ')
    return text.strip()

```

convertig into lower


```python
df["Text"] = df["Text"].apply(lambda x:x.lower())
```


```python
df["Text"][0]
```




    'says the annies list political group supports third-trimester abortions on demand.'




```python
df['Text_Tag']=df['Text_Tag'].apply(lambda x:x.lower())
```


```python
df['Text_Tag'][0]
```




    'abortion'



-----------------------------------------------------------------------------------------------------------------------------


```python
import string
special = string.punctuation
special
```




    '!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'




```python
df["Text"] = df["Text"].apply(lambda x:re.sub("[#!$%&\'()*+,-./:;<=>?@[\\]^_`{|}~]","",x))
```


```python
df['Text_Tag']=df['Text_Tag'].apply(lambda x:re.sub('[!#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~]',"",x))
```

----------------------------------------------------------------------------------------------------------------------------

Removing Stop Words


```python
import nltk
nltk.download("stopwords")
from nltk.corpus import stopwords
```

    [nltk_data] Downloading package stopwords to
    [nltk_data]     C:\Users\javee\AppData\Roaming\nltk_data...
    [nltk_data]   Package stopwords is already up-to-date!
    


```python
stope = stopwords.words('english')
stope
```




    ['i',
     'me',
     'my',
     'myself',
     'we',
     'our',
     'ours',
     'ourselves',
     'you',
     "you're",
     "you've",
     "you'll",
     "you'd",
     'your',
     'yours',
     'yourself',
     'yourselves',
     'he',
     'him',
     'his',
     'himself',
     'she',
     "she's",
     'her',
     'hers',
     'herself',
     'it',
     "it's",
     'its',
     'itself',
     'they',
     'them',
     'their',
     'theirs',
     'themselves',
     'what',
     'which',
     'who',
     'whom',
     'this',
     'that',
     "that'll",
     'these',
     'those',
     'am',
     'is',
     'are',
     'was',
     'were',
     'be',
     'been',
     'being',
     'have',
     'has',
     'had',
     'having',
     'do',
     'does',
     'did',
     'doing',
     'a',
     'an',
     'the',
     'and',
     'but',
     'if',
     'or',
     'because',
     'as',
     'until',
     'while',
     'of',
     'at',
     'by',
     'for',
     'with',
     'about',
     'against',
     'between',
     'into',
     'through',
     'during',
     'before',
     'after',
     'above',
     'below',
     'to',
     'from',
     'up',
     'down',
     'in',
     'out',
     'on',
     'off',
     'over',
     'under',
     'again',
     'further',
     'then',
     'once',
     'here',
     'there',
     'when',
     'where',
     'why',
     'how',
     'all',
     'any',
     'both',
     'each',
     'few',
     'more',
     'most',
     'other',
     'some',
     'such',
     'no',
     'nor',
     'not',
     'only',
     'own',
     'same',
     'so',
     'than',
     'too',
     'very',
     's',
     't',
     'can',
     'will',
     'just',
     'don',
     "don't",
     'should',
     "should've",
     'now',
     'd',
     'll',
     'm',
     'o',
     're',
     've',
     'y',
     'ain',
     'aren',
     "aren't",
     'couldn',
     "couldn't",
     'didn',
     "didn't",
     'doesn',
     "doesn't",
     'hadn',
     "hadn't",
     'hasn',
     "hasn't",
     'haven',
     "haven't",
     'isn',
     "isn't",
     'ma',
     'mightn',
     "mightn't",
     'mustn',
     "mustn't",
     'needn',
     "needn't",
     'shan',
     "shan't",
     'shouldn',
     "shouldn't",
     'wasn',
     "wasn't",
     'weren',
     "weren't",
     'won',
     "won't",
     'wouldn',
     "wouldn't"]




```python
df["Text"][0]
```




    'says the annies list political group supports thirdtrimester abortions on demand'




```python
df["Text"] = df["Text"].apply(lambda x:" ".join(i for i in x.split(" ") if i not in stope))
```


```python
df['Text'][0]
```




    'says annies list political group supports thirdtrimester abortions demand'



--------------------------------------------------------------------------------------------------------------------------------


```python
import nltk
nltk.download('wordnet')
nltk.download('omw-1.4')
```

    [nltk_data] Downloading package wordnet to
    [nltk_data]     C:\Users\javee\AppData\Roaming\nltk_data...
    [nltk_data]   Package wordnet is already up-to-date!
    [nltk_data] Downloading package omw-1.4 to
    [nltk_data]     C:\Users\javee\AppData\Roaming\nltk_data...
    [nltk_data]   Package omw-1.4 is already up-to-date!
    




    True




```python
def lemmatize_words(words):
    """Lemmatize words in text"""

    lemmatizer = WordNetLemmatizer()
    return [lemmatizer.lemmatize(word) for word in words]
```


```python
df['Text']=df['Text'].apply(lambda x:"".join (lemmatize_words(x)))
```


```python
df['Text'][0]
```




    'says annies list political group supports thirdtrimester abortions demand'




```python
def lemmatize_words(words):
    """Lemmatize words in text"""

    lemmatizer = WordNetLemmatizer()
    return [lemmatizer.lemmatize(word) for word in words]

```


```python
df['Text_Tag']=df['Text_Tag'].apply(lambda x:"".join (lemmatize_words(x)))
```


```python
df['Text_Tag'][0]
```




    'abortion'



-------------------------------------------------------------------------------------------------------------------------------


```python
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Labels</th>
      <th>Text</th>
      <th>Text_Tag</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>says annies list political group supports thir...</td>
      <td>abortion</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>decline coal start started natural gas took st...</td>
      <td>energyhistoryjobaccomplishments</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>hillary clinton agrees john mccain "by voting ...</td>
      <td>foreignpolicy</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>health care reform legislation likely mandate ...</td>
      <td>healthcare</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>economic turnaround started end term</td>
      <td>economyjobs</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>10233</th>
      <td>4</td>
      <td>mayor fung wants punish childrens education re...</td>
      <td>childrencitybudgetdeficiteducationstatebudgett...</td>
    </tr>
    <tr>
      <th>10234</th>
      <td>2</td>
      <td>ruling supreme court lobbyist could go legisla...</td>
      <td>corporationselections</td>
    </tr>
    <tr>
      <th>10235</th>
      <td>3</td>
      <td>larger number shark attacks florida cases vote...</td>
      <td>animalselections</td>
    </tr>
    <tr>
      <th>10237</th>
      <td>2</td>
      <td>says alternative social security operates galv...</td>
      <td>retirementsocialsecurity</td>
    </tr>
    <tr>
      <th>10238</th>
      <td>1</td>
      <td>lifting us cuban embargo allowing travel cuba</td>
      <td>floridaforeignpolicy</td>
    </tr>
  </tbody>
</table>
<p>3822 rows ?? 3 columns</p>
</div>




```python
#Text= pd.DataFrame(df['Text'].tolist()).astype(str)
#Text_Tag = pd.DataFrame(df['Text_Tag'].tolist()).astype(str)
```


```python
df['NewsText'] = df['Text_Tag'].astype(str) +" "+ df['Text']
df['NewsText'] = df['Text_Tag'].astype(str) +" "+ df['Text']
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Labels</th>
      <th>Text</th>
      <th>Text_Tag</th>
      <th>NewsText</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>says annies list political group supports thir...</td>
      <td>abortion</td>
      <td>abortion says annies list political group supp...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>decline coal start started natural gas took st...</td>
      <td>energyhistoryjobaccomplishments</td>
      <td>energyhistoryjobaccomplishments decline coal s...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>hillary clinton agrees john mccain "by voting ...</td>
      <td>foreignpolicy</td>
      <td>foreignpolicy hillary clinton agrees john mcca...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1</td>
      <td>health care reform legislation likely mandate ...</td>
      <td>healthcare</td>
      <td>healthcare health care reform legislation like...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>economic turnaround started end term</td>
      <td>economyjobs</td>
      <td>economyjobs economic turnaround started end term</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>10233</th>
      <td>4</td>
      <td>mayor fung wants punish childrens education re...</td>
      <td>childrencitybudgetdeficiteducationstatebudgett...</td>
      <td>childrencitybudgetdeficiteducationstatebudgett...</td>
    </tr>
    <tr>
      <th>10234</th>
      <td>2</td>
      <td>ruling supreme court lobbyist could go legisla...</td>
      <td>corporationselections</td>
      <td>corporationselections ruling supreme court lob...</td>
    </tr>
    <tr>
      <th>10235</th>
      <td>3</td>
      <td>larger number shark attacks florida cases vote...</td>
      <td>animalselections</td>
      <td>animalselections larger number shark attacks f...</td>
    </tr>
    <tr>
      <th>10237</th>
      <td>2</td>
      <td>says alternative social security operates galv...</td>
      <td>retirementsocialsecurity</td>
      <td>retirementsocialsecurity says alternative soci...</td>
    </tr>
    <tr>
      <th>10238</th>
      <td>1</td>
      <td>lifting us cuban embargo allowing travel cuba</td>
      <td>floridaforeignpolicy</td>
      <td>floridaforeignpolicy lifting us cuban embargo ...</td>
    </tr>
  </tbody>
</table>
<p>3822 rows ?? 4 columns</p>
</div>




```python
#df['Text']=df['Text'].apply(lambda x: str(x))
#df['Text_Tag']=df['Text_Tag'].apply(lambda x: str(x))
```


```python
type(df['Text'][2])
```




    str




```python
df=df.drop(columns=['Text','Text_Tag'])
```

--------------------------------------------------------------------------------------------------------------------------------


```python
df['column']=df['NewsText'].apply(lambda x:len(x.split(" ")))

```


```python
df['column'].value_counts()
```




    11     373
    10     363
    9      343
    12     338
    13     299
    8      295
    14     270
    15     242
    7      212
    16     200
    17     148
    6      121
    18     118
    19     116
    20      73
    5       58
    21      50
    22      40
    4       37
    23      33
    24      18
    25      18
    3       12
    26      10
    28       7
    30       7
    27       5
    29       4
    34       3
    2        2
    32       2
    35       1
    31       1
    196      1
    38       1
    48       1
    Name: column, dtype: int64



-----------------------------------------------------------------------------------------------------------------------------

Train Test Split


```python
x=df.drop(['Labels','column'],axis=1)
y=df['Labels']
```


```python
x_train, x_test, y_train, y_test = train_test_split(x, y,shuffle=True, test_size=0.33, random_state=111)
```


```python
x.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NewsText</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>abortion says annies list political group supp...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>energyhistoryjobaccomplishments decline coal s...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>foreignpolicy hillary clinton agrees john mcca...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>healthcare health care reform legislation like...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>economyjobs economic turnaround started end term</td>
    </tr>
  </tbody>
</table>
</div>




```python
x.shape
```




    (3822, 1)




```python
y.shape
```




    (3822,)




```python
x_train.shape
```




    (2560, 1)




```python
y_train.shape
```




    (2560,)




```python
from keras.preprocessing.text import Tokenizer
from keras_preprocessing.sequence import pad_sequences
```


```python
# tokenize text
token=Tokenizer()
token.fit_on_texts(x_train.NewsText)
word_index=token.word_index
vocab_size=len(word_index)
```


```python
len(word_index)
```




    9275




```python
vocab_size
```




    9275



--------------------------------------------------------------------------------------------------------------------------------


```python
 # padding data
Sequences= token.texts_to_sequences(df['NewsText'])
padded_seq= pad_sequences(Sequences, maxlen=20,padding='post',truncating='post')
```


```python
padded_seq[1]
```




    array([5102,  594,  642,  369, 1585,  188,  133,  369, 2012,    7,  218,
            412, 1428,   73,    0,    0,    0,    0,    0,    0])




```python
Sequences
```




    [[171, 1, 3808, 942, 336, 311, 211, 2286, 509, 2323],
     [5102, 594, 642, 369, 1585, 188, 133, 369, 2012, 7, 218, 412, 1428, 73],
     [6661, 77, 61, 3017, 141, 706, 6662, 317, 235, 218, 122, 835, 2930, 279],
     [515, 22, 26, 248, 191, 334, 495, 299, 329, 239, 2684],
     [8375, 285, 8376, 369, 330, 604],
     [140, 7082, 4376, 4641, 36, 89, 14, 386, 157, 2205, 1495, 36, 81, 551],
     [4590, 521, 4591, 1564, 337, 1458, 14],
     [9145, 701, 381, 2939, 724, 36, 8, 5429, 807, 978, 979, 9145, 248, 34],
     [15, 133, 13, 302, 1264, 448, 1280, 2056, 379, 18, 3898, 1374],
     [1,
      1033,
      780,
      2791,
      2453,
      4242,
      669,
      2643,
      4222,
      109,
      74,
      13,
      206,
      2733,
      272],
     [471, 35, 43, 93, 1666, 84, 837, 109, 1565, 958, 1750, 109, 1565],
     [7074, 34, 485, 82, 227, 13, 53, 7075, 501, 502, 232],
     [162, 130, 85, 432, 199, 37, 332, 17, 111, 49],
     [3592, 125, 3593, 928, 23, 454, 17, 1313],
     [706, 433, 2176, 17, 241, 4748, 31, 2994],
     [6917, 295, 278, 6918, 6919, 202, 6920, 2, 6921, 87, 743, 893, 6922, 1614],
     [107, 591, 9, 356, 2303, 2304, 36, 8],
     [7199, 38, 215, 253, 114, 7200, 1401, 141, 3088, 35, 365, 85],
     [33, 11, 88, 1101, 5, 37, 299, 87],
     [125, 500, 8, 646, 418, 66, 93],
     [4373, 1, 42, 103, 1800, 1258, 2484, 192, 119, 22, 26],
     [5125, 1, 162, 130, 273, 39, 939, 190, 240],
     [5774, 1003, 902, 63, 607, 5775, 2420, 2681, 617, 5776],
     [556, 5016, 2656, 1289, 8771],
     [7498, 1294, 145, 895, 1502, 184, 897, 2],
     [1, 323, 430, 196, 92],
     [7580, 836, 11, 118, 120, 7581, 44, 7582, 603, 112, 7583],
     [4516,
      4,
      73,
      94,
      321,
      265,
      1000,
      364,
      223,
      47,
      94,
      571,
      2521,
      594,
      4517,
      2522,
      2523],
     [7644, 49, 112, 28, 98, 451, 2, 231, 62, 27, 112, 5],
     [7002, 1, 673, 3061, 343, 4, 7003, 2, 43],
     [7864, 20, 17, 489, 881, 2092, 2898, 1771],
     [6740, 174, 45, 677, 392, 682, 25, 168, 14, 34, 2432],
     [150,
      318,
      779,
      1365,
      102,
      43,
      3129,
      7621,
      318,
      779,
      154,
      1591,
      155,
      1466,
      46,
      860,
      1365,
      7622,
      2,
      386,
      318,
      2465],
     [8791, 532, 2824, 3153, 8792, 1141, 1524, 831, 194, 1587, 8793],
     [71, 7, 218, 412, 122, 598, 1417, 230, 71, 7, 4, 210, 598, 1252, 230, 71],
     [6127, 1, 6, 286, 156, 539, 887, 6, 880, 1043, 1737, 366, 619, 24],
     [5859, 1, 384, 458, 2841, 382, 407, 343, 287, 12, 2421, 5860, 624, 556],
     [9223, 147, 179, 810, 660, 20, 587, 3025, 2049, 713, 1205, 300],
     [5489,
      1,
      177,
      44,
      5490,
      1838,
      396,
      24,
      57,
      141,
      1014,
      854,
      38,
      2768,
      171,
      1953,
      903,
      5491,
      1261],
     [4685, 1856, 1469, 46, 472, 113, 113, 604, 1004, 1857, 954, 4686, 4687, 1148],
     [8397, 51, 749, 80, 8398, 354, 47, 652],
     [2288, 7, 30, 4, 133, 1335, 718, 148, 1119, 2125, 2289, 12],
     [30, 4, 77, 61, 736, 803, 8048, 965, 74, 420],
     [3622, 77, 61, 890, 2227, 191, 10, 1099, 988],
     [1, 162, 487, 1677, 5055, 796, 432, 955, 495, 32],
     [1, 1399, 1400, 381, 1225, 1411, 3187, 1765],
     [295, 2039, 670, 19, 295, 1528, 295],
     [8458, 139, 38, 582, 1245, 341, 25, 372, 675, 215],
     [488,
      20,
      144,
      8244,
      108,
      8245,
      120,
      6,
      8246,
      1358,
      160,
      2,
      374,
      8247,
      621,
      2,
      43],
     [5126, 301, 1098, 2, 1293, 19, 1319, 109, 361, 2, 111, 730],
     [6102, 101, 2, 48, 254, 137, 32, 2576, 6103, 2008],
     [8549, 31, 303, 60, 117, 2241, 111, 730, 324, 17, 125],
     [150, 3051, 3024, 2973, 2036, 133, 1068, 286, 156, 1138, 3013],
     [48, 50, 439, 19, 48, 137],
     [4940, 200, 810, 441, 2287, 45],
     [5856, 1212, 6, 6320, 3021],
     [5689,
      2806,
      1974,
      385,
      642,
      739,
      1494,
      2807,
      300,
      355,
      5690,
      5691,
      35,
      5692,
      1368,
      4,
      5693,
      1645],
     [1, 401, 2939, 3545, 177, 556, 556, 72, 1110],
     [6, 147, 6396, 368, 17, 371, 53, 136, 7928],
     [4934, 1056, 1267, 4935, 110, 4936, 13, 64, 4937, 1129, 743, 66],
     [5764, 4, 186, 5765, 52, 2823, 875, 15, 178, 3, 254, 999, 37],
     [8467, 18, 915, 488, 24, 1, 236, 1187, 362, 960, 8468, 8469, 8470, 562, 381],
     [9073, 74, 22, 26, 9074, 9075, 613, 20, 44, 148],
     [1725, 1021, 2310, 610, 1702, 37, 1397, 2],
     [453, 6, 286, 156, 1840, 7339],
     [3254, 1, 12, 110, 6, 1044, 215, 1253],
     [2685, 1, 1125, 1177, 211, 135, 54, 8020, 91],
     [296,
      602,
      7588,
      1510,
      98,
      866,
      1022,
      371,
      1205,
      7589,
      7590,
      1080,
      1282,
      197,
      995,
      7591,
      33,
      11],
     [5864, 1, 42, 344, 2227, 191, 291, 435, 1164, 38, 469, 486, 194],
     [820, 1199, 1507, 38, 113, 486, 194],
     [5457, 352, 182, 14, 1139, 167, 536, 5458, 336, 358, 248],
     [3587, 408, 3588, 90, 564, 468, 634, 411, 469, 1057, 3589, 3590],
     [20, 595, 24, 1, 222, 437, 3, 2042],
     [3236,
      1560,
      18,
      442,
      25,
      89,
      268,
      2109,
      890,
      304,
      891,
      2110,
      3237,
      1561,
      892],
     [28, 175, 2134, 5, 67, 763, 758, 269, 501, 502],
     [9071, 174, 9072, 1370, 5, 702, 319, 45, 102],
     [2868],
     [1178, 1, 832, 569, 60, 117, 78, 8121, 30, 4, 19, 444, 745],
     [7, 694, 512, 307, 616, 3148, 12],
     [4879, 1, 179, 4880, 211, 1451, 72, 46],
     [5111, 209, 6, 225, 332, 242, 34, 536, 1911],
     [251, 34, 7405, 53, 908, 2298, 908, 3402, 1010, 93],
     [662,
      999,
      5,
      10,
      185,
      734,
      560,
      78,
      1179,
      254,
      5,
      21,
      2505,
      999,
      138,
      192,
      1410,
      2506,
      2507,
      1180,
      4475],
     [1202, 825, 2614, 1881, 322, 2615, 15, 875, 2549, 4931, 15, 265, 182, 14],
     [827, 260, 229, 4711, 940, 10, 189, 13, 53, 232],
     [7878,
      1,
      546,
      3,
      141,
      2947,
      440,
      7879,
      7880,
      1356,
      1731,
      7881,
      1533,
      7882,
      997,
      512,
      73,
      7883,
      618],
     [9267,
      1727,
      2325,
      319,
      1434,
      2007,
      701,
      1147,
      701,
      94,
      39,
      530,
      1852,
      1024,
      9268,
      1727],
     [3981,
      34,
      3982,
      3983,
      133,
      59,
      732,
      762,
      11,
      393,
      3984,
      289,
      227,
      13,
      842,
      160,
      13],
     [8624, 20, 71, 651, 373, 23, 116],
     [6094, 6095, 439, 1927, 6096, 439, 98, 160, 14, 633],
     [6587, 1066, 2000, 1367, 145, 949, 522, 1657, 120, 454, 597, 120, 1657, 2239],
     [6820,
      7,
      76,
      515,
      1037,
      2152,
      55,
      67,
      313,
      199,
      124,
      1826,
      99,
      8,
      289,
      1826,
      99,
      8],
     [7349, 519, 447, 3097, 752, 342, 342, 2006, 699, 519, 677, 1128, 1540],
     [6888, 7, 4, 1043, 1076, 2843, 2844, 2845, 2846, 18, 565],
     [243, 11, 196, 1495],
     [142, 424, 584, 142, 814],
     [4188, 4189, 109, 665, 2142, 331, 563, 928, 459, 53, 178, 267, 23, 4190],
     [164, 164, 58, 2661, 784, 2, 466, 293, 5102, 1473, 14],
     [3607, 1, 1566, 19, 3608, 908, 796, 1641],
     [45, 8202, 10, 610, 869, 13, 8, 108, 3208, 1545],
     [38, 576, 2111, 9272, 9075],
     [6398, 518, 280, 273, 158, 172, 6399, 1520, 2967],
     [6890, 30, 2052, 4, 280, 1237, 3038, 649, 1539, 24, 580, 6891],
     [228, 2, 8724, 902, 1460, 251, 9, 111, 944, 1103, 251],
     [6260, 2933, 137, 1130],
     [402, 464, 432, 10, 5749, 2172, 425, 528, 47, 622, 7765],
     [6312, 14, 692, 52, 56, 2221, 1443, 6313, 1001, 85, 1443],
     [6855, 273, 648, 2009, 711, 883, 605, 118, 778],
     [8187, 1859, 1634, 8188, 374, 624, 171, 299, 336, 8189],
     [7952, 7, 31, 303, 94, 883, 2030, 1026, 7953, 498, 32],
     [8980,
      270,
      446,
      8981,
      243,
      8982,
      245,
      98,
      321,
      496,
      17,
      1548,
      3215,
      180,
      123],
     [8123,
      68,
      79,
      554,
      2,
      151,
      113,
      11,
      2728,
      1551,
      80,
      91,
      8,
      18,
      349,
      37,
      228,
      2],
     [664, 14, 58, 72, 46, 20, 22, 26, 552, 71, 5, 404],
     [7, 4, 133, 40, 212, 246, 301, 153, 3204, 320],
     [318, 126, 92, 31, 318, 4970, 2629],
     [6080,
      1,
      57,
      128,
      326,
      2232,
      20,
      17,
      876,
      991,
      1115,
      2732,
      81,
      1436,
      1689,
      327],
     [8349, 1223, 1188, 275, 908, 1521, 26],
     [5175, 58, 1079, 2, 125, 17],
     [9258, 716, 1906, 211, 1451, 54],
     [3712, 890, 948, 2285, 263, 1117, 2286, 509, 19],
     [7692, 247, 167, 7693, 2091, 1140, 15, 1929, 137],
     [3092, 1296, 432, 202, 78, 628, 1690, 91, 254, 2709],
     [3575, 701, 12, 2241, 625, 2242, 3576, 3577],
     [9152, 1, 555, 414, 1688, 72, 46, 2776, 1411, 273, 907, 91],
     [11, 865, 443, 6485, 1159, 134, 1754, 476, 2589, 56, 21, 101],
     [378, 239, 2182, 332, 618],
     [6171, 15, 21, 950, 7, 76, 5, 6172, 6173],
     [96, 1, 4, 73, 143, 367, 64, 801, 1197, 2592, 4839, 106],
     [8738,
      3212,
      200,
      8739,
      263,
      403,
      3102,
      8740,
      45,
      3213,
      3213,
      534,
      8741,
      1875,
      8742,
      2072,
      3212,
      513,
      1174],
     [127, 1429, 2665, 207, 9187, 160, 45, 10, 1227, 427, 2397, 206],
     [2316, 6, 59, 429, 967, 292, 7210, 699],
     [8131,
      424,
      22,
      26,
      1519,
      751,
      53,
      783,
      255,
      2080,
      540,
      138,
      9,
      816,
      597,
      80,
      469],
     [5019, 328, 63, 15, 1205, 418, 594, 1735],
     [8031,
      1,
      720,
      1257,
      117,
      21,
      8032,
      35,
      8033,
      2120,
      780,
      3171,
      21,
      10,
      8034,
      8035,
      2265,
      2266,
      188,
      2237],
     [6026, 30, 4, 125, 180, 6027, 34, 303, 7, 33, 11],
     [8277,
      1,
      18,
      349,
      82,
      1061,
      15,
      8278,
      8279,
      57,
      468,
      634,
      18,
      349,
      8280,
      709,
      66,
      1404,
      145,
      227,
      3003,
      364],
     [1, 2737, 788, 363, 33, 11, 3173],
     [417, 417, 628, 262, 379, 6688, 159, 221, 93],
     [8855, 188, 351, 289, 101, 34, 4, 133, 59],
     [2023, 1698, 21, 322, 650, 389, 10, 38, 7414],
     [1545, 1, 1736, 39, 643, 316, 516, 74, 1540, 1134, 3],
     [8251, 573, 8252, 437, 32, 289, 1203],
     [325, 231, 322, 451, 2, 15, 66],
     [4131, 4132, 50, 256, 917, 52, 411, 52, 221, 787, 1145],
     [8861, 19, 32, 125, 10, 1777, 8862, 87],
     [4808, 105, 4809, 1799, 33, 11, 709, 87, 284],
     [4426, 3462, 2709, 846, 2, 552],
     [4, 73, 559, 91, 17, 1313],
     [7962, 129, 101, 2, 465, 109],
     [8148, 723, 1069, 442, 225, 3060, 66, 640, 1532, 9, 729, 2243, 102, 87],
     [7474,
      1,
      2692,
      147,
      7475,
      7476,
      887,
      1845,
      179,
      2729,
      3107,
      197,
      7477,
      114,
      3108,
      7478,
      2874,
      104,
      7479,
      7480,
      7481,
      1483,
      541,
      7482,
      937,
      3109],
     [7188, 19, 41, 699, 18, 1380, 524, 198, 109, 12, 116, 7189],
     [4402, 384, 458, 16, 2491, 7, 4, 2492, 1323, 1804, 195, 1805, 1806, 9],
     [8069,
      813,
      3114,
      91,
      210,
      193,
      124,
      249,
      8070,
      1472,
      193,
      8071,
      56,
      223,
      94,
      39,
      8072],
     [12, 60, 8647, 378, 239, 688],
     [3491, 9, 120, 628, 459, 516, 629, 107, 18, 100, 492, 360, 41, 2],
     [8951, 820, 8952, 3120, 3183, 2214, 1093, 8953, 231, 8954],
     [6775, 36, 154, 14, 3032, 9, 375, 807, 1689, 327],
     [1, 4, 22, 248, 24, 8039, 138, 553, 636, 80],
     [4826, 1, 1714, 561, 1874, 4827, 12, 4828, 476, 2589, 125],
     [618, 313, 270, 860, 387, 17, 914, 1004, 311],
     [6408, 1, 147, 521, 6409, 166, 21, 550, 257, 22, 26, 2398, 6410, 119],
     [3273, 1, 2119, 306, 497, 57, 491, 3274, 161, 2120, 1565, 1257, 117, 349, 93],
     [4530, 51, 593, 5, 86, 63, 93],
     [843, 1380, 348, 32, 882, 205, 1401, 8012, 219, 14, 677],
     [7031, 127, 200, 491, 757, 1228, 796, 252, 28],
     [4902, 701, 549, 56, 474, 202, 153, 448, 36, 306, 325, 1450, 114, 773],
     [1430, 292, 621, 2, 394, 134, 204, 2698, 829, 123],
     [8462, 1538, 2027, 2219, 391, 482, 1883, 732, 172, 3119],
     [3348, 128, 42, 133, 1061, 313, 3349, 379, 506, 558, 3350],
     [6471, 40, 659, 2985, 177, 1802, 10, 195],
     [3199,
      36,
      340,
      66,
      44,
      1235,
      773,
      23,
      43,
      133,
      59,
      148,
      41,
      230,
      1304,
      7582,
      1142,
      230,
      265,
      340],
     [5035, 7, 5036, 5037, 2644, 83, 5038],
     [9105, 6, 158, 851, 172, 9106, 9107, 9108, 1732, 501, 685],
     [5365,
      127,
      75,
      158,
      90,
      1943,
      2400,
      1775,
      658,
      1498,
      179,
      1922,
      2688,
      113,
      5366,
      2730,
      421,
      1944,
      2731,
      2732,
      5367],
     [539, 165, 1159, 661, 1856, 7068, 20, 977, 648, 306],
     [6202, 358, 424, 6203, 6204, 2919, 1140, 1420, 15],
     [277, 669, 1967, 16, 35, 919, 159, 433, 8122, 513, 3042, 277],
     [8305, 1, 8306, 8307, 167, 693, 1989, 1944, 723, 32, 8308, 7, 30, 4, 8309],
     [150, 2567, 121, 3024, 25, 315, 116, 1572, 43, 2036],
     [5029, 1, 128, 1207, 67, 19, 49, 169, 48, 137, 168],
     [8327,
      1,
      179,
      3193,
      273,
      8328,
      6,
      165,
      140,
      10,
      74,
      139,
      137,
      41,
      23,
      8,
      815,
      138,
      339,
      728],
     [7499, 323, 430, 44, 635, 10, 907, 54],
     [3392,
      254,
      50,
      237,
      917,
      2166,
      405,
      334,
      1285,
      83,
      69,
      114,
      334,
      1285,
      918,
      2167,
      156],
     [6077,
      179,
      6078,
      817,
      244,
      64,
      123,
      60,
      233,
      742,
      287,
      141,
      2528,
      1445,
      1446],
     [5548, 63, 515, 144, 5549, 382, 2, 84, 428, 1957, 1134, 648],
     [3365, 1279, 1280, 773, 889, 1281, 1282, 2155, 1280],
     [5945, 1, 963, 1412, 785, 1103, 5946, 19],
     [2571, 2, 662, 337, 636],
     [4328, 4329, 1792, 1133, 1360, 1793, 297, 4330, 15, 285, 332],
     [275, 1, 77, 61, 31, 275, 2478, 1941, 5423, 31, 5424, 94],
     [3611,
      7,
      30,
      76,
      3612,
      1642,
      1643,
      1318,
      2252,
      2253,
      258,
      2254,
      97,
      3613,
      2255],
     [4248,
      1,
      175,
      596,
      344,
      2181,
      2456,
      23,
      96,
      24,
      713,
      1271,
      2457,
      129,
      2458,
      516,
      53,
      28,
      1345,
      96,
      21,
      199,
      656,
      516,
      28],
     [8135, 8136, 8137, 8138, 115, 533, 132, 3018, 852, 983, 2098, 656],
     [1736, 705, 5467, 22, 26, 24, 2353, 22, 26, 24, 2050],
     [3715, 1, 218, 3716, 12, 81, 129, 16, 7, 30, 76, 15, 21],
     [9, 145, 133, 59, 145],
     [12, 381, 1408, 4348, 94, 453, 1829, 3212, 10, 347, 708],
     [249, 24, 578, 553, 51, 848, 800],
     [222, 2950, 200, 1415, 222, 278, 824, 242, 504, 168, 14],
     [9127, 30, 76, 1739, 475, 785, 1, 262, 339, 121, 9128, 9129],
     [4209, 1399, 1400, 273, 205, 28, 4210, 27, 4211, 4212],
     [5612,
      2393,
      61,
      1962,
      360,
      1380,
      961,
      167,
      55,
      5613,
      360,
      1168,
      961,
      167,
      5614,
      5615],
     [34, 20, 400, 38, 97, 1198, 2600, 880, 580, 400, 2581, 1387, 2],
     [6048,
      1,
      20,
      24,
      578,
      421,
      49,
      2003,
      1318,
      116,
      2252,
      2253,
      817,
      6049,
      1525,
      219,
      2004,
      2003],
     [4954, 82, 142, 1087, 6, 4955],
     [5896,
      1574,
      1186,
      83,
      14,
      631,
      1516,
      620,
      36,
      83,
      14,
      386,
      5897,
      645,
      252,
      5,
      40],
     [6045, 33, 11, 144, 6046, 6047, 883, 638, 944, 6],
     [54, 1, 175, 716, 1112, 31, 10, 49, 54],
     [574, 5947, 152, 1891, 2255, 1477, 5948, 7, 485],
     [1, 175, 596, 344, 16, 1152, 63, 296, 1156, 86, 296, 28],
     [6944, 1, 1699, 1700, 16, 1111, 10, 2770, 18, 84, 296, 5],
     [4152, 292, 481, 2428, 132, 297, 32, 134, 400, 4153],
     [7968, 314, 424, 30, 4, 129, 247, 263, 1117, 22, 26, 53],
     [4240,
      1097,
      1422,
      4241,
      1691,
      2452,
      2163,
      775,
      3,
      175,
      2453,
      4242,
      31,
      199,
      496,
      38,
      2454,
      2452,
      2454,
      186,
      2455,
      4243,
      524],
     [5405,
      197,
      1473,
      23,
      404,
      202,
      89,
      14,
      5406,
      1504,
      1102,
      78,
      322,
      5407,
      1102,
      15],
     [840, 311, 496, 70],
     [4641, 1030, 630, 13, 15, 217, 19, 1476, 709, 32, 178, 1842, 15],
     [4953, 22, 248, 191, 292, 134, 36, 8, 167, 1794, 1897, 255, 272, 435],
     [8966, 19, 886, 8, 40, 8967, 1417, 11, 284],
     [1979, 3064, 184, 2, 34, 19, 134, 1850, 24],
     [345, 2, 38, 2768, 171, 171, 4575, 7375],
     [9140, 1, 42, 103, 273, 305, 700, 9141, 486, 194, 287, 9142],
     [75, 2328, 701, 368, 2595, 137, 45, 6196],
     [360, 6, 913, 655, 549, 70, 287, 7, 1432, 1217, 374, 1265, 1323, 277],
     [7993, 7994, 641, 7995, 510, 615, 293, 1772, 8, 7996, 702, 72, 46, 2779],
     [1102, 687, 559, 606, 2214],
     [644, 1033, 1072, 72, 46, 399, 2391, 1072, 152],
     [1957, 5268, 7, 4, 1024, 883, 624, 419, 883, 132],
     [6682, 9, 68, 79, 62, 176, 871, 389],
     [147, 323, 1419, 54, 67, 2042, 3182, 238, 80, 131, 702, 272, 654, 1121, 148],
     [8207, 520, 256, 824, 254, 50, 472, 124, 173, 642, 1031],
     [4581, 47, 633, 1845, 4582, 4583, 150, 1457, 7],
     [5762, 838, 85, 135, 5763, 170, 544, 10, 196, 665, 151, 643, 238, 456, 2116],
     [213, 39, 4496, 485, 9, 942, 571, 241, 132],
     [8566, 1812, 2, 129, 123, 636, 26, 97],
     [25, 41, 143, 503, 610, 261, 107, 666, 222, 272, 796],
     [20, 17, 228, 230, 813, 27, 394, 1442],
     [426, 1, 1056, 1267, 713, 52, 2797, 726, 3461, 4142],
     [191, 139, 56, 119, 705, 498, 78, 1795, 3, 3409, 238, 137],
     [121,
      896,
      228,
      13,
      15,
      36,
      288,
      364,
      122,
      73,
      2017,
      107,
      2502,
      238,
      456,
      15,
      35,
      288,
      364,
      8],
     [6578, 6579, 177, 172, 429, 88, 659, 530, 482, 12, 448, 333],
     [158, 1219, 114, 786, 166, 18, 825, 1025, 448, 2708, 40, 1404],
     [3786, 36, 707, 364, 231, 217, 228, 13, 15, 36, 8, 217, 15, 34, 890],
     [5007, 173, 587, 587, 33, 11, 472, 1863, 552, 278, 124],
     [93, 56, 474, 115, 134, 2364, 8610, 2597, 155, 3068],
     [6345, 1, 202, 55, 313, 100, 160, 2, 2950, 2951, 85],
     [5679, 263, 235, 6, 572, 13, 376, 1248, 1797, 492, 2798, 889, 940],
     [7352,
      1,
      7353,
      1039,
      695,
      2949,
      122,
      5,
      135,
      10,
      680,
      7354,
      13,
      15,
      592,
      7355,
      13,
      15,
      466],
     [3387, 7, 512, 202, 71, 1594, 382, 69, 218, 412, 122, 314, 69],
     [3555,
      387,
      179,
      3556,
      3557,
      55,
      63,
      632,
      707,
      2,
      100,
      20,
      28,
      3558,
      572,
      53,
      27,
      410,
      2,
      100,
      20,
      28],
     [7404,
      502,
      7405,
      7406,
      7407,
      475,
      7408,
      7409,
      7410,
      12,
      155,
      502,
      7411,
      7412,
      180,
      35,
      7413,
      7414,
      224,
      3076,
      3077],
     [6411, 6, 440, 1153, 6412, 6413, 360, 41, 2, 196, 90, 520, 2, 28, 36, 8],
     [3558,
      630,
      53,
      255,
      1032,
      722,
      23,
      612,
      60,
      44,
      10,
      529,
      53,
      3208,
      12,
      3127,
      148,
      1119],
     [6654, 1, 3016, 200, 128, 6655, 6656, 52, 56, 474, 191, 134, 56, 6657, 699],
     [1, 19, 756, 573, 20, 58, 632, 28],
     [7942, 900, 1175, 409, 124, 48, 312],
     [1, 1443, 227, 13, 1702, 592, 13, 1702, 265, 168, 14, 1242, 2355, 45, 127],
     [3486,
      2206,
      75,
      2207,
      703,
      3487,
      704,
      516,
      2208,
      3488,
      2209,
      3489,
      1090,
      1040,
      1091,
      2210,
      13,
      168,
      13],
     [8364, 1, 19, 413, 427, 2352, 1726, 8365],
     [511, 4, 73, 4788, 221, 4789, 2533],
     [801,
      959,
      452,
      139,
      2028,
      5419,
      34,
      1209,
      5,
      530,
      1241,
      773,
      647,
      1876,
      2570,
      2569,
      8387,
      41],
     [7209, 7210, 1288, 2145, 187, 1894, 184, 13, 18, 238, 732, 45],
     [6664, 1033, 44, 67, 10, 49, 82, 13, 15],
     [198,
      288,
      20,
      165,
      44,
      394,
      1646,
      1556,
      1635,
      10,
      305,
      2442,
      1631,
      988,
      452,
      20,
      575,
      5348,
      2088],
     [3819, 56, 474, 184, 427, 15, 394, 134, 107, 829],
     [5387,
      5388,
      5389,
      1841,
      19,
      31,
      2736,
      5390,
      5391,
      5392,
      43,
      708,
      291,
      265,
      1348,
      5393,
      5394],
     [8332, 290, 66, 32, 8333, 8334, 1729, 102, 87],
     [4928, 301, 4929, 8, 159, 417, 63, 1483],
     [34, 128, 326, 893, 85, 19, 289, 4410, 32, 225, 5517, 254, 50],
     [5694, 1, 5695, 5696, 5697, 114, 261, 2, 45, 1374],
     [3477, 770, 90, 3478, 1088, 35, 43, 185, 626, 3479, 2204, 81, 3480, 443],
     [1445, 1446, 6842, 38, 1020, 478, 2046, 811, 6176, 974],
     [9171, 715, 506, 296],
     [3414, 166, 357, 207, 250, 52, 62, 39, 3415, 39, 2180],
     [10, 243, 9, 3445, 1972, 2021, 48, 22, 26, 2513],
     [8190,
      2746,
      8191,
      97,
      10,
      408,
      809,
      528,
      329,
      892,
      605,
      214,
      8192,
      1232,
      972,
      2494,
      1806,
      437,
      3,
      2042],
     [3950, 1, 3951, 59, 645, 761, 13, 927, 272, 1724, 48, 2359],
     [273, 1959, 4552, 100, 5],
     [4206, 286, 156, 213, 4207, 81, 4208, 17],
     [4930, 535, 168, 874, 595, 391],
     [6942, 432, 48, 137, 1531, 256, 224, 6943, 464, 389],
     [70, 2755, 6636, 301, 466, 8577, 2353, 250, 6473, 3804],
     [6858, 288, 9, 52, 708, 390, 809, 164, 25, 116],
     [8662, 292, 10, 735, 201, 740, 13, 1531, 8663, 146, 8664],
     [197, 1001, 49, 150, 142],
     [8379, 342, 333, 262, 40, 58, 1194, 8380, 863, 63, 471],
     [409, 123],
     [974, 101, 411, 149, 6, 2360, 656, 856, 95],
     [9097,
      1,
      3168,
      145,
      37,
      9098,
      1929,
      9099,
      688,
      95,
      460,
      691,
      2483,
      2059,
      683,
      280],
     [5230,
      57,
      42,
      103,
      1414,
      493,
      2698,
      692,
      42,
      2873,
      3,
      449,
      450,
      144,
      189,
      962,
      3,
      44],
     [204, 8468, 838, 118, 120, 21, 10, 334, 2343, 1701, 711, 8515, 331],
     [1360,
      407,
      2844,
      4984,
      5237,
      111,
      7896,
      72,
      1110,
      1176,
      150,
      893,
      2201,
      1385,
      72,
      1110],
     [8893,
      118,
      12,
      88,
      95,
      332,
      8894,
      8895,
      8896,
      210,
      598,
      15,
      1150,
      8897,
      563,
      3205,
      1668,
      456,
      418,
      8898,
      15,
      6,
      165,
      683,
      1039,
      418,
      456,
      242,
      55,
      629,
      242,
      2294,
      638,
      1563,
      260,
      8899,
      229,
      15],
     [3943, 57, 42, 103, 1607, 621, 18, 3944, 2354, 3, 965, 62, 963],
     [3717,
      1,
      134,
      444,
      1317,
      287,
      3718,
      1118,
      3719,
      1301,
      46,
      949,
      1668,
      2287,
      443,
      19],
     [9070, 1696, 820, 55, 20, 348, 411, 405, 170, 55, 238, 456, 348],
     [7333, 422, 336, 2162, 7334, 146, 319, 22, 26, 1240, 750, 13, 53],
     [36, 110, 424, 124, 208, 739, 601, 110, 424, 2891, 3325],
     [98, 35, 43, 2771, 14, 744, 127, 921, 754],
     [3332, 556, 2145, 889, 19, 1578, 1579, 3333],
     [700, 1298, 20, 17, 878],
     [1, 42, 103, 4492, 573, 987, 849, 236, 833, 3197, 199, 172],
     [32, 2432, 70, 1335, 6192, 1318, 98, 686],
     [8315, 8316, 75, 201, 50, 144, 182, 13, 25, 8, 8317, 274, 245],
     [5422, 92, 166, 2746, 161, 5, 86, 63, 93],
     [3181, 9032, 310, 10, 10, 197, 151, 2881, 418, 2229, 2386],
     [5632, 1964, 5633, 5634, 343, 145, 80, 343, 585, 82, 1418, 139, 15],
     [141, 16, 3933, 89, 230],
     [7389,
      1,
      568,
      989,
      3041,
      402,
      7390,
      10,
      522,
      151,
      235,
      7391,
      515,
      450,
      316,
      104,
      57,
      103],
     [4165, 706, 1414, 593, 1162, 292, 395, 153],
     [7084, 176, 205, 3072, 5, 278, 180, 115, 1470],
     [605, 375, 2047, 53, 694, 512, 247, 199, 146, 2894, 974],
     [56,
      129,
      199,
      656,
      43,
      8741,
      1151,
      305,
      105,
      506,
      635,
      268,
      686,
      3,
      2667,
      105,
      506],
     [5067, 5068, 2315, 294, 110, 69, 1905, 295, 294, 5069, 5070, 5071, 295],
     [7524, 19, 129, 496, 1619, 72, 46, 1619, 1223, 22, 26],
     [4412, 532, 204, 4413, 122, 73, 975, 983, 136, 1808, 1809, 239],
     [3481,
      627,
      3482,
      2205,
      293,
      36,
      282,
      18,
      1089,
      379,
      1617,
      1256,
      242,
      182,
      2,
      319,
      6,
      270,
      2170],
     [4344,
      1,
      1439,
      531,
      2475,
      2476,
      211,
      506,
      715,
      1320,
      16,
      7,
      76,
      850,
      624,
      1321,
      1440,
      378,
      239],
     [7423, 7, 117, 4, 542, 2937, 2809, 7424, 262, 1548, 40, 153, 534],
     [302, 3, 160, 11, 6996, 1785, 27, 267, 357, 316, 108, 192],
     [1,
      126,
      92,
      2401,
      1783,
      2044,
      20,
      448,
      2107,
      189,
      325,
      231,
      47,
      811,
      268,
      1857,
      92,
      31,
      1580,
      2680],
     [5875,
      987,
      1163,
      2440,
      359,
      217,
      22,
      26,
      248,
      24,
      104,
      54,
      519,
      371,
      49,
      54,
      541,
      27,
      18,
      17,
      503],
     [5245, 2198, 1423, 2701, 816, 41, 5246, 25, 8, 157, 9, 94, 2701, 94],
     [9198, 21, 483, 3146, 124, 156, 403, 449],
     [7, 4, 31, 37, 10, 134, 96, 2324, 2],
     [218, 747, 2519, 65, 730, 593, 1235, 93, 33, 11, 593, 148, 93, 33, 11],
     [3988, 6, 480, 2372, 58, 385, 1121, 336, 3989, 400, 184, 2, 3990, 221],
     [5718, 402, 70, 884, 884, 714, 2812],
     [8085, 2059, 875, 1129, 210, 3005, 1655, 321, 1000, 2332, 13, 64, 1039],
     [7395, 9, 511, 216, 560],
     [1,
      934,
      935,
      2578,
      21,
      10,
      529,
      992,
      22,
      26,
      151,
      1517,
      2543,
      287,
      224,
      119,
      453,
      464,
      4044,
      209,
      2163,
      9051],
     [119,
      825,
      41,
      6738,
      820,
      1394,
      5736,
      66,
      267,
      773,
      1206,
      998,
      5736,
      825,
      25,
      312],
     [8297, 886, 8, 8298, 8299, 509, 90, 158, 8300, 302, 22, 67],
     [1326, 1555, 538, 78, 419, 1649, 658, 286, 156, 1881, 4045],
     [5550, 701, 5551, 111, 862, 682, 17],
     [3573, 1634, 791, 65, 328, 16, 1635, 557, 792, 226, 8, 36, 8],
     [7933, 1, 3, 286, 156, 117, 669, 126, 405, 152, 2143, 539, 3160, 3161],
     [5130,
      12,
      25,
      83,
      237,
      2666,
      582,
      254,
      50,
      2216,
      319,
      443,
      437,
      66,
      142,
      256,
      62,
      1285],
     [4829, 4830, 4831, 77, 61, 2590, 2442, 1631, 988, 852],
     [3609, 18, 173, 797, 3610, 704, 1062, 411, 173, 140, 636, 29, 354],
     [1, 2722, 6638, 211, 171, 1075, 793, 577],
     [259,
      312,
      8387,
      667,
      7829,
      366,
      1552,
      255,
      1464,
      2,
      573,
      583,
      1666,
      8068,
      308,
      6847,
      8006],
     [8641,
      57,
      128,
      326,
      143,
      13,
      64,
      481,
      1162,
      1808,
      20,
      40,
      1199,
      1546,
      56,
      272,
      158,
      865,
      312],
     [6085,
      276,
      178,
      6086,
      15,
      614,
      911,
      197,
      2894,
      6087,
      35,
      1060,
      6088,
      6089,
      36,
      8,
      6090],
     [8877, 1, 3149, 790, 16, 1001, 71, 296, 28, 96, 196, 178, 168, 13, 15],
     [8159, 1, 323, 1419, 44, 67, 3176, 54],
     [7977, 1, 18, 1687, 7978, 252, 5, 32],
     [1, 778, 117, 413, 23, 18, 58, 3, 483],
     [2, 53, 98, 496, 30, 4, 196, 465],
     [7470, 7471, 7472, 273, 86, 28, 325, 112, 95, 3106],
     [7564,
      21,
      2054,
      642,
      2008,
      683,
      116,
      10,
      1850,
      50,
      885,
      7565,
      261,
      754,
      668,
      50,
      8],
     [2, 256, 347, 232, 3409, 48, 137],
     [7889, 141, 3088, 217, 7890, 18, 15, 85],
     [4163, 1, 19, 462, 2356, 1161, 123, 4164, 732, 140, 3],
     [3660, 1329, 2269, 3661, 84, 1110, 3662, 249, 1653, 807, 36, 110, 364],
     [7428, 3103, 832, 1139, 36, 8, 451, 2, 2344, 62, 176, 40],
     [7682, 42, 7683, 395, 896, 2085, 228, 611, 373, 1559, 2161, 1406],
     [6495, 44, 2993, 165, 140, 310, 13],
     [35, 43, 320, 296, 987],
     [36, 8, 2520, 82, 1473, 13, 25, 940],
     [1740, 161, 80, 32, 9047, 552, 4575, 583, 80],
     [7204, 384, 458, 16, 169, 22, 2078, 749, 3047, 26, 48, 22, 1181, 1201],
     [223, 665, 959, 665, 214, 959],
     [3633, 16, 2388, 809, 603, 3068, 329, 892],
     [8383, 3165, 1461, 825, 1196, 1681, 8384, 1423],
     [4914, 619, 63, 215, 442, 1099, 4915, 2609, 4916, 4917],
     [4452, 1, 1447, 67, 4453, 1448, 175, 673, 4454],
     [7101, 192, 2562, 7102, 45, 174, 354, 1100, 7103, 124, 7104, 1055, 1409],
     [7787, 162, 487, 2365, 338, 105, 131, 391, 802, 1027, 425, 5, 1038],
     [6235, 673, 1987, 16, 82, 41, 230, 18, 28, 525, 230, 18, 71],
     [4918, 3, 17, 58, 86, 4919, 2, 8],
     [3911,
      1,
      247,
      35,
      166,
      3912,
      17,
      1710,
      351,
      839,
      588,
      54,
      399,
      269,
      2346,
      24],
     [7767, 7768, 1032, 3144, 1170, 950, 7769, 679, 1064, 5, 86],
     [4507,
      2518,
      163,
      1380,
      348,
      32,
      253,
      688,
      484,
      253,
      2225,
      55,
      348,
      253,
      253,
      4508],
     [33, 221, 2100, 2064, 10, 107, 280, 6, 322, 1969, 132, 2939, 132, 6213],
     [8389, 9, 708, 357, 8390, 993, 2022, 1161, 52],
     [100, 260, 229, 1150, 229, 224, 9],
     [8794, 1, 171, 992, 2101, 3, 3155, 509, 2101],
     [692, 1029, 182, 14, 152, 20, 955, 495, 22, 80],
     [4658, 264, 267, 833, 8, 2459, 1424, 200, 441],
     [4662, 4663, 4664, 1, 2559, 2560, 16, 4665, 2, 43, 370, 233],
     [52, 4629, 73, 4630, 114, 761, 48, 655, 1464, 364],
     [5353,
      2559,
      2560,
      41,
      52,
      550,
      1204,
      813,
      433,
      5,
      5354,
      10,
      385,
      27,
      1331,
      22,
      26],
     [20,
      8967,
      1159,
      477,
      7717,
      2,
      1104,
      3159,
      351,
      2595,
      2,
      1104,
      351,
      422,
      2,
      6389,
      351,
      36,
      8],
     [5150, 1269, 2, 1381, 258, 176, 894, 609, 648, 1178, 206, 994, 1134],
     [6935,
      35,
      43,
      320,
      1649,
      623,
      3047,
      1220,
      1521,
      942,
      2837,
      2848,
      6936,
      39,
      189,
      963],
     [7570,
      323,
      430,
      193,
      642,
      1191,
      224,
      7571,
      173,
      216,
      7572,
      141,
      2460,
      44,
      1251],
     [1,
      6,
      65,
      1648,
      934,
      935,
      211,
      5,
      86,
      7,
      4,
      1,
      10,
      74,
      8932,
      192,
      1122,
      99,
      8],
     [6615, 10, 35, 60, 440, 365, 18, 349, 869, 14],
     [838, 350, 204, 3, 4477, 970, 1215, 116, 2673, 2907, 378, 239, 3, 43],
     [398, 1, 398, 185, 257, 3001, 40, 439, 856, 236, 421, 2991, 1191],
     [7, 122, 115, 12, 1919, 18, 565, 69, 353, 1234],
     [5059, 2650, 1487, 900, 270, 2651, 330, 12, 603, 2448, 2652, 1339],
     [4854, 29, 12, 75, 1407, 180, 2450, 2563, 343, 2202, 599, 2593],
     [3316, 1, 768, 3317, 3318, 2136, 307, 2137, 1268, 189, 1057],
     [9112,
      128,
      326,
      190,
      1310,
      483,
      500,
      2336,
      179,
      668,
      9113,
      9114,
      1391,
      921,
      1033,
      1218,
      663,
      1980],
     [5416, 5417, 5418, 207, 5419, 255, 5420, 400, 5421, 1193, 47, 434, 901],
     [8716, 441, 1185, 1242, 12, 2095, 32],
     [836, 71, 36, 81, 14, 71, 7, 424, 284],
     [7557, 468, 7558, 7559, 366, 1127, 3123, 7560, 943],
     [647, 255, 68, 79, 227, 5586, 36, 1664],
     [6937,
      1,
      3048,
      3049,
      1749,
      31,
      10,
      16,
      1316,
      323,
      430,
      44,
      67,
      31,
      6938,
      10,
      109,
      782,
      6939,
      736,
      1068,
      1316],
     [5617, 3, 44, 635, 5618, 43, 5619, 85, 266, 81, 551],
     [8103, 8104, 786, 8105, 1721, 1008, 19],
     [6776, 2057, 1165, 603, 43, 21, 353, 33, 11],
     [4046,
      30,
      4,
      975,
      4047,
      1739,
      475,
      4048,
      4049,
      4050,
      31,
      976,
      368,
      53,
      1740,
      4051,
      2393],
     [1, 193, 398, 1270, 89, 13, 1119, 627, 22, 316, 1060, 1407, 75, 781, 29],
     [4884, 4885, 2117, 1375, 1284, 1806],
     [7762, 1, 153, 1547, 950, 593, 732, 18, 7763, 15, 34, 87, 159, 2551],
     [3547, 68, 79, 3548, 450, 67, 419, 35, 3, 102],
     [7770, 21, 1208, 1873, 1068, 2650, 1487, 900, 270, 30, 4],
     [5959, 2189, 2865, 32],
     [4638, 7, 30, 4, 253, 1345, 1088, 585, 2853, 2854, 2855, 301, 592],
     [1, 7, 30, 4, 660, 264, 83, 63, 733, 2801, 7192],
     [2348, 1598, 2107, 81, 2348, 1622, 1341, 1900, 2903, 2377, 683],
     [5020, 1, 781, 541, 1283, 224, 1206, 5021, 2638, 2639, 903],
     [8536, 1, 2653, 1230, 1290, 1291, 2938, 2101, 437, 327],
     [56,
      474,
      2666,
      3524,
      8932,
      5,
      135,
      1842,
      8932,
      68,
      79,
      192,
      723,
      20,
      100,
      28,
      86],
     [906, 2, 100, 2041, 36, 14, 289, 360, 41, 2],
     [444, 953, 11, 65, 6594, 146, 114, 89, 2, 4483],
     [7439, 95, 460, 36, 83, 364, 217, 15, 8, 353, 122, 1234],
     [1, 364, 425, 1713, 74, 3, 116, 2290, 2220, 769, 1909],
     [3886, 22, 80, 131, 27, 1364, 928, 13, 8],
     [8478, 25, 7, 34, 1993, 8479, 1994, 2260, 259, 881],
     [243, 53],
     [1, 57, 468, 634, 829, 18, 349, 95, 332, 413, 14],
     [8265, 1, 8266, 8267, 16, 172, 3188, 3189, 3190],
     [9056, 9057, 562, 151, 27, 20, 100, 28, 581, 294, 20, 100, 28, 126, 92, 632],
     [3946,
      34,
      33,
      11,
      1289,
      2355,
      279,
      180,
      965,
      35,
      8,
      142,
      279,
      1378,
      318,
      91,
      36,
      340],
     [4881, 1, 1883, 2586, 853, 195, 536, 744, 244, 9, 724, 356, 15, 1874, 548],
     [7861, 18, 24, 992, 3155, 7862, 509, 78, 2081, 38, 62, 1013, 7863, 26],
     [5788, 260, 184, 2, 2443, 214, 2830, 330, 5789, 119, 390],
     [7681, 1387, 427, 2296, 85, 2580, 202, 28, 501, 100, 192],
     [4739, 130, 1661, 1340, 2512, 4740, 51],
     [3602, 3603, 39, 9, 1315, 109, 1316, 1105, 1317, 2246, 183, 1556],
     [7435, 231, 302, 7436, 208, 231, 390, 24],
     [8933, 155, 70, 333, 754, 27, 450, 1060, 12, 604],
     [1,
      724,
      358,
      483,
      962,
      58,
      952,
      330,
      3240,
      604,
      2413,
      116,
      172,
      7413,
      101,
      23],
     [5906, 1, 101, 751, 434, 6, 38, 160, 375, 5907, 580, 2277],
     [3843,
      1693,
      3844,
      58,
      459,
      95,
      114,
      43,
      192,
      1694,
      186,
      958,
      958,
      3845,
      380,
      294,
      188,
      1132,
      1055,
      2330],
     [767, 940, 4105, 816, 239, 2408, 1402, 43, 4106, 1749, 4107, 22, 2409],
     [5569, 35, 5570, 395, 85, 19],
     [1, 25, 1326, 337, 3, 264, 12, 3958, 186],
     [5537, 838, 595, 5538, 5539, 374, 796, 11, 2548, 6, 293, 1827, 1023],
     [8506, 1030, 510, 615, 1356, 7, 1804, 1323, 1957, 193, 8507],
     [6848, 68, 79, 6849, 6850, 6851, 84, 1115, 33, 11],
     [7923,
      7924,
      7925,
      7926,
      493,
      264,
      83,
      427,
      1237,
      437,
      32,
      69,
      2108,
      47,
      7927,
      7928,
      1224,
      1945],
     [4961,
      1702,
      4962,
      385,
      172,
      2625,
      327,
      390,
      4963,
      1824,
      167,
      2626,
      4964,
      4965,
      4966],
     [232, 1, 157, 53, 769, 232, 796, 88, 388, 34],
     [1, 90, 313, 665, 91, 1983, 154, 221, 324, 87],
     [8067, 183, 591, 9, 216, 165, 663, 180, 41, 449, 25, 219, 8068],
     [3284,
      1,
      350,
      204,
      1568,
      1569,
      549,
      21,
      1052,
      407,
      377,
      2123,
      2124,
      1570,
      1261],
     [8658,
      1,
      244,
      8659,
      8660,
      126,
      92,
      8661,
      1003,
      607,
      1003,
      1343,
      176,
      536,
      32,
      208],
     [7437,
      1,
      162,
      130,
      3013,
      823,
      58,
      4,
      7438,
      7439,
      136,
      7440,
      274,
      93,
      111,
      901,
      2375,
      43,
      82,
      310,
      14],
     [4765, 1, 7, 30, 4, 590, 4766, 1452, 562, 245, 199, 4767, 1862, 4768],
     [6607, 436, 2493, 95, 332, 6608, 709, 102],
     [635, 105, 506, 363],
     [6707, 6708, 73, 368, 1354, 523, 710, 149, 245, 18, 309],
     [8094,
      223,
      1578,
      39,
      307,
      208,
      223,
      8095,
      32,
      104,
      1012,
      63,
      283,
      1681,
      33,
      11],
     [6736,
      1,
      1505,
      2043,
      12,
      110,
      814,
      16,
      305,
      6737,
      6738,
      155,
      70,
      355,
      109,
      6739],
     [637, 4867, 5207, 12, 8, 442, 8571, 464, 1184],
     [4079, 68, 79, 3, 421, 4080, 1300, 972, 4081, 1745],
     [4141, 226, 1754, 737, 4142],
     [275, 198, 880, 8777, 2752, 149, 245],
     [7125, 1, 323, 430, 263, 239, 926, 7126, 72, 46],
     [8991,
      1,
      716,
      1112,
      1319,
      262,
      8992,
      2496,
      8993,
      2297,
      43,
      1216,
      3111,
      8994,
      8995,
      1463,
      9],
     [6617, 1, 7, 30, 76, 6618, 649, 541, 6619, 6620, 2050],
     [7668, 2323, 1202, 1550, 1104, 630, 2, 1879, 1759, 7669, 7670, 352, 8],
     [3893, 832, 1365, 1139, 1366, 621, 2, 2344, 2345, 1367, 262, 764, 33, 11],
     [6295, 6296, 2767, 721, 1008, 581, 6297, 6298, 6299, 118, 2920, 800, 1229],
     [1, 1997, 2860, 292, 132, 8581, 172, 2085],
     [6518, 2997, 272, 1209, 1418, 876, 570, 6519, 12, 388, 228, 329, 6520],
     [7096, 1010, 125, 7097, 953, 11, 100, 5, 111, 1108, 15, 1172, 431],
     [25, 8, 201, 535, 438, 23, 5, 6706, 535, 44, 353, 150, 2718, 2936, 5, 6706],
     [3795, 45, 825, 920, 88, 1682, 55, 295, 394, 32],
     [5923,
      162,
      487,
      2806,
      2857,
      802,
      5924,
      5925,
      1838,
      121,
      112,
      2318,
      2858,
      5926,
      5927],
     [6139, 1982, 6140, 1656, 95, 1149, 158, 40],
     [6709, 627, 174, 354, 62, 120, 45, 17, 94, 228, 2, 8, 36, 1000, 14],
     [2036, 133, 40, 2500, 1816, 104, 3130, 4],
     [8301, 209, 560, 160, 2, 524, 467, 302, 930],
     [5431,
      716,
      1906,
      5432,
      81,
      14,
      537,
      2753,
      31,
      31,
      10,
      49,
      72,
      46,
      54,
      503,
      5433,
      6,
      407,
      44],
     [6798, 1, 19, 6799, 1980, 1190, 6800, 6801, 819, 1243, 233, 6802],
     [9197, 830, 792, 16, 235, 425, 5, 1038, 149, 1005],
     [6497, 542, 457, 27, 38, 1479, 2994, 2995, 43],
     [6804,
      825,
      1224,
      1106,
      157,
      2989,
      6805,
      1106,
      408,
      404,
      34,
      200,
      6806,
      6807,
      133,
      59,
      431],
     [3813, 2303, 71, 479],
     [3, 68, 79, 455, 393, 32],
     [415,
      77,
      61,
      3,
      165,
      216,
      6,
      426,
      2297,
      205,
      260,
      229,
      7988,
      341,
      940,
      928,
      341],
     [60, 85, 904, 471, 133, 206, 1532, 60, 612, 447, 9148, 227, 745],
     [2416, 466, 2416, 29, 335, 885],
     [1, 303, 115, 542, 44, 702, 89, 1001, 127, 75, 1498],
     [7027, 18, 565, 200, 21, 3056, 7028, 3064, 182, 2, 7029, 2554, 7030, 897, 2],
     [1,
      6,
      147,
      8367,
      1951,
      152,
      3104,
      23,
      212,
      246,
      1250,
      301,
      374,
      1037,
      10,
      5223,
      2463,
      10,
      115,
      1160],
     [4358, 525, 89, 302, 192, 769, 232, 264, 12, 1676, 4359],
     [6482, 6483, 2, 43, 180, 2714, 673, 1987, 343, 7, 4],
     [1,
      2431,
      674,
      1809,
      1518,
      4992,
      1648,
      670,
      743,
      476,
      1552,
      443,
      1081,
      106,
      366,
      1127],
     [5328, 56, 134, 5329, 1504, 31, 2696, 513, 732, 5330, 5331, 27, 5332],
     [4667,
      1,
      2498,
      4668,
      696,
      4669,
      2555,
      1453,
      2561,
      1852,
      272,
      1468,
      4670,
      2437,
      188,
      2542],
     [6985,
      68,
      79,
      107,
      2651,
      36,
      282,
      6986,
      11,
      2913,
      43,
      6987,
      6988,
      899,
      201,
      442,
      380,
      2415,
      2609],
     [8083,
      639,
      536,
      503,
      63,
      93,
      72,
      46,
      266,
      136,
      129,
      54,
      266,
      136,
      60,
      123,
      36,
      109,
      8084,
      39,
      359],
     [290, 16, 172, 51],
     [4414, 996, 966, 4415, 1810, 4416, 320],
     [1, 57, 42, 103, 2223, 27, 2348, 62, 205, 27, 961, 2170],
     [8451,
      597,
      473,
      2491,
      150,
      1555,
      649,
      3,
      10,
      35,
      43,
      70,
      10,
      687,
      1832,
      8452,
      850,
      8453],
     [7483, 1477, 3110, 196, 211, 1451, 72, 46],
     [588, 207, 256, 7222, 98, 207, 4796, 122, 184, 14, 383],
     [1, 92, 167, 92, 2188, 309, 92, 6221, 92, 2187, 106],
     [1, 7, 30, 4, 18, 259, 7414, 221],
     [8425, 1, 367, 302, 7220, 853, 2833, 267, 13],
     [6357,
      1,
      241,
      83,
      1090,
      2956,
      2210,
      945,
      315,
      2957,
      2958,
      6358,
      6359,
      272,
      89,
      2959],
     [9130, 21, 2147, 3029, 280, 729, 25, 319, 5, 86, 276, 36, 340],
     [5054, 1, 77, 61, 47, 5055, 391, 386, 123, 1903, 1904, 259, 704, 1677],
     [5853, 1053, 5854, 2839, 2840, 954, 618, 593, 1265, 84, 46],
     [7246, 904, 471, 63, 9, 1362, 31, 62, 176, 51, 62, 176, 1513],
     [6612, 948, 1192, 193, 1275, 294, 28, 2520, 108, 1789, 28],
     [6109,
      1,
      6110,
      6111,
      2902,
      1165,
      909,
      1803,
      664,
      2009,
      711,
      1625,
      399,
      311,
      676,
      2010,
      1146,
      3,
      165,
      2903,
      605],
     [70, 70, 413, 2, 1576, 1577, 612, 3324, 2, 2143, 770, 3325, 904],
     [7965, 3165, 122, 5, 135, 7966, 10, 205, 28, 325, 231],
     [2431, 43, 32, 3, 411, 6811, 171, 297, 1274, 124, 509, 111, 86],
     [7341, 424, 44, 10, 185, 7342, 58, 242, 388, 1532, 125, 34, 7343],
     [8321, 855, 666, 13, 8322, 759, 33, 11, 8323, 8324],
     [4285, 123, 646, 2463],
     [3778,
      2312,
      3779,
      508,
      1677,
      2313,
      955,
      22,
      26,
      495,
      2314,
      3780,
      519,
      100,
      5,
      721],
     [3994, 60, 1146, 263, 185, 257, 171, 528, 3995, 3996, 844],
     [9205, 1, 385, 2070, 1518, 1886, 212, 246, 248, 21],
     [63, 37, 2202, 242, 37],
     [7997, 487, 54, 67, 1515, 74, 55, 678, 1418, 8],
     [7706, 2884, 421, 354, 3243, 750, 421, 165],
     [207, 165, 275, 2478, 992, 2479, 2480, 357, 207],
     [6276, 1031, 1263, 1533, 2023, 893, 692, 56, 2936, 6277, 40, 6278, 6279, 59],
     [8788, 8789, 115, 8790, 66],
     [1638, 2236, 3367, 1974, 87],
     [8465, 8466, 1527, 2935, 475],
     [4603, 1, 6, 147, 747, 670, 4604, 4605],
     [2235, 8, 141, 997, 1168],
     [1576, 33, 11, 182, 745, 206, 77, 61, 356, 546, 11, 59, 133],
     [4023, 25, 12, 1387, 1733, 471, 52, 534, 972, 1388, 1389, 4024, 514, 2385],
     [4397, 4398, 129, 70, 788, 4399, 2156, 637, 163, 1051, 12, 290, 440, 1262],
     [5376,
      1334,
      494,
      1022,
      5377,
      1560,
      22,
      608,
      5378,
      671,
      217,
      5379,
      5380,
      2734,
      1573,
      688,
      689,
      22,
      2409,
      1936,
      5381],
     [387, 274, 644, 838, 254, 50, 1722, 278, 447, 786, 500, 32],
     [8091,
      488,
      248,
      384,
      458,
      807,
      1024,
      327,
      46,
      35,
      756,
      8092,
      1002,
      3176,
      1024,
      115,
      39,
      374],
     [8755, 491, 2965, 579, 676, 222, 127, 2080, 182, 242, 504, 182, 14],
     [52, 2494, 1947, 2270, 102, 82, 1011, 293, 3, 19],
     [6060, 7, 218, 412, 122, 6, 2135, 364, 6061, 95, 460, 952, 1846, 254, 1030],
     [6351, 1, 1538, 2027, 16, 291, 80, 131, 1164, 469, 6352],
     [546,
      61,
      1072,
      1068,
      132,
      1395,
      25,
      306,
      8,
      9098,
      12,
      1068,
      485,
      1831,
      7,
      4,
      1024,
      62,
      963,
      20,
      1567],
     [7887, 29, 12, 7888, 11, 42, 103, 145, 37, 324, 361],
     [6463, 2917, 252, 28, 441, 82, 740, 2, 200, 1424],
     [3413, 88, 90, 449, 3, 68, 79, 2179, 1602],
     [3769, 809, 164, 10, 529, 1639, 40, 3770, 809, 148],
     [3815, 3816, 12, 1350, 651, 1341, 125],
     [6307, 210, 882, 86, 794, 113, 2435, 256, 47, 811, 794, 149, 256, 1716],
     [166, 753, 1499, 105, 543, 829, 635],
     [1, 162, 130, 368, 2825, 668, 383, 31, 196, 616, 277],
     [8576, 1, 179, 1837, 8577, 1433, 2859, 348, 91, 149, 245],
     [6467, 60, 635, 1072, 54, 1155, 1379, 2013, 1041, 22, 67, 155, 70],
     [3896,
      1,
      636,
      26,
      97,
      20,
      1707,
      339,
      587,
      418,
      802,
      728,
      124,
      189,
      27,
      22,
      26,
      24],
     [192, 170, 160, 8683, 499],
     [1, 568, 21, 385, 395, 300, 1455, 1122, 151, 90, 7386, 138],
     [3900, 1, 1368, 4, 1369, 9, 961, 510, 512, 228],
     [5352, 1, 128, 326, 392, 58, 340],
     [19, 196, 1036, 48, 2306, 20, 273, 322, 1423, 899, 1517, 2103],
     [345, 658, 36, 564, 43, 286, 156, 16, 345, 7, 1151, 881],
     [6327, 1, 934, 935, 211, 24, 17, 185, 1240, 2340, 992, 1213],
     [4529, 141, 2528, 16, 86, 71, 205, 28, 2529, 212, 246, 461, 172, 51],
     [6376,
      1,
      60,
      599,
      117,
      57,
      141,
      1014,
      143,
      6377,
      268,
      3,
      195,
      2484,
      367,
      40,
      1935,
      153],
     [1, 1145, 1306, 166, 29, 4605, 422, 459, 257, 48, 2692, 1735, 1228],
     [5985, 530, 482, 121, 34, 589, 59],
     [1489, 119, 438, 2, 334, 708, 2516, 163, 256],
     [6929, 483, 115, 533, 1928, 473, 109, 6930, 765, 13, 600, 1474],
     [8514, 1, 413, 834, 8515, 6, 1895, 84, 377, 245],
     [7, 781, 541, 78, 1617, 317, 528, 149, 245, 2281, 5096, 191],
     [6590, 716, 1112, 3009, 343, 177, 3010, 65, 6591, 34, 485],
     [8667, 1, 6, 65, 117, 1368, 8668, 8669, 7, 30, 4],
     [106, 106, 1036, 1035, 2, 6, 71],
     [4071, 468, 4072, 152, 76, 700, 1298, 1126, 550, 257, 249, 194, 137],
     [7703, 7704, 75, 17, 167, 689, 7705, 7706],
     [222, 6539, 1502, 8252],
     [3543,
      790,
      3544,
      84,
      2197,
      33,
      11,
      518,
      280,
      1306,
      3545,
      10,
      563,
      268,
      1100,
      310,
      2,
      6,
      225],
     [7146, 771, 1258, 44, 7147, 3012, 771, 546, 407, 7],
     [7756, 7757, 7758, 7759, 401, 997, 1747, 1748, 833, 3143, 7760, 4, 73],
     [6826,
      873,
      6827,
      6828,
      6829,
      1275,
      6830,
      6831,
      3036,
      20,
      977,
      652,
      6832,
      6833,
      6834],
     [3300,
      118,
      3301,
      216,
      1572,
      2130,
      66,
      3302,
      309,
      1572,
      2130,
      66,
      1573,
      689,
      3303,
      1265,
      33,
      11,
      66],
     [3312, 1, 550, 760, 181, 445, 145, 10, 610, 182, 13],
     [7675,
      179,
      2769,
      166,
      834,
      1025,
      7676,
      1280,
      1599,
      7677,
      15,
      740,
      23,
      285,
      671,
      1882,
      7678,
      7679,
      3134],
     [6989, 1, 7, 21, 61, 202, 360, 5, 278, 504, 166, 4, 32, 6990, 689, 95, 332],
     [42, 103, 368, 29, 95, 460, 40, 1101, 1157, 8003, 2085, 193, 13],
     [454, 1511, 7, 30, 4, 1685, 84, 8688, 912, 312, 6747, 150],
     [5088, 42, 103, 67, 907, 3, 2306, 273, 567, 795, 88, 2658],
     [9082,
      128,
      326,
      489,
      9083,
      413,
      2703,
      1206,
      9084,
      220,
      2638,
      2065,
      136,
      499,
      1342,
      603,
      905],
     [8766, 1, 52, 8767, 8768, 1554, 2096],
     [1, 385, 626],
     [7785, 325, 961, 3113, 196, 526, 340, 383, 612, 7786, 104, 1237, 888],
     [6925, 2009, 711, 6926, 39, 6927, 2065],
     [453, 10, 109, 44, 166, 147, 323, 430],
     [7176, 38, 177, 582, 55, 1154, 341, 25, 372, 675, 401],
     [7855, 77, 61, 153, 301, 7856, 7857, 7858],
     [8934, 55, 367, 723, 1421, 101, 2, 27, 205, 390, 2079, 3167, 351],
     [6346,
      29,
      435,
      1320,
      31,
      2875,
      2952,
      6347,
      378,
      12,
      2953,
      2195,
      6348,
      712,
      1240],
     [791, 352, 8, 5, 135, 107, 43, 221, 93],
     [6019, 836, 661, 1856, 31, 77, 61, 6020, 6021, 6022, 32],
     [5030,
      1,
      2251,
      20,
      764,
      1117,
      164,
      19,
      876,
      554,
      23,
      20,
      28,
      90,
      1381,
      11,
      5031,
      216,
      846],
     [4053, 786, 224, 9, 66, 66, 98, 171, 12, 110, 1087, 330, 171],
     [5772, 1, 162, 487, 67, 411, 2824, 135, 140],
     [4897, 1, 20, 188, 5, 202, 34, 1884],
     [5187, 2, 394, 1324, 65, 473, 109, 1635, 1723, 21, 1992, 3804],
     [2135, 15, 645, 217, 611, 335, 337],
     [7494, 34, 857, 1891, 1146, 7495, 7496, 7497, 375, 53, 649, 1146],
     [7637,
      1,
      21,
      61,
      396,
      24,
      804,
      30,
      4,
      16,
      24,
      586,
      104,
      2739,
      416,
      681,
      1435,
      97],
     [5337, 1, 10, 883, 5338, 18, 15, 29, 126, 92, 10, 74, 29, 5339, 15],
     [4792, 1, 869, 11, 17, 151, 643, 90, 158, 123],
     [817, 1099, 3121, 224, 9, 7222, 207],
     [8220, 1393, 9, 396, 51],
     [4062, 1, 978, 979, 49, 54, 4063, 23],
     [313, 2001, 170, 302, 5149, 256, 91],
     [5755, 1, 5756, 75, 146, 857, 1018, 223, 334, 1717, 940, 1642],
     [9079, 1, 85, 247, 16, 9080, 9081, 135, 6, 65, 1648, 934, 935, 16],
     [7892, 3157, 735, 1078, 263, 395, 137, 589, 1818, 338, 249, 194],
     [6566,
      12,
      29,
      50,
      337,
      2839,
      896,
      3,
      169,
      844,
      817,
      1675,
      3006,
      50,
      256,
      6567,
      43],
     [1183, 705, 391, 12, 3624, 1073, 25, 406, 105, 300, 216, 401],
     [4478, 567, 56, 466, 27, 600, 2, 452, 5],
     [3876, 5, 67, 2313, 476, 397, 65, 117, 3877, 3878, 1362, 835, 727],
     [1540, 50, 369, 1512, 137],
     [3226,
      213,
      235,
      214,
      3227,
      50,
      136,
      2103,
      213,
      1041,
      2104,
      2105,
      185,
      214,
      171],
     [4718, 2571, 2, 315, 1095, 2215, 1716, 168, 256, 216, 47, 50, 558, 601, 95],
     [1, 220, 2740, 726, 72, 1110, 2576, 5838, 119, 1012, 66, 39, 33, 11],
     [5561, 7, 4, 544, 1959, 5, 264, 1227, 2, 54, 839, 220, 835, 1373, 2779],
     [8593, 1, 789, 8594, 590, 6, 15, 317, 2456, 23, 96, 74, 1107, 8595, 15, 178],
     [6679, 2054, 22, 26, 495, 112],
     [929, 1006, 297, 282, 455, 32, 455, 366],
     [188, 351, 289, 1507, 2, 34, 4, 893, 7, 88, 188, 808, 86, 34, 8194],
     [5829, 130, 10, 697, 54, 1328, 91],
     [374, 20, 17, 201, 611, 2353, 1248, 1040, 40],
     [6206,
      387,
      30,
      76,
      6207,
      6208,
      1238,
      24,
      22,
      26,
      6209,
      522,
      6210,
      6211,
      2920,
      297,
      1825,
      357,
      848,
      17,
      195,
      6212,
      132,
      6213],
     [432, 114, 41, 2, 256, 1529, 57, 326, 13, 256, 1529, 19],
     [82, 784, 13, 9, 225, 52, 866, 2235, 780, 4825, 11, 284],
     [1, 1312, 868, 345, 2, 267, 2],
     [5235, 893, 399, 2700, 5236, 5235, 1864, 399, 1009, 368, 2700, 9],
     [4021, 1, 4022, 1386, 112, 313, 108, 27, 5, 944, 522],
     [7525,
      662,
      479,
      7526,
      7527,
      7528,
      368,
      7529,
      1242,
      861,
      138,
      481,
      24,
      7530,
      2328,
      244,
      336,
      2085],
     [128,
      42,
      3143,
      851,
      2992,
      969,
      300,
      3536,
      1064,
      2002,
      517,
      287,
      630,
      2352,
      982,
      2259,
      664,
      23,
      1396],
     [8412, 1, 20, 17, 8413, 170, 8414, 197, 181],
     [9109,
      9110,
      63,
      2278,
      159,
      870,
      2858,
      193,
      969,
      9111,
      263,
      2105,
      205,
      5,
      1243],
     [5139,
      166,
      5140,
      1493,
      5141,
      2669,
      1016,
      236,
      5142,
      881,
      304,
      2222,
      2669,
      1912,
      47,
      1212,
      5143],
     [5635,
      1,
      568,
      5636,
      5637,
      1046,
      174,
      5638,
      170,
      2786,
      856,
      5639,
      338,
      192,
      2788],
     [9143, 1028, 1029, 9144, 1188, 9145, 2397],
     [8157, 1982, 2826, 8158, 1578, 151, 603, 139],
     [8715, 1, 1081, 2093, 15, 884, 130],
     [5929, 84, 71, 1996, 1011, 76, 1234],
     [7845, 1, 1650, 3154, 211, 279, 318, 187, 7846, 90, 7847],
     [7441, 1, 555, 414, 679, 266, 269, 624, 1333, 7442, 7443, 951],
     [276, 3404, 500, 2993, 3, 500, 3, 112, 500, 3, 214, 85],
     [7432, 14, 3072, 320, 882, 126, 3065, 5, 956, 1791, 199, 27, 20, 100, 5],
     [4848, 1, 6, 56, 117, 2475, 2476, 152, 827, 1612, 80, 278],
     [1, 190, 240, 7744, 1493, 1518, 7745, 2709],
     [8932, 192, 67, 520, 2, 146, 78, 2182, 3845, 1360, 5],
     [4138, 1, 145, 2420, 1407, 75, 454, 1408, 4139, 5, 1159],
     [4615, 68, 867, 514, 1302, 24, 1598, 290, 370, 56, 290, 370, 65, 180, 791],
     [123, 159, 277, 391, 446, 701, 118],
     [3846, 7, 30, 4, 1356, 2321, 7, 2331, 3847],
     [5440, 1399, 1400, 1314, 559, 1508, 17, 44, 67, 771, 152],
     [3490, 1, 243, 787, 788, 2211, 355, 1619, 72, 46, 54],
     [6788, 115, 31, 607, 10, 2061, 6789, 1159],
     [3901, 98, 63, 131, 82, 101, 230, 1708, 962, 3902],
     [8539, 139, 828, 526, 1534, 2856, 64, 3, 366],
     [3745, 4, 1340, 1673],
     [4562, 1841, 2363, 177, 178, 82, 1842, 15],
     [3429, 699, 2191, 901, 3430, 182, 2, 34, 3431, 56, 3432, 3433],
     [9196, 262, 131, 33, 11, 1266, 206, 2292, 37, 131, 1522],
     [20,
      655,
      6643,
      1617,
      448,
      78,
      195,
      2964,
      2614,
      1091,
      1888,
      771,
      659,
      40,
      48,
      1202,
      48,
      137],
     [3549, 141, 706, 1307, 898, 413, 13, 3550, 1308, 163, 56, 1307, 898, 3551],
     [4597, 7, 4, 6, 46, 169, 574, 1846, 254],
     [6729, 360, 6730, 41, 2, 1036, 107, 170, 722, 1482, 345, 2],
     [5535, 33, 11, 35, 217, 20, 100, 5, 5536, 1957, 13, 64, 8, 90, 994],
     [683, 1, 127, 75, 1019, 1856, 2832, 290, 429, 27, 205, 154, 14],
     [5836,
      1,
      57,
      141,
      1014,
      5837,
      2663,
      151,
      1801,
      5838,
      435,
      5839,
      208,
      650,
      153,
      813],
     [1, 147, 3117, 7900, 1536, 7073, 2, 275, 2478, 1251],
     [5278, 2711, 1934, 92],
     [4484, 162, 130, 85, 432, 1818, 1819, 594, 744, 4485, 31, 744, 1820],
     [4752, 1012, 1475, 50, 144, 143, 591, 43, 4753],
     [9261, 873, 9262, 9263, 369, 932, 2007, 1855, 9264],
     [6121,
      200,
      639,
      1217,
      6122,
      6123,
      6124,
      6125,
      1984,
      6126,
      886,
      479,
      78,
      2012,
      2904,
      682,
      73,
      328,
      897,
      2,
      866,
      2905,
      184,
      2,
      1935],
     [6631, 360, 6632, 41, 2, 98, 66, 1036, 107, 170, 722, 1482, 345, 2],
     [8706, 8707, 803, 8708, 1776, 3210, 552],
     [6689, 1, 17, 6690, 6691, 818],
     [4557, 24, 1, 236, 1187, 4558, 4559, 1, 187, 1838],
     [8760, 521, 1537, 8761, 100, 28, 8762, 1673],
     [4168,
      84,
      1761,
      1762,
      2432,
      416,
      1047,
      513,
      338,
      4169,
      1763,
      4170,
      4171,
      4172],
     [705, 177, 403, 449, 464],
     [6823, 6824, 6825, 1558, 522, 972, 646, 80, 139, 24],
     [536,
      1725,
      777,
      399,
      5,
      1159,
      6634,
      28,
      727,
      713,
      4703,
      501,
      502,
      133,
      212,
      246,
      217,
      4416],
     [7253, 589, 111, 7254, 1196, 1114],
     [2024, 3095, 581, 2262, 169, 2788],
     [4957, 7, 4, 402, 9, 2620, 547, 2621, 2622, 4958, 2623],
     [186, 2089, 818],
     [8580, 160, 3, 1162, 8581, 483, 40, 25, 8],
     [2720, 521, 5319, 2721, 340, 34, 5320, 225, 1810, 45, 111, 121],
     [7849, 1, 2209, 7850, 446, 7851, 111, 680, 1349, 7852, 272],
     [7055, 1198, 1346, 72, 46, 144, 43, 55, 381, 908, 633, 656, 39, 72, 46],
     [7335, 209, 1918, 7336, 1706, 533, 1390, 163, 215],
     [7444, 7445, 7446, 812, 282, 6, 7447, 352, 154, 14],
     [60, 117, 85, 395, 267, 2, 1958, 153],
     [5448,
      350,
      204,
      788,
      2668,
      847,
      5449,
      595,
      1200,
      18,
      290,
      2761,
      233,
      514,
      5450],
     [2073, 1223, 183, 696, 547, 94],
     [323, 1419, 54, 67, 78, 205, 2262, 3050, 272, 1418],
     [1, 8194, 1320, 195, 153, 4406, 8189, 3187, 2564, 3, 56, 59],
     [4092, 17, 167, 4093, 1747, 1748, 2405, 291, 4094, 4095, 416, 4096],
     [5459, 1, 4, 73, 5460, 1467, 279, 1646, 263, 362, 70, 1948, 1949, 1467, 687],
     [310,
      8110,
      825,
      2778,
      176,
      376,
      1025,
      2751,
      967,
      40,
      1019,
      5088,
      2751,
      967,
      40,
      140],
     [3803, 2320, 126, 92, 94, 20, 471, 723, 271, 3804, 650, 1128, 5, 956],
     [4364, 142, 224, 9, 357, 4365, 24, 137, 38],
     [4351, 990, 1152, 211, 51],
     [1, 77, 61, 273, 686, 2204],
     [5930, 1997, 2860, 16, 83, 69, 20, 1593, 1865, 97],
     [6896, 1, 6897, 258, 183, 656, 848, 6898, 6899, 3044, 1520, 6900, 6901, 6902],
     [2060,
      857,
      2804,
      1335,
      2773,
      121,
      208,
      1336,
      1337,
      81,
      268,
      858,
      941,
      3101,
      2882,
      2101],
     [8943,
      1,
      6,
      147,
      510,
      615,
      31,
      435,
      1258,
      459,
      603,
      878,
      1016,
      381,
      43,
      1705,
      8944,
      8945],
     [5397, 52, 24, 104, 2739, 416, 681, 1435, 97],
     [631, 31, 1018, 1033, 2501, 179, 1085, 631, 589, 31, 1575, 807, 29, 44, 473],
     [1, 669, 1967, 121, 106, 31, 1195, 1170, 851, 509],
     [145,
      282,
      302,
      254,
      50,
      5517,
      382,
      182,
      255,
      773,
      2,
      301,
      592,
      6,
      55,
      255,
      219,
      2],
     [5176,
      1213,
      992,
      5177,
      2684,
      5178,
      5179,
      195,
      423,
      1916,
      32,
      198,
      283,
      515,
      5180,
      497,
      85,
      432,
      60,
      1917,
      698,
      2399,
      1352,
      784,
      463,
      653,
      537,
      1918,
      1919,
      5181,
      5182,
      51,
      135,
      654,
      54,
      2685,
      5183,
      60,
      5184,
      292,
      481,
      60,
      101,
      228,
      101,
      101,
      41,
      463,
      153,
      737,
      5185,
      1920,
      1740,
      1656,
      198,
      20,
      108,
      1214,
      1602,
      272,
      158,
      5186,
      13,
      842,
      5187,
      5188,
      5189,
      1305,
      52,
      718,
      234,
      234,
      234,
      41,
      234,
      1921,
      737,
      5190,
      5191,
      1920,
      52,
      828,
      750,
      13,
      8,
      119,
      5192,
      5193,
      5194,
      2686,
      140,
      5195,
      52,
      370,
      41,
      101,
      101,
      373,
      234,
      2687,
      5196,
      153,
      1305,
      5197,
      845,
      127,
      75,
      1498,
      179,
      1922,
      2688,
      5198,
      733,
      820,
      5199,
      853,
      891,
      2689,
      5200,
      857,
      500,
      2513,
      5201,
      5202,
      29,
      718,
      228,
      228,
      228,
      41,
      41,
      463,
      153,
      1002,
      1082,
      5203,
      2391,
      107,
      761,
      2,
      386,
      100,
      190,
      240,
      509,
      171,
      5204,
      3,
      2690,
      52,
      60,
      234,
      41,
      234,
      234,
      234,
      2691,
      1251,
      2692,
      5205,
      2693,
      1740,
      33,
      11,
      88,
      37,
      1494,
      232,
      107,
      319,
      32,
      1215,
      2541,
      5206,
      6,
      440,
      5207,
      987,
      382,
      227,
      707,
      869,
      234,
      2691,
      5208,
      358,
      473,
      5209,
      845,
      5210,
      85,
      5211,
      874,
      5212,
      91,
      226,
      562,
      9,
      113,
      800,
      5,
      1302,
      157,
      2343,
      333,
      874,
      523,
      2247,
      5213,
      5214,
      5215,
      5216,
      5217,
      68,
      79,
      68,
      79,
      2694,
      234,
      234,
      234,
      234,
      234,
      463,
      537,
      1803,
      5218,
      2695,
      149,
      1005,
      74,
      3,
      68,
      79,
      572,
      13,
      8,
      5219,
      5220,
      7,
      68,
      1671,
      488,
      24,
      578,
      68,
      79,
      2694,
      41,
      234,
      234,
      234,
      41,
      463,
      5221,
      1919,
      5222,
      2695,
      1,
      788,
      2211,
      16,
      5223,
      70,
      10,
      90,
      17,
      559,
      2537,
      616,
      10,
      90,
      5224,
      5225,
      1923,
      276,
      370,
      41,
      41,
      234,
      234,
      41,
      463,
      473,
      5226,
      1920,
      2696,
      70,
      2697,
      597,
      58,
      388,
      49,
      101,
      2,
      842,
      78,
      962,
      44,
      154,
      14,
      2288,
      5227,
      276,
      60,
      41,
      41,
      41,
      101,
      234,
      463,
      5228,
      5229,
      845,
      25,
      29,
      2508,
      343,
      21,
      235,
      408,
      27,
      3,
      108,
      5230,
      5231,
      2407,
      692,
      3,
      2698,
      29,
      60,
      101,
      234,
      41,
      101,
      101,
      1921,
      1919,
      653,
      537,
      5232,
      5233,
      2693,
      384,
      458,
      1113,
      448,
      2502,
      202,
      2515,
      121,
      742,
      2699,
      1924,
      2565,
      365,
      59],
     [3363, 1, 77, 1052, 153, 561, 693, 1588, 3364, 120, 1065],
     [9269, 1, 147, 1968, 9270, 16, 3151, 478],
     [7080, 141, 323, 2136, 2829, 6, 286, 156],
     [3379, 2158, 2, 3380, 32, 2159, 2160],
     [8, 788, 2211, 429, 40, 105, 131, 549, 70],
     [9235, 1, 147, 789, 9236, 677, 273, 9237, 9238, 137],
     [6006, 1, 150, 2240, 33, 11, 640, 284, 58, 265, 630, 161, 221],
     [4202,
      279,
      187,
      1,
      32,
      1589,
      4203,
      1662,
      318,
      91,
      189,
      1769,
      574,
      1139,
      4204,
      2439],
     [7905,
      663,
      165,
      1039,
      535,
      1079,
      2,
      3159,
      3091,
      195,
      146,
      1530,
      408,
      3159,
      195,
      1893,
      2160,
      7906,
      33,
      11],
     [7487, 2850, 705, 2263, 119],
     [247,
      1102,
      503,
      96,
      4582,
      8806,
      91,
      166,
      438,
      23,
      2653,
      144,
      380,
      94,
      197,
      219,
      23,
      3343],
     [6613, 531, 491, 2996, 6614, 70, 370, 60, 109, 1148, 1985, 154, 69],
     [8360, 1601, 464, 401, 320, 365, 139, 483],
     [261, 23, 1305, 51, 115, 724, 196, 203, 120],
     [4835,
      948,
      4836,
      16,
      1011,
      4837,
      1876,
      762,
      1149,
      2591,
      75,
      354,
      666,
      13,
      842],
     [5519, 228, 2, 470, 382, 1917, 62, 496, 378, 239, 639, 2773],
     [1, 6, 65, 117, 8574, 8575, 123, 84, 260, 229],
     [165, 1469, 2443, 150, 275, 132, 1211, 334, 1726, 1004, 7701],
     [848, 676, 222, 446, 29, 1972, 114, 1760, 42, 103, 133, 59],
     [508, 126, 92, 688, 2411, 2410],
     [5355, 1, 41, 525, 63, 192, 181, 445],
     [7648,
      7649,
      3131,
      10,
      12,
      11,
      161,
      435,
      27,
      3,
      300,
      7650,
      679,
      10,
      838,
      593,
      367],
     [1, 2212, 8777, 197, 778, 6246, 206, 295, 45, 872, 2611, 48, 113],
     [7, 30, 4, 115, 7522, 574],
     [9201, 1621, 2602, 566, 156, 3138, 1094, 1377, 637, 1751],
     [4303, 2467, 12, 161, 50, 1372, 102, 71, 299],
     [98, 154, 161, 271, 2170, 600, 2, 2389, 208, 301, 406, 8308],
     [56, 109, 8048, 21, 61, 1097, 7839, 3002, 109, 1324],
     [25, 43, 210, 255, 260, 229, 210, 882, 332, 15],
     [9, 216, 5071, 1701, 296, 105, 188, 32],
     [5089, 1370, 399, 2659, 44, 22, 26, 1909],
     [5045, 81, 455, 1120, 164, 517, 32, 3, 2646, 5046, 75],
     [4275, 4276, 115, 2461, 113, 1785, 86, 8],
     [7742, 1, 126, 92, 555, 414, 2345, 3141, 118, 1817, 414, 16, 400, 38, 97],
     [6032,
      1,
      184,
      2,
      274,
      75,
      748,
      6033,
      12,
      2888,
      1963,
      75,
      6034,
      1492,
      879,
      189,
      146,
      685,
      545,
      6035,
      687,
      347],
     [5669, 5670, 5671, 1518, 33, 11, 5672, 2764, 2765],
     [7625,
      1,
      165,
      1469,
      46,
      970,
      656,
      7626,
      184,
      7627,
      7628,
      585,
      25,
      401,
      805,
      214,
      33,
      11,
      2642,
      1440,
      1064,
      1006,
      7629,
      7630,
      262,
      371],
     [1161,
      145,
      37,
      275,
      423,
      640,
      84,
      145,
      37,
      29,
      12,
      7688,
      145,
      111,
      516,
      275,
      225,
      926,
      516,
      1161],
     [4780, 1477, 2580, 476, 397, 2581, 4781, 1185, 27],
     [68, 79, 15, 1871, 259, 5672],
     [5685,
      1,
      29,
      350,
      204,
      5686,
      1973,
      5687,
      2804,
      990,
      5688,
      2805,
      532,
      82,
      8,
      269],
     [8497, 1993, 1994, 36, 7, 321, 70, 1517, 2103, 8498, 159],
     [542, 3, 4419, 198, 9, 908, 209, 4926, 2926],
     [432, 42, 6406, 60, 1912, 3508],
     [25, 312, 34, 857, 198, 2735, 858, 33, 11, 2439, 14, 982, 857],
     [7000, 1877, 2, 692, 7001, 5, 67, 492, 360, 41, 2],
     [3221, 3222, 42, 490, 1556, 1248, 1040, 375, 1249, 29, 15],
     [5097, 117, 5098, 406, 40, 9, 176, 1353, 594, 1663, 2279],
     [821, 822, 513, 848, 2864],
     [1, 18, 349, 1013, 20, 5, 64],
     [8987, 6, 65, 12, 8988, 8989],
     [4040,
      1,
      945,
      108,
      2390,
      161,
      311,
      164,
      181,
      1334,
      973,
      198,
      11,
      1149,
      367,
      1137,
      99,
      348],
     [5321, 1, 2722, 5322, 5323, 5324, 5325, 1166, 486, 194, 778],
     [3455,
      1,
      1085,
      923,
      623,
      624,
      291,
      1612,
      80,
      783,
      3456,
      380,
      58,
      138,
      241,
      65,
      2138,
      21,
      3457,
      624],
     [4584,
      497,
      7,
      21,
      61,
      61,
      22,
      553,
      2542,
      4585,
      4586,
      4587,
      4588,
      2543,
      4589,
      1127],
     [1, 77, 1052, 67, 10, 5, 198, 325, 231, 107, 160, 2],
     [11, 214, 26, 272, 173, 797],
     [684, 2144, 1024, 665, 318, 2995],
     [8516, 1, 358, 6, 65, 117, 934, 935, 462, 752, 549, 56],
     [1534, 1, 125, 18, 349, 756, 600, 23, 231, 6710],
     [264, 110, 8894, 327, 12, 1649],
     [4596, 264, 1061, 1188, 1003, 902, 533, 8],
     [6258, 1, 6259, 355, 2441, 2932, 21, 511],
     [5849, 127, 1682, 157, 119, 5850, 329, 1523],
     [8673, 199, 2585, 20, 96, 21],
     [5978, 2665, 157, 237, 226, 1887, 219],
     [4041, 276, 2370, 355, 42, 4042, 153, 1150, 2391],
     [4901,
      1,
      433,
      1317,
      1885,
      1886,
      212,
      246,
      1481,
      21,
      4902,
      191,
      1443,
      4903,
      394],
     [4455, 1, 648, 345, 2, 225, 332, 19, 36, 340, 4456],
     [1, 29, 57, 42, 103, 1241, 772, 1278, 1529, 38, 1443],
     [1, 5466, 6523, 5, 135, 1246, 21, 31, 129, 536, 2289, 2289],
     [454,
      2545,
      295,
      1985,
      97,
      4329,
      105,
      131,
      506,
      919,
      309,
      20,
      2306,
      294,
      7272,
      20,
      17],
     [3952, 1, 77, 61, 16, 758, 25, 259, 704, 74, 151, 32, 138, 15],
     [6360,
      1,
      147,
      141,
      6361,
      590,
      1195,
      109,
      1236,
      190,
      240,
      34,
      303,
      59,
      303,
      266,
      83,
      69],
     [791, 425, 59, 411, 110, 69, 55, 8140],
     [7368, 40, 1628, 527, 2065, 677, 49, 142, 135, 547, 335, 964],
     [8589, 1, 8590, 8591, 3017, 77, 61, 752, 8592, 247, 2193],
     [5931,
      1,
      799,
      683,
      2431,
      644,
      2203,
      5932,
      86,
      5933,
      1274,
      1106,
      409,
      108,
      523,
      78,
      1144,
      2861,
      1525,
      746,
      5934],
     [5313, 29, 2717, 84, 55, 84, 55, 5314, 2, 225, 2717, 29, 1281, 1938, 2],
     [232, 37, 192, 3142, 120, 261, 2, 192, 1516, 12, 381, 219, 2, 98],
     [52, 462, 160, 11, 1481, 378, 112],
     [4593, 1, 77, 61, 4594, 35, 704, 320, 106, 1081, 4595, 1460, 1121, 938],
     [4101,
      2407,
      60,
      4102,
      502,
      419,
      4103,
      311,
      82,
      142,
      535,
      621,
      155,
      115,
      776,
      4104,
      59],
     [1137, 9, 1422, 19, 25, 116],
     [352, 110, 14, 2403, 620, 880, 553, 48, 1527, 1186, 227, 14, 284],
     [6243, 222, 915],
     [1, 128, 326, 58, 40, 3, 872, 7582, 391, 1639],
     [3655,
      1,
      162,
      130,
      54,
      78,
      330,
      209,
      1327,
      3656,
      269,
      1328,
      241,
      80,
      581,
      3657,
      936,
      99,
      8,
      804,
      67],
     [5326,
      5327,
      3,
      44,
      1054,
      57,
      42,
      103,
      60,
      770,
      356,
      1562,
      1300,
      623,
      2723,
      504],
     [723, 926, 2076, 1197, 33, 11, 62, 723, 63, 1197, 1552],
     [1776, 1840, 1018, 282, 631, 264, 227, 14, 1063, 24, 1179, 2059, 24],
     [8356, 8357, 507, 8358, 8359, 5],
     [52, 786, 5],
     [7283, 2624, 11, 1238, 2556, 1037],
     [1, 6, 440, 978, 979, 16, 2491, 150, 413, 932, 69],
     [8799, 123, 964],
     [3391, 1, 6, 147, 1596, 2164, 2165, 16, 49, 611, 275, 316],
     [5968, 836, 210, 266, 36, 1000, 14, 222, 37, 111, 289, 441, 157, 833, 94],
     [1056, 1267, 4283, 740, 23, 1600, 11, 393, 172],
     [29, 24, 8776, 129, 226, 151, 2832, 3, 156, 1041, 1797, 3858],
     [2242, 851, 113, 379, 331, 936, 89, 13, 875, 1091],
     [181, 1334, 58, 392, 7, 122, 392, 7, 4],
     [7392, 896, 438, 23, 8, 484, 7393, 106],
     [8696, 1, 18, 271, 298, 24, 854, 8697, 1481, 8698],
     [85, 2118, 2894, 858, 1125, 1177, 610, 15, 626, 3, 285, 1798],
     [6593, 678, 733, 6594, 1370, 6595, 32],
     [1, 1092, 1135, 751, 434, 2099, 38],
     [8111, 3007, 515, 144],
     [7307, 6, 3100, 5, 1038, 7308, 800, 100, 28],
     [6050,
      83,
      725,
      1001,
      2891,
      18,
      28,
      960,
      500,
      95,
      332,
      227,
      14,
      82,
      6051,
      18,
      1420,
      15,
      6052,
      27,
      2892,
      18,
      2893,
      500,
      439,
      140,
      169,
      320],
     [5651,
      1,
      669,
      1967,
      31,
      1968,
      5652,
      5653,
      870,
      209,
      484,
      125,
      1500,
      639,
      1179],
     [8049, 1462, 316, 1581, 436, 463, 1506, 74, 51],
     [7086,
      499,
      439,
      328,
      7087,
      1018,
      92,
      734,
      644,
      7088,
      2386,
      1104,
      137,
      2246,
      3073,
      237,
      2074,
      7089,
      245],
     [8582,
      2981,
      6,
      286,
      156,
      1218,
      2258,
      364,
      7,
      76,
      604,
      10,
      665,
      221,
      88,
      156,
      2033,
      8583,
      2058],
     [4025,
      1699,
      1700,
      4026,
      847,
      4027,
      1734,
      4028,
      7,
      30,
      76,
      336,
      4029,
      519,
      626,
      91,
      2386,
      1390,
      2218,
      342,
      466,
      306],
     [8699,
      1,
      1685,
      246,
      2524,
      1228,
      3099,
      8700,
      8701,
      1046,
      8702,
      846,
      109,
      8703],
     [6693, 1, 130, 2820, 1496, 6694, 3022, 1471, 167, 6695],
     [5840,
      1,
      82,
      765,
      2,
      53,
      5841,
      875,
      543,
      5842,
      1220,
      5843,
      302,
      192,
      16,
      2835,
      1091],
     [3823, 768, 3824, 105, 106, 39, 105, 768],
     [7585, 70, 425, 311, 9, 621, 2, 115, 356, 33, 11],
     [1, 774, 613, 174],
     [130, 1856, 2490, 300, 54, 138],
     [7463, 323, 430, 10, 1166, 7464, 7465],
     [4772,
      3,
      19,
      169,
      1136,
      22,
      298,
      1864,
      254,
      504,
      255,
      388,
      1107,
      160,
      13,
      265,
      81,
      14],
     [4286,
      1384,
      105,
      10,
      4287,
      78,
      307,
      32,
      87,
      1429,
      113,
      262,
      105,
      12,
      315,
      824],
     [1, 65, 2364, 10, 2471, 8231, 204, 971, 1753, 1118, 9, 2349, 222],
     [68, 1671, 27, 282, 88, 646, 394, 102],
     [8828,
      7,
      4,
      2682,
      768,
      592,
      63,
      9,
      433,
      150,
      1555,
      361,
      310,
      133,
      1555,
      123,
      1098,
      1369],
     [987, 9086, 569, 3065, 5, 67, 533, 727, 406, 426, 10, 6553, 219, 230, 71],
     [4468, 1, 4469, 4470, 4471, 5, 4472, 592],
     [673, 94, 159, 433, 501, 502, 5, 135, 31, 406, 974],
     [7006, 2392, 7007, 119, 7008, 3062, 50, 2067, 334, 7009, 1548, 3062, 116],
     [1, 747, 670, 385, 1004, 4027, 2656, 132, 3162, 7953, 1557, 31, 3013],
     [9155, 1514, 9156, 2886, 9157, 106, 1599, 480, 48, 22],
     [1, 60, 742, 4293, 29, 2698, 545, 176, 907, 72, 46, 608, 212, 246],
     [6699, 1, 127, 75, 337, 350, 117, 6700, 6701, 724, 2921],
     [139, 645, 182, 13, 1072, 595, 4866],
     [5766, 115, 5767, 5768, 141, 1014, 365, 85],
     [6974, 59, 6975, 3058, 584, 59],
     [7274, 86, 7275, 5, 200, 1692, 1355, 2569, 2997, 9, 170, 773],
     [254, 2976, 272, 2297, 1985, 163, 56],
     [9019,
      1183,
      9020,
      2334,
      3185,
      1157,
      9021,
      5,
      64,
      2178,
      13,
      17,
      575,
      9022,
      9023,
      3188,
      3189,
      591,
      9024,
      9025,
      2863,
      13,
      376,
      238,
      442,
      189,
      406,
      2501],
     [14, 3, 483, 115, 16, 5, 86],
     [6075,
      1923,
      620,
      532,
      6076,
      31,
      1575,
      1056,
      1267,
      269,
      532,
      269,
      2115,
      199,
      209,
      1169],
     [6924, 58, 138, 955, 1228, 2009, 711, 806],
     [8471, 30, 4, 1442, 47, 113, 604, 159, 618],
     [8146, 1742, 1743, 1097, 883, 8147, 132, 194, 19],
     [3955, 81, 13, 20, 151, 2360, 3956, 967, 840, 841, 20, 108],
     [8057, 1, 1458, 2412, 142, 617, 8058, 337, 8059, 306],
     [3009, 9, 6164, 32, 34, 3201],
     [8408, 3156, 2, 9, 32, 31, 736, 737, 1175, 1744, 2387, 848, 226, 2738, 97],
     [6793, 1, 7, 30, 4, 71, 255, 1227, 2, 114, 7, 94, 208, 694, 512],
     [143, 153, 85],
     [4417,
      1,
      4418,
      4419,
      424,
      84,
      46,
      1661,
      328,
      4420,
      279,
      2493,
      4421,
      4422,
      2451],
     [183,
      249,
      2513,
      281,
      1502,
      305,
      1665,
      864,
      2088,
      250,
      101,
      8830,
      4852,
      864,
      418,
      1422,
      118,
      265,
      1813,
      2933,
      3532,
      112,
      47,
      50],
     [658, 453, 403, 1845, 7, 4, 1383, 612, 1107, 684, 4527, 122, 73],
     [5753, 22, 26, 24, 5754, 161, 5, 86, 93, 501, 502],
     [8391,
      1889,
      97,
      2954,
      2955,
      10,
      226,
      321,
      156,
      258,
      3163,
      8392,
      1229,
      8393,
      8394,
      1889,
      97],
     [7247, 1517, 5, 86, 395, 45, 17],
     [8096, 1430, 207, 357, 8097, 105, 472, 39, 254],
     [1, 1183, 49, 3006, 50, 2008, 1406, 2179, 503, 16, 205, 50, 2067, 351],
     [5252, 183, 40, 190, 240, 21, 10, 304, 17, 686],
     [5512,
      36,
      8,
      21,
      1208,
      143,
      707,
      13,
      64,
      1226,
      518,
      5513,
      1002,
      1299,
      2590,
      107,
      25,
      5514],
     [210, 238, 456, 1011, 732, 167, 48, 456, 4245, 112, 1423],
     [4061, 38, 90, 1154, 341, 25, 41, 215, 39, 120],
     [3258, 85, 162, 130, 202, 894, 186, 762, 444, 69],
     [5368,
      5369,
      1220,
      5370,
      371,
      744,
      354,
      294,
      5371,
      2733,
      180,
      5372,
      262,
      105,
      351,
      1506],
     [726, 1485, 4, 1971, 300, 2945, 2154, 295, 2611, 1296],
     [6222, 1, 128, 326, 513, 2922, 331],
     [4125, 85, 130, 304, 48, 793, 169, 796, 1401],
     [57, 128, 326, 7606, 2815, 10, 3038, 5091, 27, 86],
     [5501, 392, 682, 300, 5502],
     [4201, 1387, 394, 134, 24, 560, 264, 2438, 8, 335, 93, 34, 210, 2269, 391],
     [2867, 1276, 1, 18, 937, 577, 585, 15, 8, 74, 55, 192, 2980, 8],
     [1, 4270, 3, 24, 7440, 432, 584, 3999, 3052, 356, 59],
     [3725, 142, 3726, 19, 90, 164],
     [8060, 689, 8061, 829, 1695],
     [3507, 12, 631, 569, 3508, 2221, 2222, 3509, 332, 418],
     [6803, 1, 978, 979, 440, 109, 1469, 46],
     [2643, 2062, 109, 2388, 7, 76, 881, 974, 781, 1513, 149, 1005],
     [4343, 556, 199, 2474, 1437, 7, 1438, 277, 1588, 1437, 7],
     [6624, 1417, 427, 20, 95, 1288, 503, 644, 444, 914],
     [6777, 19, 3033, 32, 2048, 613, 1274],
     [2512, 375, 5362, 432, 15],
     [8572, 1, 8573, 60, 65, 117, 8574, 8575, 10, 907, 72, 46],
     [5844, 1, 747, 670, 5845, 24, 5846, 5847, 6, 988],
     [1, 42, 103, 31, 2540, 38, 2505, 171, 39, 7154, 878],
     [4886, 2601, 4887, 853, 1480, 1145],
     [1264, 7376, 2120, 1686, 7126, 41],
     [3774, 18, 349, 314, 530, 3775, 34, 468, 634, 893, 85],
     [7267, 20, 17, 7268, 101, 23, 2808, 105, 300, 1988, 281, 7269, 105, 188],
     [3372, 694, 512, 774, 3373, 695, 3374, 1592, 152, 1284, 3375, 305],
     [6642, 773, 14, 383, 33, 11, 6643, 159, 1543, 2737, 6644, 477, 507],
     [8979, 33, 11, 882, 1032, 1600, 9, 22, 80, 51],
     [5626,
      127,
      75,
      1836,
      22,
      1453,
      754,
      2786,
      5627,
      5628,
      1197,
      5629,
      5630,
      1829,
      5631,
      201,
      353,
      2787],
     [44, 1021, 271, 933, 799, 41, 2, 408, 461, 4330, 762, 762],
     [1, 128, 3235, 16, 398],
     [8785, 8786, 24, 578, 421, 908, 796, 1641, 8, 8787, 1176, 375, 1952, 2311],
     [1, 168, 2, 1859, 2345, 434, 2781],
     [1230, 744, 740, 744, 2157, 754, 1230],
     [7638, 1, 7, 30, 4, 538, 114, 1553, 268, 7, 1348, 70, 538, 1553, 268, 70, 93],
     [114, 81, 14, 383, 167, 4182, 8746, 6, 158, 754, 98, 167, 603, 2815],
     [2722, 5322, 21, 305, 509, 118],
     [6009,
      2885,
      265,
      340,
      1293,
      3,
      19,
      6010,
      1160,
      2885,
      265,
      154,
      444,
      6011,
      14],
     [4482, 765, 2, 638, 265, 89, 14],
     [8114, 8115, 250, 625, 8116, 141, 1718, 31, 8117, 477, 159, 3179],
     [3957, 2361, 968, 583, 651, 3958, 968, 2361],
     [4574, 4575, 256, 4576],
     [4229, 345, 4230, 38, 1379, 38, 4231, 2445],
     [8910, 12, 784, 53, 98, 39, 8911, 8912, 180, 12, 784, 53, 1484],
     [77, 1052, 3, 165, 307, 110, 1081, 146, 4585, 159, 7250],
     [9036, 868, 913, 570, 1849, 112, 1527, 9037, 53, 2978, 618],
     [7081, 47, 811, 7082, 7083, 132, 297, 2072, 196, 88, 1092, 37, 32],
     [1,
      9,
      396,
      922,
      29,
      3,
      175,
      521,
      429,
      1175,
      1688,
      1654,
      8901,
      2390,
      358,
      233,
      2991,
      9132],
     [7468, 2914, 26, 289, 219, 2, 35, 364, 164, 712, 11, 1198],
     [3632, 3633, 385, 1324, 297, 322, 3634, 933, 91],
     [1347, 1068, 1903, 1904, 259, 187],
     [141, 273, 4390, 5, 1485, 111, 7385, 426, 15, 802, 548],
     [1957, 1327, 6196, 180],
     [4683, 183, 1854, 5, 27, 63, 231, 124, 106, 4684, 106, 1855, 6, 183, 5],
     [3973, 2367, 107, 219, 14, 48, 308, 3974],
     [8811,
      874,
      359,
      471,
      1643,
      1308,
      8812,
      441,
      354,
      823,
      8813,
      2016,
      149,
      1005,
      317,
      249,
      471],
     [34,
      7,
      4,
      363,
      59,
      1107,
      101,
      13,
      6594,
      8442,
      462,
      232,
      157,
      930,
      119,
      769,
      232,
      255,
      422,
      2,
      2507,
      1476,
      43],
     [8294,
      311,
      1613,
      19,
      465,
      3164,
      116,
      174,
      957,
      2273,
      12,
      1245,
      8295,
      2903,
      1004,
      311,
      33,
      2865,
      8296],
     [3566,
      7,
      121,
      33,
      221,
      3567,
      574,
      575,
      3568,
      31,
      269,
      244,
      3569,
      3570,
      574,
      2139,
      2236],
     [3264, 292, 3265, 3266, 249, 236, 3267],
     [4294, 4295, 904, 499, 1788, 237, 18, 565, 3, 534, 291, 185, 700, 1298, 2465],
     [175, 384, 1479, 3, 52, 667, 72, 46, 2381, 13, 9, 232],
     [6018, 432, 137, 925, 41, 160, 11],
     [3483, 243, 9, 547, 915, 3484, 1618, 220, 3485],
     [9250,
      68,
      79,
      1016,
      562,
      2099,
      38,
      22,
      80,
      1933,
      813,
      1678,
      572,
      312,
      2047,
      312,
      9251,
      312,
      201,
      181],
     [5586, 257, 628, 299, 125],
     [6256, 33, 11, 982, 102, 87, 88, 408, 2930, 2931, 6257, 378, 239],
     [8841, 55, 872, 750, 53, 216, 2094, 8, 146, 170, 40, 12, 360, 873],
     [1, 30, 4, 5268, 345, 7626, 9024, 7],
     [8005, 285, 1467, 279, 8006, 219, 2, 1401, 182, 2, 145],
     [5401, 1, 29, 85, 5402, 511],
     [4123, 4124, 335, 337, 267, 801, 1348, 1404, 594],
     [1, 7, 30, 4, 35, 7, 210, 320, 489, 1817, 297, 6229, 297, 4316],
     [129, 70, 942, 315, 5, 1485, 6476, 648],
     [8527,
      1,
      77,
      1052,
      279,
      318,
      187,
      8528,
      3182,
      3123,
      157,
      12,
      3,
      3171,
      618,
      40],
     [3785, 118, 66, 413, 230, 339, 71, 180, 530, 1678, 71],
     [215, 3404, 7150, 365, 1109, 59, 314, 38],
     [4370, 4, 133, 59, 125, 33, 11, 111, 2483, 35, 43, 451, 14],
     [9252, 1, 384, 458, 31, 759, 317, 528, 9253, 493, 759, 226, 251, 266, 43],
     [199, 40, 499],
     [6552, 213, 197, 132, 6553, 6554, 6555],
     [2870, 6169, 601, 631, 104, 222, 595, 2038, 2272, 2388, 77, 61],
     [6492, 84, 767, 1962, 143, 227, 13, 294, 654, 2991, 87, 6493, 346, 671, 1966],
     [3905, 17, 118, 58, 184, 2, 185],
     [4463, 1817, 51, 712, 2502, 816, 2503, 1048, 4464, 10, 851, 164],
     [1,
      6,
      3283,
      308,
      538,
      367,
      764,
      34,
      448,
      1374,
      2718,
      1180,
      445,
      1312,
      411,
      2709,
      8,
      8],
     [220, 6849, 700, 751, 434, 18, 1687],
     [4489, 1, 568, 4490, 4491, 4492, 1182, 2511, 1367, 149, 153, 1450],
     [3627,
      1,
      468,
      2256,
      568,
      127,
      75,
      781,
      1322,
      713,
      3628,
      228,
      932,
      1300,
      3629,
      228,
      69],
     [4598, 4, 746, 21, 4599, 339, 587, 71],
     [644, 957, 31, 82, 160, 2, 325, 112, 1211, 7728, 1814, 1018, 1481],
     [3763, 1066, 9, 642, 260, 229, 260, 229, 563, 8],
     [3, 139, 178, 410, 2, 108, 36, 8, 1532, 901, 352, 110, 14],
     [7985, 951, 1625],
     [6227,
      1,
      350,
      204,
      117,
      1968,
      6228,
      121,
      423,
      258,
      10,
      6229,
      297,
      1189,
      1978],
     [8160, 1324, 37, 254, 50, 388, 779, 3046, 150, 361, 184, 2],
     [6886, 1639, 6887, 3, 44, 114, 142, 2905, 36, 44, 242, 340],
     [2920, 1, 20, 17, 3868, 7159, 119, 9238, 699, 1301, 46],
     [6365, 1345, 669, 1967, 31, 594, 1361, 605],
     [3340, 771, 1273, 3341, 95, 70],
     [5295, 206, 968, 1156, 206, 5296, 124, 5297],
     [5594, 1, 183, 373, 23, 238, 732, 807, 2614],
     [7598, 7599, 99, 741, 100, 640, 965, 7600, 709, 87],
     [9100, 419, 345, 2, 1023, 880, 102, 1460, 9101, 1108, 9102, 980],
     [35, 1919, 7, 30, 4, 4695, 2865, 1002, 7249],
     [3817, 231, 1130, 2325, 276, 98],
     [8587, 8588, 323, 211, 76, 2355, 279],
     [3521,
      72,
      46,
      1304,
      1626,
      3522,
      3523,
      2226,
      2227,
      21,
      10,
      3524,
      420,
      91,
      2228,
      765,
      14],
     [4514, 315, 1676, 1184, 256, 2261, 39, 4515, 8, 48, 971, 29],
     [8112,
      236,
      33,
      11,
      226,
      113,
      8113,
      188,
      47,
      811,
      2634,
      1805,
      1947,
      774,
      1969,
      3148,
      34,
      1884],
     [4300, 1, 77, 61, 4301, 1432, 38, 4302, 21, 61],
     ...]




```python
import gensim
from gensim.models import Word2Vec
from nltk.tokenize import sent_tokenize, word_tokenize

```

-------------------------------------------------------------------------------------------------------


```python
#Create Embedding_index
embedding_index ={}
with open('glove.txt',encoding='utf-8') as f:
    for line in f:
        values=line.split()
        word=values[0]
        coefs=np.asarray(values[1:],dtype="float32")
        embedding_index[word]=coefs


```


```python
len(embedding_index['the'])
```




    100




```python
type(coefs)
```




    numpy.ndarray




```python
vocab_size
```




    9275




```python
n=2560
```


```python
word_index.items()
```




    dict_items([('says', 1), ('percent', 2), ('state', 3), ('obama', 4), ('tax', 5), ('us', 6), ('president', 7), ('year', 8), ('people', 9), ('would', 10), ('states', 11), ('one', 12), ('million', 13), ('years', 14), ('jobs', 15), ('voted', 16), ('government', 17), ('new', 18), ('texas', 19), ('federal', 20), ('bill', 21), ('health', 22), ('billion', 23), ('law', 24), ('every', 25), ('care', 26), ('pay', 27), ('taxes', 28), ('wisconsin', 29), ('barack', 30), ('said', 31), ('country', 32), ('united', 33), ('since', 34), ('first', 35), ('last', 36), ('rate', 37), ('women', 38), ('get', 39), ('money', 40), ('1', 41), ('scott', 42), ('time', 43), ('budget', 44), ('city', 45), ('security', 46), ('even', 47), ('public', 48), ('cut', 49), ('school', 50), ('obamacare', 51), ('florida', 52), ('americans', 53), ('medicare', 54), ('average', 55), ('house', 56), ('gov', 57), ('spending', 58), ('office', 59), ('republican', 60), ('clinton', 61), ('dont', 62), ('american', 63), ('dollars', 64), ('senate', 65), ('america', 66), ('plan', 67), ('rhode', 68), ('times', 69), ('congress', 70), ('debt', 71), ('social', 72), ('administration', 73), ('cost', 74), ('county', 75), ('obamas', 76), ('hillary', 77), ('could', 78), ('island', 79), ('insurance', 80), ('two', 81), ('nearly', 82), ('four', 83), ('national', 84), ('governor', 85), ('increase', 86), ('world', 87), ('highest', 88), ('10', 89), ('paid', 90), ('program', 91), ('trump', 92), ('history', 93), ('going', 94), ('job', 95), ('stimulus', 96), ('act', 97), ('today', 98), ('per', 99), ('income', 100), ('2', 101), ('nation', 102), ('walker', 103), ('like', 104), ('oil', 105), ('china', 106), ('almost', 107), ('employees', 108), ('vote', 109), ('three', 110), ('actually', 111), ('business', 112), ('use', 113), ('less', 114), ('never', 115), ('day', 116), ('candidate', 117), ('right', 118), ('children', 119), ('work', 120), ('went', 121), ('bush', 122), ('support', 123), ('go', 124), ('economy', 125), ('donald', 126), ('milwaukee', 127), ('rick', 128), ('republicans', 129), ('romney', 130), ('companies', 131), ('gun', 132), ('took', 133), ('passed', 134), ('cuts', 135), ('without', 136), ('schools', 137), ('millions', 138), ('georgia', 139), ('education', 140), ('john', 141), ('half', 142), ('spent', 143), ('system', 144), ('unemployment', 145), ('make', 146), ('rep', 147), ('deficit', 148), ('illegal', 149), ('military', 150), ('workers', 151), ('supported', 152), ('campaign', 153), ('five', 154), ('members', 155), ('court', 156), ('number', 157), ('taxpayers', 158), ('war', 159), ('50', 160), ('largest', 161), ('mitt', 162), ('white', 163), ('medicaid', 164), ('department', 165), ('proposed', 166), ('made', 167), ('25', 168), ('funding', 169), ('much', 170), ('abortion', 171), ('fund', 172), ('college', 173), ('austin', 174), ('sen', 175), ('want', 176), ('ohio', 177), ('lost', 178), ('david', 179), ('thats', 180), ('food', 181), ('20', 182), ('theres', 183), ('40', 184), ('take', 185), ('home', 186), ('deal', 187), ('gas', 188), ('help', 189), ('planned', 190), ('legislation', 191), ('families', 192), ('got', 193), ('control', 194), ('used', 195), ('still', 196), ('put', 197), ('many', 198), ('didnt', 199), ('mayor', 200), ('spend', 201), ('raised', 202), ('doesnt', 203), ('general', 204), ('raise', 205), ('higher', 206), ('marijuana', 207), ('back', 208), ('know', 209), ('weve', 210), ('supports', 211), ('wall', 212), ('cant', 213), ('child', 214), ('men', 215), ('working', 216), ('created', 217), ('george', 218), ('30', 219), ('drug', 220), ('nations', 221), ('crime', 222), ('youre', 223), ('young', 224), ('population', 225), ('allowed', 226), ('12', 227), ('3', 228), ('wage', 229), ('trillion', 230), ('businesses', 231), ('poverty', 232), ('party', 233), ('0', 234), ('give', 235), ('police', 236), ('students', 237), ('private', 238), ('change', 239), ('parenthood', 240), ('buy', 241), ('lowest', 242), ('majority', 243), ('thousands', 244), ('immigrants', 245), ('street', 246), ('democrats', 247), ('reform', 248), ('local', 249), ('amendment', 250), ('guns', 251), ('property', 252), ('making', 253), ('high', 254), ('increased', 255), ('kids', 256), ('away', 257), ('say', 258), ('trade', 259), ('minimum', 260), ('5', 261), ('foreign', 262), ('tried', 263), ('least', 264), ('next', 265), ('done', 266), ('100', 267), ('days', 268), ('nothing', 269), ('research', 270), ('financial', 271), ('costs', 272), ('wants', 273), ('recent', 274), ('veterans', 275), ('virginia', 276), ('iraq', 277), ('rates', 278), ('iran', 279), ('force', 280), ('allow', 281), ('among', 282), ('countries', 283), ('combined', 284), ('economic', 285), ('supreme', 286), ('including', 287), ('six', 288), ('gone', 289), ('black', 290), ('let', 291), ('washington', 292), ('second', 293), ('paying', 294), ('water', 295), ('energy', 296), ('laws', 297), ('services', 298), ('free', 299), ('company', 300), ('2008', 301), ('oregon', 302), ('hes', 303), ('keep', 304), ('ban', 305), ('election', 306), ('sent', 307), ('service', 308), ('mexico', 309), ('60', 310), ('group', 311), ('month', 312), ('family', 313), ('seven', 314), ('single', 315), ('benefits', 316), ('voting', 317), ('nuclear', 318), ('major', 319), ('ever', 320), ('come', 321), ('create', 322), ('paul', 323), ('around', 324), ('small', 325), ('perry', 326), ('border', 327), ('already', 328), ('sex', 329), ('end', 330), ('union', 331), ('growth', 332), ('receive', 333), ('likely', 334), ('congressional', 335), ('political', 336), ('district', 337), ('protect', 338), ('student', 339), ('decade', 340), ('cents', 341), ('groups', 342), ('votes', 343), ('brown', 344), ('90', 345), ('study', 346), ('live', 347), ('worker', 348), ('jersey', 349), ('attorney', 350), ('prices', 351), ('past', 352), ('entire', 353), ('residents', 354), ('called', 355), ('left', 356), ('medical', 357), ('democratic', 358), ('board', 359), ('top', 360), ('35', 361), ('stop', 362), ('came', 363), ('months', 364), ('elected', 365), ('south', 366), ('taxpayer', 367), ('gave', 368), ('started', 369), ('democrat', 370), ('power', 371), ('dollar', 372), ('4', 373), ('way', 374), ('killed', 375), ('build', 376), ('team', 377), ('climate', 378), ('land', 379), ('theyre', 380), ('person', 381), ('18', 382), ('ago', 383), ('marco', 384), ('helped', 385), ('total', 386), ('according', 387), ('level', 388), ('marriage', 389), ('result', 390), ('record', 391), ('doubled', 392), ('pension', 393), ('bills', 394), ('run', 395), ('signed', 396), ('carolina', 397), ('unions', 398), ('part', 399), ('violence', 400), ('man', 401), ('told', 402), ('fire', 403), ('revenue', 404), ('twice', 405), ('big', 406), ('defense', 407), ('amount', 408), ('numbers', 409), ('16', 410), ('makes', 411), ('w', 412), ('11', 413), ('johnson', 414), ('2009', 415), ('religious', 416), ('afghanistan', 417), ('industry', 418), ('well', 419), ('life', 420), ('officers', 421), ('15', 422), ('far', 423), ('presidents', 424), ('special', 425), ('corporations', 426), ('different', 427), ('gdp', 428), ('received', 429), ('ryan', 430), ('2010', 431), ('massachusetts', 432), ('opposed', 433), ('death', 434), ('employers', 435), ('california', 436), ('across', 437), ('500', 438), ('teachers', 439), ('senator', 440), ('providence', 441), ('prison', 442), ('cities', 443), ('eight', 444), ('stamps', 445), ('shows', 446), ('also', 447), ('funds', 448), ('employee', 449), ('retirement', 450), ('70', 451), ('sales', 452), ('whether', 453), ('due', 454), ('worst', 455), ('sector', 456), ('equal', 457), ('rubio', 458), ('hours', 459), ('creation', 460), ('banks', 461), ('ranks', 462), ('a', 463), ('gay', 464), ('muslim', 465), ('2012', 466), ('born', 467), ('chris', 468), ('coverage', 469), ('voters', 470), ('elections', 471), ('cannot', 472), ('debate', 473), ('representatives', 474), ('university', 475), ('north', 476), ('led', 477), ('rape', 478), ('crisis', 479), ('global', 480), ('dc', 481), ('rating', 482), ('legislature', 483), ('terms', 484), ('2000', 485), ('birth', 486), ('romneys', 487), ('immigration', 488), ('thinks', 489), ('walkers', 490), ('tom', 491), ('goes', 492), ('convicted', 493), ('farm', 494), ('mandate', 495), ('believe', 496), ('former', 497), ('refugees', 498), ('parents', 499), ('best', 500), ('middle', 501), ('class', 502), ('programs', 503), ('levels', 504), ('gallon', 505), ('drilling', 506), ('charge', 507), ('2006', 508), ('abortions', 509), ('nancy', 510), ('welfare', 511), ('reagan', 512), ('wanted', 513), ('voter', 514), ('healthcare', 515), ('lower', 516), ('fraud', 517), ('air', 518), ('irs', 519), ('13', 520), ('jim', 521), ('required', 522), ('drivers', 523), ('babies', 524), ('7', 525), ('fewer', 526), ('ebola', 527), ('rights', 528), ('require', 529), ('credit', 530), ('congressman', 531), ('case', 532), ('given', 533), ('refused', 534), ('roughly', 535), ('great', 536), ('news', 537), ('taken', 538), ('justice', 539), ('hundreds', 540), ('order', 541), ('asked', 542), ('pipeline', 543), ('plans', 544), ('side', 545), ('secretary', 546), ('coming', 547), ('overseas', 548), ('member', 549), ('taking', 550), ('decades', 551), ('interest', 552), ('access', 553), ('9', 554), ('ron', 555), ('isis', 556), ('2015', 557), ('trying', 558), ('shut', 559), ('2013', 560), ('hasnt', 561), ('undocumented', 562), ('within', 563), ('week', 564), ('york', 565), ('may', 566), ('sell', 567), ('opponent', 568), ('found', 569), ('community', 570), ('able', 571), ('400', 572), ('lot', 573), ('israel', 574), ('building', 575), ('getting', 576), ('regulations', 577), ('enforcement', 578), ('leadership', 579), ('domestic', 580), ('means', 581), ('earn', 582), ('market', 583), ('governors', 584), ('kill', 585), ('something', 586), ('loan', 587), ('drugs', 588), ('ive', 589), ('promised', 590), ('100000', 591), ('2011', 592), ('biggest', 593), ('coal', 594), ('criminal', 595), ('sherrod', 596), ('current', 597), ('added', 598), ('presidential', 599), ('38', 600), ('find', 601), ('think', 602), ('good', 603), ('term', 604), ('terrorists', 605), ('internet', 606), ('muslims', 607), ('risk', 608), ('homes', 609), ('save', 610), ('floridas', 611), ('yet', 612), ('driving', 613), ('tim', 614), ('pelosi', 615), ('troops', 616), ('san', 617), ('terrorism', 618), ('african', 619), ('brought', 620), ('80', 621), ('ted', 622), ('funded', 623), ('fight', 624), ('fought', 625), ('lead', 626), ('salaries', 627), ('longer', 628), ('wages', 629), ('14', 630), ('things', 631), ('pays', 632), ('old', 633), ('christie', 634), ('proposal', 635), ('affordable', 636), ('christian', 637), ('workforce', 638), ('really', 639), ('greater', 640), ('mike', 641), ('start', 642), ('better', 643), ('report', 644), ('saved', 645), ('auto', 646), ('annual', 647), ('close', 648), ('islamic', 649), ('form', 650), ('growing', 651), ('illegally', 652), ('fox', 653), ('seniors', 654), ('officials', 655), ('enough', 656), ('someone', 657), ('decision', 658), ('possible', 659), ('responsible', 660), ('fbi', 661), ('housing', 662), ('agriculture', 663), ('17', 664), ('leave', 665), ('6', 666), ('alone', 667), ('weeks', 668), ('joe', 669), ('west', 670), ('impact', 671), ('outside', 672), ('mark', 673), ('show', 674), ('earned', 675), ('violent', 676), ('basically', 677), ('senior', 678), ('essentially', 679), ('add', 680), ('freedom', 681), ('size', 682), ('labor', 683), ('problem', 684), ('east', 685), ('open', 686), ('place', 687), ('real', 688), ('significant', 689), ('gasoline', 690), ('caused', 691), ('speaker', 692), ('clear', 693), ('ronald', 694), ('fully', 695), ('moving', 696), ('turn', 697), ('32', 698), ('name', 699), ('common', 700), ('im', 701), ('increases', 702), ('approved', 703), ('agreement', 704), ('legal', 705), ('mccain', 706), ('22', 707), ('die', 708), ('rest', 709), ('licenses', 710), ('bay', 711), ('expansion', 712), ('failed', 713), ('bridge', 714), ('offshore', 715), ('rob', 716), ('sign', 717), ('none', 718), ('20000', 719), ('illinois', 720), ('code', 721), ('wealth', 722), ('see', 723), ('worked', 724), ('balanced', 725), ('using', 726), ('wealthy', 727), ('loans', 728), ('behind', 729), ('turned', 730), ('28', 731), ('investment', 732), ('citizens', 733), ('effect', 734), ('politicians', 735), ('changed', 736), ('email', 737), ('couldnt', 738), ('early', 739), ('27', 740), ('capita', 741), ('candidates', 742), ('bank', 743), ('plant', 744), ('points', 745), ('signs', 746), ('allen', 747), ('bonds', 748), ('provide', 749), ('300', 750), ('cause', 751), ('liberal', 752), ('keystone', 753), ('full', 754), ('warming', 755), ('gets', 756), ('barrett', 757), ('virtually', 758), ('felons', 759), ('millionaires', 760), ('37', 761), ('fees', 762), ('provides', 763), ('aid', 764), ('75', 765), ('tens', 766), ('science', 767), ('libya', 768), ('living', 769), ('lawmakers', 770), ('isnt', 771), ('cancer', 772), ('200', 773), ('banned', 774), ('discrimination', 775), ('served', 776), ('large', 777), ('colorado', 778), ('test', 779), ('primary', 780), ('executive', 781), ('folks', 782), ('premiums', 783), ('19', 784), ('id', 785), ('third', 786), ('leader', 787), ('eric', 788), ('steve', 789), ('space', 790), ('fact', 791), ('reid', 792), ('safety', 793), ('alcohol', 794), ('deer', 795), ('line', 796), ('tuition', 797), ('census', 798), ('bureau', 799), ('personal', 800), ('electric', 801), ('profits', 802), ('rules', 803), ('similar', 804), ('woman', 805), ('cuba', 806), ('along', 807), ('price', 808), ('expanding', 809), ('cicilline', 810), ('though', 811), ('deaths', 812), ('cash', 813), ('senators', 814), ('eliminate', 815), ('lose', 816), ('giving', 817), ('development', 818), ('150', 819), ('point', 820), ('tommy', 821), ('thompson', 822), ('claim', 823), ('drop', 824), ('atlanta', 825), ('mr', 826), ('raising', 827), ('spends', 828), ('bipartisan', 829), ('harry', 830), ('disease', 831), ('poll', 832), ('shootings', 833), ('soccer', 834), ('benefit', 835), ('look', 836), ('popular', 837), ('wisconsins', 838), ('prescription', 839), ('collective', 840), ('bargaining', 841), ('annually', 842), ('median', 843), ('stopped', 844), ('true', 845), ('poor', 846), ('involved', 847), ('data', 848), ('agency', 849), ('ability', 850), ('forced', 851), ('background', 852), ('center', 853), ('requires', 854), ('approximately', 855), ('firefighters', 856), ('911', 857), ('attacks', 858), ('places', 859), ('council', 860), ('worth', 861), ('reduced', 862), ('influence', 863), ('pot', 864), ('10000', 865), ('larger', 866), ('islands', 867), ('currently', 868), ('41', 869), ('guy', 870), ('samesex', 871), ('takes', 872), ('ceo', 873), ('ri', 874), ('construction', 875), ('send', 876), ('unemployed', 877), ('thing', 878), ('projects', 879), ('cases', 880), ('authority', 881), ('seen', 882), ('bring', 883), ('thanks', 884), ('districts', 885), ('fiscal', 886), ('suggested', 887), ('grown', 888), ('miles', 889), ('2005', 890), ('mass', 891), ('offenders', 892), ('became', 893), ('nursing', 894), ('minority', 895), ('losing', 896), ('45', 897), ('account', 898), ('must', 899), ('cell', 900), ('risen', 901), ('crimes', 902), ('mandatory', 903), ('2014', 904), ('idea', 905), ('95', 906), ('privatize', 907), ('died', 908), ('resulted', 909), ('indiana', 910), ('kaine', 911), ('announced', 912), ('intelligence', 913), ('agencies', 914), ('arizona', 915), ('repeal', 916), ('arrested', 917), ('theyve', 918), ('gulf', 919), ('either', 920), ('running', 921), ('recall', 922), ('brothers', 923), ('avoid', 924), ('ranked', 925), ('whole', 926), ('management', 927), ('24', 928), ('georgias', 929), ('hispanic', 930), ('protections', 931), ('separate', 932), ('protection', 933), ('tammy', 934), ('baldwin', 935), ('extra', 936), ('carbon', 937), ('emissions', 938), ('rid', 939), ('hour', 940), ('watch', 941), ('list', 942), ('hospital', 943), ('legally', 944), ('walmart', 945), ('interests', 946), ('released', 947), ('dan', 948), ('checks', 949), ('includes', 950), ('abuse', 951), ('revenues', 952), ('nine', 953), ('instead', 954), ('individual', 955), ('returns', 956), ('recently', 957), ('smaller', 958), ('car', 959), ('anyone', 960), ('staff', 961), ('balance', 962), ('need', 963), ('sequester', 964), ('probably', 965), ('sandy', 966), ('zero', 967), ('beer', 968), ('head', 969), ('ordered', 970), ('assistance', 971), ('carry', 972), ('recipients', 973), ('issue', 974), ('wrote', 975), ('constitution', 976), ('investigation', 977), ('russ', 978), ('feingold', 979), ('households', 980), ('beliefs', 981), ('leading', 982), ('check', 983), ('800', 984), ('household', 985), ('toward', 986), ('independent', 987), ('flag', 988), ('mary', 989), ('ken', 990), ('coast', 991), ('doctors', 992), ('firearm', 993), ('anything', 994), ('together', 995), ('hurricane', 996), ('accused', 997), ('sold', 998), ('foreclosure', 999), ('several', 1000), ('budgets', 1001), ('tv', 1002), ('hate', 1003), ('terrorist', 1004), ('aliens', 1005), ('civil', 1006), ('evidence', 1007), ('language', 1008), ('russia', 1009), ('countrys', 1010), ('double', 1011), ('central', 1012), ('return', 1013), ('kasich', 1014), ('accepted', 1015), ('gives', 1016), ('girl', 1017), ('call', 1018), ('parks', 1019), ('victims', 1020), ('consumer', 1021), ('wind', 1022), ('felony', 1023), ('saying', 1024), ('stadium', 1025), ('syrian', 1026), ('billions', 1027), ('newt', 1028), ('gingrich', 1029), ('2007', 1030), ('career', 1031), ('net', 1032), ('gop', 1033), ('ferguson', 1034), ('29', 1035), ('owns', 1036), ('reforms', 1037), ('breaks', 1038), ('estimates', 1039), ('rail', 1040), ('kind', 1041), ('treasury', 1042), ('decided', 1043), ('homeless', 1044), ('immigrant', 1045), ('believes', 1046), ('leaders', 1047), ('subsidies', 1048), ('holding', 1049), ('traffic', 1050), ('except', 1051), ('clintons', 1052), ('barbara', 1053), ('backed', 1054), ('grocery', 1055), ('alex', 1056), ('benghazi', 1057), ('30000', 1058), ('survey', 1059), ('serving', 1060), ('200000', 1061), ('minnesota', 1062), ('experience', 1063), ('massive', 1064), ('emails', 1065), ('twothirds', 1066), ('testing', 1067), ('position', 1068), ('americas', 1069), ('houston', 1070), ('dallas', 1071), ('changes', 1072), ('bp', 1073), ('allows', 1074), ('opposes', 1075), ('meet', 1076), ('human', 1077), ('bureaucrats', 1078), ('42', 1079), ('manufacturing', 1080), ('india', 1081), ('ad', 1082), ('letter', 1083), ('strickland', 1084), ('koch', 1085), ('bernie', 1086), ('pregnancies', 1087), ('efforts', 1088), ('england', 1089), ('light', 1090), ('project', 1091), ('murder', 1092), ('owned', 1093), ('soldiers', 1094), ('adults', 1095), ('departments', 1096), ('fighting', 1097), ('54', 1098), ('jail', 1099), ('drive', 1100), ('corporate', 1101), ('infrastructure', 1102), ('purchase', 1103), ('rise', 1104), ('ballot', 1105), ('increasing', 1106), ('another', 1107), ('grew', 1108), ('statewide', 1109), ('media', 1110), ('measure', 1111), ('portman', 1112), ('controlled', 1113), ('substance', 1114), ('guard', 1115), ('developed', 1116), ('expand', 1117), ('prevent', 1118), ('reduction', 1119), ('counties', 1120), ('reduce', 1121), ('3000', 1122), ('bankruptcy', 1123), ('sanders', 1124), ('charlie', 1125), ('agenda', 1126), ('africa', 1127), ('looking', 1128), ('contracts', 1129), ('closing', 1130), ('capandtrade', 1131), ('pump', 1132), ('shown', 1133), ('else', 1134), ('chief', 1135), ('womens', 1136), ('1000', 1137), ('ruled', 1138), ('egypt', 1139), ('creating', 1140), ('offices', 1141), ('8', 1142), ('animals', 1143), ('read', 1144), ('area', 1145), ('extremists', 1146), ('sure', 1147), ('disaster', 1148), ('costing', 1149), ('mostly', 1150), ('constitutional', 1151), ('block', 1152), ('salary', 1153), ('77', 1154), ('age', 1155), ('production', 1156), ('friends', 1157), ('commission', 1158), ('policies', 1159), ('happen', 1160), ('nationally', 1161), ('lobbyists', 1162), ('payment', 1163), ('deny', 1164), ('opinion', 1165), ('outlaw', 1166), ('bond', 1167), ('female', 1168), ('problems', 1169), ('policy', 1170), ('nfl', 1171), ('2001', 1172), ('navy', 1173), ('removed', 1174), ('phone', 1175), ('shot', 1176), ('crist', 1177), ('polls', 1178), ('mean', 1179), ('sale', 1180), ('emergency', 1181), ('facebook', 1182), ('jeff', 1183), ('couple', 1184), ('teacher', 1185), ('previous', 1186), ('officer', 1187), ('300000', 1188), ('personally', 1189), ('michael', 1190), ('politics', 1191), ('patrick', 1192), ('harm', 1193), ('huge', 1194), ('wouldnt', 1195), ('protesters', 1196), ('cars', 1197), ('adopted', 1198), ('lives', 1199), ('charges', 1200), ('zika', 1201), ('transportation', 1202), ('bankrupt', 1203), ('tobacco', 1204), ('solar', 1205), ('girls', 1206), ('perrys', 1207), ('mccollum', 1208), ('5000', 1209), ('stand', 1210), ('owners', 1211), ('individuals', 1212), ('hospitals', 1213), ('fly', 1214), ('earth', 1215), ('baby', 1216), ('serious', 1217), ('nomination', 1218), ('hook', 1219), ('needs', 1220), ('virus', 1221), ('al', 1222), ('va', 1223), ('issued', 1224), ('invented', 1225), ('produce', 1226), ('23', 1227), ('held', 1228), ('information', 1229), ('closed', 1230), ('compared', 1231), ('abusers', 1232), ('road', 1233), ('presidency', 1234), ('surplus', 1235), ('defund', 1236), ('courts', 1237), ('enacted', 1238), ('role', 1239), ('decisions', 1240), ('eliminated', 1241), ('contract', 1242), ('tea', 1243), ('inside', 1244), ('82', 1245), ('rich', 1246), ('faster', 1247), ('highspeed', 1248), ('13000', 1249), ('bailout', 1250), ('committee', 1251), ('65', 1252), ('veteran', 1253), ('address', 1254), ('employer', 1255), ('universities', 1256), ('gubernatorial', 1257), ('cutting', 1258), ('manufacturers', 1259), ('credits', 1260), ('probe', 1261), ('appointed', 1262), ('politician', 1263), ('lottery', 1264), ('threat', 1265), ('significantly', 1266), ('sink', 1267), ('requests', 1268), ('ninety', 1269), ('agree', 1270), ('specifically', 1271), ('steps', 1272), ('schedule', 1273), ('fatalities', 1274), ('caught', 1275), ('commerce', 1276), ('performed', 1277), ('screenings', 1278), ('providences', 1279), ('port', 1280), ('closer', 1281), ('europe', 1282), ('requiring', 1283), ('assault', 1284), ('graduate', 1285), ('views', 1286), ('invested', 1287), ('training', 1288), ('began', 1289), ('el', 1290), ('paso', 1291), ('antonio', 1292), ('latinos', 1293), ('youth', 1294), ('telling', 1295), ('michigan', 1296), ('employment', 1297), ('core', 1298), ('ads', 1299), ('colleges', 1300), ('airport', 1301), ('identification', 1302), ('fifth', 1303), ('projected', 1304), ('website', 1305), ('near', 1306), ('twitter', 1307), ('official', 1308), ('midwest', 1309), ('retire', 1310), ('resources', 1311), ('products', 1312), ('shutdown', 1313), ('threatened', 1314), ('portland', 1315), ('yes', 1316), ('measures', 1317), ('christmas', 1318), ('registered', 1319), ('repeatedly', 1320), ('pollution', 1321), ('race', 1322), ('syria', 1323), ('pass', 1324), ('forward', 1325), ('legislative', 1326), ('leaving', 1327), ('voucher', 1328), ('despite', 1329), ('pence', 1330), ('childrens', 1331), ('reason', 1332), ('heroin', 1333), ('stamp', 1334), ('exactly', 1335), ('saudi', 1336), ('arabia', 1337), ('expanded', 1338), ('treatment', 1339), ('admits', 1340), ('areas', 1341), ('consent', 1342), ('blacks', 1343), ('whites', 1344), ('claims', 1345), ('modern', 1346), ('havent', 1347), ('generation', 1348), ('transit', 1349), ('fastest', 1350), ('crossing', 1351), ('33', 1352), ('destroy', 1353), ('50000', 1354), ('fung', 1355), ('met', 1356), ('governmentrun', 1357), ('anywhere', 1358), ('talk', 1359), ('socalled', 1360), ('dangerous', 1361), ('overwhelmingly', 1362), ('cap', 1363), ('ceos', 1364), ('conducted', 1365), ('revealed', 1366), ('receiving', 1367), ('michelle', 1368), ('43', 1369), ('fastestgrowing', 1370), ('attempted', 1371), ('systems', 1372), ('lowincome', 1373), ('operations', 1374), ('banning', 1375), ('basic', 1376), ('hold', 1377), ('advanced', 1378), ('55', 1379), ('male', 1380), ('texans', 1381), ('2003', 1382), ('fix', 1383), ('truth', 1384), ('pulled', 1385), ('responsibility', 1386), ('67', 1387), ('effort', 1388), ('remove', 1389), ('conservative', 1390), ('parties', 1391), ('environmental', 1392), ('500000', 1393), ('sexual', 1394), ('issues', 1395), ('fine', 1396), ('62', 1397), ('boards', 1398), ('terry', 1399), ('mcauliffe', 1400), ('inflation', 1401), ('saving', 1402), ('thought', 1403), ('comes', 1404), ('serve', 1405), ('positions', 1406), ('dane', 1407), ('kathleen', 1408), ('store', 1409), ('modified', 1410), ('scheme', 1411), ('photo', 1412), ('tell', 1413), ('hired', 1414), ('saw', 1415), ('epa', 1416), ('49', 1417), ('6000', 1418), ('ryans', 1419), ('privatesector', 1420), ('equivalent', 1421), ('move', 1422), ('park', 1423), ('taveras', 1424), ('deals', 1425), ('town', 1426), ('little', 1427), ('bushs', 1428), ('reducing', 1429), ('unlike', 1430), ('investigations', 1431), ('attacked', 1432), ('advocated', 1433), ('terror', 1434), ('restoration', 1435), ('lakes', 1436), ('brother', 1437), ('alqaida', 1438), ('miami', 1439), ('combat', 1440), ('dozens', 1441), ('wont', 1442), ('offered', 1443), ('include', 1444), ('greg', 1445), ('abbott', 1446), ('contraception', 1447), ('easier', 1448), ('extreme', 1449), ('contributions', 1450), ('privatizing', 1451), ('citizenship', 1452), ('facility', 1453), ('package', 1454), ('laid', 1455), ('lie', 1456), ('personnel', 1457), ('represents', 1458), ('diseases', 1459), ('committed', 1460), ('occupy', 1461), ('taxpayerfunded', 1462), ('killing', 1463), ('36', 1464), ('jews', 1465), ('un', 1466), ('sanctions', 1467), ('associated', 1468), ('homeland', 1469), ('table', 1470), ('games', 1471), ('youve', 1472), ('47', 1473), ('studios', 1474), ('falls', 1475), ('period', 1476), ('pat', 1477), ('earmarks', 1478), ('rubios', 1479), ('prayer', 1480), ('regulatory', 1481), ('bottom', 1482), ('forces', 1483), ('disabled', 1484), ('loophole', 1485), ('missouri', 1486), ('stem', 1487), ('plane', 1488), ('africanamerican', 1489), ('bucks', 1490), ('governments', 1491), ('identify', 1492), ('body', 1493), ('childhood', 1494), ('fired', 1495), ('2002', 1496), ('spread', 1497), ('sheriff', 1498), ('xl', 1499), ('finance', 1500), ('institution', 1501), ('communities', 1502), ('unpopular', 1503), ('rule', 1504), ('richard', 1505), ('skyrocketing', 1506), ('99', 1507), ('virginias', 1508), ('carrying', 1509), ('chinese', 1510), ('actions', 1511), ('bible', 1512), ('amnesty', 1513), ('dog', 1514), ('estimated', 1515), ('headed', 1516), ('ask', 1517), ('parts', 1518), ('proposals', 1519), ('football', 1520), ('waiting', 1521), ('overall', 1522), ('trafficking', 1523), ('centers', 1524), ('warning', 1525), ('sources', 1526), ('records', 1527), ('truck', 1528), ('uninsured', 1529), ('ethanol', 1530), ('teach', 1531), ('percentage', 1532), ('daniel', 1533), ('tourism', 1534), ('main', 1535), ('missed', 1536), ('renacci', 1537), ('josh', 1538), ('sharia', 1539), ('everybody', 1540), ('quadrupled', 1541), ('delivered', 1542), ('islam', 1543), ('uses', 1544), ('pensions', 1545), ('luxury', 1546), ('platform', 1547), ('source', 1548), ('deadly', 1549), ('choice', 1550), ('disability', 1551), ('korea', 1552), ('vacation', 1553), ('horse', 1554), ('action', 1555), ('opposition', 1556), ('reported', 1557), ('firms', 1558), ('available', 1559), ('built', 1560), ('nonviolent', 1561), ('technical', 1562), ('earning', 1563), ('lived', 1564), ('margin', 1565), ('88', 1566), ('standards', 1567), ('loretta', 1568), ('lynch', 1569), ('corruption', 1570), ('lots', 1571), ('throughout', 1572), ('poses', 1573), ('difference', 1574), ('publicly', 1575), ('approval', 1576), ('ratings', 1577), ('mexican', 1578), ('authorities', 1579), ('properties', 1580), ('illegals', 1581), ('mixed', 1582), ('1300', 1583), ('fish', 1584), ('natural', 1585), ('hike', 1586), ('prevention', 1587), ('wiped', 1588), ('tries', 1589), ('respect', 1590), ('permanent', 1591), ('rifles', 1592), ('paycheck', 1593), ('ceiling', 1594), ('hands', 1595), ('debbie', 1596), ('concluded', 1597), ('pushed', 1598), ('threatens', 1599), ('loss', 1600), ('openly', 1601), ('coach', 1602), ('drink', 1603), ('kucinich', 1604), ('kicking', 1605), ('scores', 1606), ('bought', 1607), ('hedge', 1608), ('managers', 1609), ('campuses', 1610), ('destroyed', 1611), ('flood', 1612), ('organized', 1613), ('owner', 1614), ('foreclosures', 1615), ('teen', 1616), ('grant', 1617), ('becoming', 1618), ('abolishing', 1619), ('83', 1620), ('pentagon', 1621), ('classrooms', 1622), ('sharron', 1623), ('angle', 1624), ('prisoners', 1625), ('remain', 1626), ('pages', 1627), ('towards', 1628), ('dating', 1629), ('kinds', 1630), ('battle', 1631), ('males', 1632), ('minorities', 1633), ('amazing', 1634), ('amendments', 1635), ('hunters', 1636), ('wildlife', 1637), ('tampa', 1638), ('borrowing', 1639), ('dime', 1640), ('duty', 1641), ('response', 1642), ('heard', 1643), ('86', 1644), ('praised', 1645), ('delayed', 1646), ('34000', 1647), ('rival', 1648), ('completely', 1649), ('deborah', 1650), ('counted', 1651), ('mortality', 1652), ('press', 1653), ('outofstate', 1654), ('date', 1655), ('refusal', 1656), ('search', 1657), ('night', 1658), ('grants', 1659), ('81', 1660), ('adviser', 1661), ('irans', 1662), ('mitch', 1663), ('century', 1664), ('limit', 1665), ('share', 1666), ('dad', 1667), ('bar', 1668), ('instate', 1669), ('certain', 1670), ('islanders', 1671), ('reached', 1672), ('citizen', 1673), ('trust', 1674), ('milk', 1675), ('parent', 1676), ('book', 1677), ('card', 1678), ('gap', 1679), ('mothers', 1680), ('stay', 1681), ('secondhighest', 1682), ('prolife', 1683), ('robert', 1684), ('king', 1685), ('win', 1686), ('hampshire', 1687), ('calls', 1688), ('usmexico', 1689), ('afford', 1690), ('penalties', 1691), ('allan', 1692), ('everyone', 1693), ('bringing', 1694), ('basis', 1695), ('gotten', 1696), ('driven', 1697), ('sponsored', 1698), ('jeanne', 1699), ('shaheen', 1700), ('green', 1701), ('savings', 1702), ('red', 1703), ('couples', 1704), ('pursue', 1705), ('usually', 1706), ('takeover', 1707), ('sitting', 1708), ('neither', 1709), ('negotiating', 1710), ('southeast', 1711), ('victim', 1712), ('session', 1713), ('hb2', 1714), ('1800', 1715), ('21', 1716), ('wait', 1717), ('adams', 1718), ('approve', 1719), ('word', 1720), ('spoken', 1721), ('graduation', 1722), ('reading', 1723), ('oversight', 1724), ('recession', 1725), ('commit', 1726), ('nato', 1727), ('greece', 1728), ('wealthiest', 1729), ('review', 1730), ('communist', 1731), ('trip', 1732), ('supervisors', 1733), ('lois', 1734), ('mining', 1735), ('judges', 1736), ('based', 1737), ('orientation', 1738), ('columbia', 1739), ('the', 1740), ('cocaine', 1741), ('wendy', 1742), ('davis', 1743), ('habits', 1744), ('unarmed', 1745), ('attorneys', 1746), ('fort', 1747), ('hood', 1748), ('initially', 1749), ('latino', 1750), ('faith', 1751), ('showing', 1752), ('payments', 1753), ('referring', 1754), ('genetically', 1755), ('purchased', 1756), ('prosecutors', 1757), ('figure', 1758), ('riding', 1759), ('safe', 1760), ('rifle', 1761), ('association', 1762), ('slaves', 1763), ('agents', 1764), ('lincoln', 1765), ('incomes', 1766), ('drinking', 1767), ('rising', 1768), ('defend', 1769), ('treatments', 1770), ('seats', 1771), ('straight', 1772), ('hit', 1773), ('sotomayor', 1774), ('lawsuit', 1775), ('judicial', 1776), ('rank', 1777), ('bad', 1778), ('super', 1779), ('bowl', 1780), ('stadiums', 1781), ('ticket', 1782), ('sept', 1783), ('160', 1784), ('continues', 1785), ('85', 1786), ('embassy', 1787), ('60000', 1788), ('payroll', 1789), ('1947', 1790), ('showed', 1791), ('studies', 1792), ('righttowork', 1793), ('difficult', 1794), ('obtain', 1795), ('tolls', 1796), ('train', 1797), ('recovery', 1798), ('operation', 1799), ('favors', 1800), ('compensation', 1801), ('turnpike', 1802), ('release', 1803), ('assad', 1804), ('chemical', 1805), ('weapons', 1806), ('lobbyist', 1807), ('asking', 1808), ('conditions', 1809), ('atlantic', 1810), ('visit', 1811), ('74', 1812), ('door', 1813), ('hiring', 1814), ('raped', 1815), ('sachs', 1816), ('choose', 1817), ('stood', 1818), ('front', 1819), ('kills', 1820), ('incidents', 1821), ('following', 1822), ('suspects', 1823), ('arrests', 1824), ('submit', 1825), ('2500', 1826), ('offense', 1827), ('mother', 1828), ('patient', 1829), ('region', 1830), ('campaigning', 1831), ('limits', 1832), ('trillions', 1833), ('reports', 1834), ('residency', 1835), ('mental', 1836), ('dewhurst', 1837), ('later', 1838), ('piece', 1839), ('nominee', 1840), ('lee', 1841), ('400000', 1842), ('nondiscrimination', 1843), ('april', 1844), ('gen', 1845), ('alltime', 1846), ('camps', 1847), ('designed', 1848), ('collect', 1849), ('permit', 1850), ('rose', 1851), ('largely', 1852), ('southwest', 1853), ('tremendous', 1854), ('sells', 1855), ('director', 1856), ('attack', 1857), ('touch', 1858), ('oregonians', 1859), ('receives', 1860), ('knows', 1861), ('jack', 1862), ('refinance', 1863), ('historically', 1864), ('fairness', 1865), ('policemen', 1866), ('bathroom', 1867), ('murphy', 1868), ('153', 1869), ('arena', 1870), ('depend', 1871), ('citation', 1872), ('holds', 1873), ('moved', 1874), ('removing', 1875), ('registration', 1876), ('fifty', 1877), ('polling', 1878), ('ohioans', 1879), ('africanamericans', 1880), ('expected', 1881), ('tied', 1882), ('ohios', 1883), ('1993', 1884), ('weaken', 1885), ('doddfrank', 1886), ('classroom', 1887), ('meaning', 1888), ('espionage', 1889), ('israeli', 1890), ('rightwing', 1891), ('base', 1892), ('feed', 1893), ('generated', 1894), ('mens', 1895), ('disney', 1896), ('hire', 1897), ('detroit', 1898), ('53', 1899), ('supposed', 1900), ('lloyd', 1901), ('tarp', 1902), ('transpacific', 1903), ('partnership', 1904), ('poison', 1905), ('cornilles', 1906), ('cheating', 1907), ('63', 1908), ('expenses', 1909), ('fracking', 1910), ('depression', 1911), ('secret', 1912), ('theft', 1913), ('somebody', 1914), ('requested', 1915), ('expensive', 1916), ('34', 1917), ('sunday', 1918), ('interview', 1919), ('mostlytrue', 1920), ('an', 1921), ('clarke', 1922), ('lawyer', 1923), ('improve', 1924), ('soviet', 1925), ('mill', 1926), ('k12', 1927), ('opportunity', 1928), ('improving', 1929), ('screening', 1930), ('procedures', 1931), ('98', 1932), ('plus', 1933), ('endorsed', 1934), ('costly', 1935), ('particularly', 1936), ('maybe', 1937), ('26', 1938), ('blocked', 1939), ('set', 1940), ('scandal', 1941), ('francisco', 1942), ('370000', 1943), ('rather', 1944), ('arrest', 1945), ('suspected', 1946), ('weapon', 1947), ('putting', 1948), ('tough', 1949), ('demanding', 1950), ('smith', 1951), ('cold', 1952), ('undergo', 1953), ('lets', 1954), ('cdc', 1955), ('armed', 1956), ('nobody', 1957), ('negative', 1958), ('impose', 1959), ('conduct', 1960), ('repealing', 1961), ('foundation', 1962), ('travis', 1963), ('sens', 1964), ('heart', 1965), ('brain', 1966), ('biden', 1967), ('jon', 1968), ('international', 1969), ('sandwich', 1970), ('allowing', 1971), ('become', 1972), ('van', 1973), ('capital', 1974), ('cheese', 1975), ('reduces', 1976), ('whatsoever', 1977), ('agreed', 1978), ('firearms', 1979), ('commissioner', 1980), ('radical', 1981), ('roy', 1982), ('standard', 1983), ('citys', 1984), ('relief', 1985), ('simply', 1986), ('warner', 1987), ('brazil', 1988), ('hed', 1989), ('meeting', 1990), ('360', 1991), ('online', 1992), ('franklin', 1993), ('roosevelt', 1994), ('perhaps', 1995), ('track', 1996), ('kelly', 1997), ('elect', 1998), ('sheriffs', 1999), ('wisconsinites', 2000), ('earns', 2001), ('amounts', 2002), ('questioning', 2003), ('minutes', 2004), ('regarding', 2005), ('progressive', 2006), ('division', 2007), ('classes', 2008), ('guantanamo', 2009), ('islamist', 2010), ('louisiana', 2011), ('begin', 2012), ('participate', 2013), ('denver', 2014), ('pounds', 2015), ('known', 2016), ('adding', 2017), ('72', 2018), ('legitimate', 2019), ('outreach', 2020), ('covered', 2021), ('accidents', 2022), ('webster', 2023), ('passage', 2024), ('decreases', 2025), ('bob', 2026), ('mandel', 2027), ('dropped', 2028), ('fair', 2029), ('250000', 2030), ('roe', 2031), ('diversity', 2032), ('unprecedented', 2033), ('step', 2034), ('strong', 2035), ('kagan', 2036), ('risky', 2037), ('activity', 2038), ('towns', 2039), ('son', 2040), ('gains', 2041), ('lines', 2042), ('burr', 2043), ('150000', 2044), ('automobile', 2045), ('incest', 2046), ('250', 2047), ('drunk', 2048), ('solyndra', 2049), ('unconstitutional', 2050), ('break', 2051), ('hussein', 2052), ('quarter', 2053), ('delaying', 2054), ('long', 2055), ('newport', 2056), ('generous', 2057), ('vacancy', 2058), ('actual', 2059), ('wives', 2060), ('subject', 2061), ('deciding', 2062), ('pretty', 2063), ('treaty', 2064), ('vaccine', 2065), ('katrina', 2066), ('lunch', 2067), ('words', 2068), ('debates', 2069), ('draft', 2070), ('langevin', 2071), ('books', 2072), ('richmond', 2073), ('color', 2074), ('deliberately', 2075), ('bunch', 2076), ('causes', 2077), ('clinics', 2078), ('low', 2079), ('decreased', 2080), ('charged', 2081), ('los', 2082), ('angeles', 2083), ('discretionary', 2084), ('campaigns', 2085), ('mo', 2086), ('successfully', 2087), ('shops', 2088), ('commercial', 2089), ('baseball', 2090), ('progress', 2091), ('regulate', 2092), ('gained', 2093), ('solid', 2094), ('longest', 2095), ('racing', 2096), ('denied', 2097), ('fast', 2098), ('pregnant', 2099), ('arms', 2100), ('flying', 2101), ('barely', 2102), ('permission', 2103), ('medication', 2104), ('secretly', 2105), ('slush', 2106), ('intended', 2107), ('failing', 2108), ('1990', 2109), ('incarceration', 2110), ('degrees', 2111), ('corner', 2112), ('parttime', 2113), ('7000', 2114), ('situation', 2115), ('counterparts', 2116), ('shes', 2117), ('face', 2118), ('1985', 2119), ('winning', 2120), ('tougher', 2121), ('advance', 2122), ('1992', 2123), ('whitewater', 2124), ('commissions', 2125), ('buono', 2126), ('utilities', 2127), ('phones', 2128), ('drove', 2129), ('latin', 2130), ('homosexual', 2131), ('latest', 2132), ('prosecution', 2133), ('mccains', 2134), ('52', 2135), ('stevens', 2136), ('600', 2137), ('seat', 2138), ('gaza', 2139), ('dr', 2140), ('steven', 2141), ('european', 2142), ('incumbent', 2143), ('scientists', 2144), ('camp', 2145), ('hud', 2146), ('howell', 2147), ('bathrooms', 2148), ('drunken', 2149), ('dues', 2150), ('doubledigit', 2151), ('premium', 2152), ('interior', 2153), ('fresh', 2154), ('eastern', 2155), ('defeat', 2156), ('operating', 2157), ('eighty', 2158), ('fed', 2159), ('livestock', 2160), ('cabinet', 2161), ('appointees', 2162), ('gender', 2163), ('wasserman', 2164), ('schultz', 2165), ('campus', 2166), ('appeared', 2167), ('nonpartisan', 2168), ('800000', 2169), ('institutions', 2170), ('status', 2171), ('advocate', 2172), ('tainted', 2173), ('91', 2174), ('dream', 2175), ('requirement', 2176), ('registrations', 2177), ('48', 2178), ('basketball', 2179), ('forever', 2180), ('admitted', 2181), ('directly', 2182), ('carries', 2183), ('marcy', 2184), ('kaptur', 2185), ('dennis', 2186), ('ties', 2187), ('suits', 2188), ('turkey', 2189), ('picture', 2190), ('frank', 2191), ('relative', 2192), ('everything', 2193), ('host', 2194), ('factors', 2195), ('dealers', 2196), ('museum', 2197), ('mitchell', 2198), ('protected', 2199), ('threatening', 2200), ('civilian', 2201), ('2016', 2202), ('attributed', 2203), ('borders', 2204), ('faculty', 2205), ('clackamas', 2206), ('commissioners', 2207), ('countys', 2208), ('trimets', 2209), ('199', 2210), ('cantor', 2211), ('austins', 2212), ('whopping', 2213), ('privately', 2214), ('ages', 2215), ('diploma', 2216), ('poizner', 2217), ('nonprofit', 2218), ('lying', 2219), ('daily', 2220), ('consistently', 2221), ('video', 2222), ('raises', 2223), ('meal', 2224), ('1973', 2225), ('whitehouse', 2226), ('cosponsored', 2227), ('additional', 2228), ('lawsuits', 2229), ('metropolitan', 2230), ('childless', 2231), ('begging', 2232), ('galveston', 2233), ('meant', 2234), ('earlier', 2235), ('strip', 2236), ('chambers', 2237), ('granted', 2238), ('exemptions', 2239), ('expenditures', 2240), ('whos', 2241), ('developers', 2242), ('bars', 2243), ('restaurants', 2244), ('sea', 2245), ('especially', 2246), ('license', 2247), ('renewal', 2248), ('process', 2249), ('walking', 2250), ('rejecting', 2251), ('underwear', 2252), ('bomber', 2253), ('this', 2254), ('extremist', 2255), ('abele', 2256), ('assaults', 2257), ('remaining', 2258), ('historic', 2259), ('fasttrack', 2260), ('easily', 2261), ('future', 2262), ('mail', 2263), ('timing', 2264), ('dogs', 2265), ('cats', 2266), ('770', 2267), ('expectancy', 2268), ('keeping', 2269), ('permits', 2270), ('tells', 2271), ('google', 2272), ('listed', 2273), ('killer', 2274), ('deeper', 2275), ('venture', 2276), ('partner', 2277), ('revolutionary', 2278), ('mcconnell', 2279), ('improvements', 2280), ('newly', 2281), ('mcdonalds', 2282), ('toxic', 2283), ('mom', 2284), ('branch', 2285), ('thirdtrimester', 2286), ('sanctuary', 2287), ('deficitfederalbudget', 2288), ('ideas', 2289), ('legislators', 2290), ('estimate', 2291), ('unionization', 2292), ('follow', 2293), ('proportion', 2294), ('entering', 2295), ('ways', 2296), ('haiti', 2297), ('gunfire', 2298), ('ground', 2299), ('aig', 2300), ('regime', 2301), ('backs', 2302), ('puerto', 2303), ('rico', 2304), ('hiv', 2305), ('lands', 2306), ('lifted', 2307), ('moratorium', 2308), ('yesterday', 2309), ('native', 2310), ('blood', 2311), ('ed', 2312), ('promoted', 2313), ('recommends', 2314), ('mich', 2315), ('sports', 2316), ('rely', 2317), ('described', 2318), ('prochoice', 2319), ('learn', 2320), ('iranian', 2321), ('gays', 2322), ('demand', 2323), ('56', 2324), ('opening', 2325), ('apprehended', 2326), ('metro', 2327), ('donated', 2328), ('applied', 2329), ('stores', 2330), ('hassan', 2331), ('hundred', 2332), ('mandels', 2333), ('busy', 2334), ('representation', 2335), ('friend', 2336), ('quickly', 2337), ('pers', 2338), ('sport', 2339), ('medicine', 2340), ('blue', 2341), ('extortion', 2342), ('apply', 2343), ('egyptians', 2344), ('oppose', 2345), ('provision', 2346), ('register', 2347), ('inmates', 2348), ('committing', 2349), ('stock', 2350), ('congressmen', 2351), ('felonies', 2352), ('rejected', 2353), ('vehicles', 2354), ('negotiations', 2355), ('49th', 2356), ('moon', 2357), ('infant', 2358), ('investments', 2359), ('negotiate', 2360), ('segment', 2361), ('regulating', 2362), ('fisher', 2363), ('resolution', 2364), ('pledged', 2365), ('violation', 2366), ('proud', 2367), ('1950', 2368), ('anymore', 2369), ('papers', 2370), ('716', 2371), ('aids', 2372), ('dependence', 2373), ('twoandahalf', 2374), ('slower', 2375), ('24000', 2376), ('inmate', 2377), ('statistical', 2378), ('reality', 2379), ('werent', 2380), ('count', 2381), ('120000', 2382), ('granite', 2383), ('considered', 2384), ('rolls', 2385), ('harassment', 2386), ('collection', 2387), ('favor', 2388), ('bigger', 2389), ('represent', 2390), ('false', 2391), ('deprive', 2392), ('hypocrisy', 2393), ('rio', 2394), ('grande', 2395), ('valley', 2396), ('violations', 2397), ('80000', 2398), ('58', 2399), ('settle', 2400), ('cashed', 2401), ('ongoing', 2402), ('prosecuted', 2403), ('violated', 2404), ('shooter', 2405), ('develop', 2406), ('incoming', 2407), ('daylight', 2408), ('risks', 2409), ('crash', 2410), ('estate', 2411), ('represented', 2412), ('rainy', 2413), ('240', 2414), ('eligible', 2415), ('redistricting', 2416), ('continue', 2417), ('perform', 2418), ('contributes', 2419), ('tripled', 2420), ('arm', 2421), ('processed', 2422), ('foods', 2423), ('contain', 2424), ('exploring', 2425), ('alaska', 2426), ('maryland', 2427), ('strictest', 2428), ('connected', 2429), ('exchange', 2430), ('statistics', 2431), ('founded', 2432), ('chafee', 2433), ('wine', 2434), ('underage', 2435), ('simple', 2436), ('greenhouse', 2437), ('productive', 2438), ('allies', 2439), ('advisory', 2440), ('original', 2441), ('confederate', 2442), ('view', 2443), ('simpson', 2444), ('morocco', 2445), ('nafta', 2446), ('trucks', 2447), ('example', 2448), ('threw', 2449), ('hand', 2450), ('capability', 2451), ('workplace', 2452), ('glenn', 2453), ('belonged', 2454), ('cooking', 2455), ('787', 2456), ('attacking', 2457), ('wanting', 2458), ('angel', 2459), ('kasichs', 2460), ('safer', 2461), ('czar', 2462), ('bailouts', 2463), ('aspects', 2464), ('tests', 2465), ('84', 2466), ('cobb', 2467), ('1917', 2468), ('therapy', 2469), ('cuccinelli', 2470), ('restore', 2471), ('sixtytwo', 2472), ('bus', 2473), ('exist', 2474), ('carlos', 2475), ('curbelo', 2476), ('holes', 2477), ('affairs', 2478), ('prohibited', 2479), ('recommending', 2480), ('uk', 2481), ('todays', 2482), ('shrinking', 2483), ('350000', 2484), ('eminent', 2485), ('domain', 2486), ('branded', 2487), ('memorial', 2488), ('hurt', 2489), ('corporation', 2490), ('authorizing', 2491), ('strike', 2492), ('robust', 2493), ('concealed', 2494), ('canadians', 2495), ('whose', 2496), ('customer', 2497), ('ocean', 2498), ('900', 2499), ('goldman', 2500), ('contributor', 2501), ('600000', 2502), ('eligibility', 2503), ('internal', 2504), ('facing', 2505), ('mortgage', 2506), ('short', 2507), ('legislator', 2508), ('room', 2509), ('visits', 2510), ('page', 2511), ('romneycare', 2512), ('option', 2513), ('occurred', 2514), ('4000', 2515), ('asthma', 2516), ('proof', 2517), ('typical', 2518), ('colleagues', 2519), ('pocketed', 2520), ('burn', 2521), ('existing', 2522), ('plants', 2523), ('patriots', 2524), ('panel', 2525), ('gm', 2526), ('buddies', 2527), ('cornyn', 2528), ('bail', 2529), ('approach', 2530), ('lifting', 2531), ('bans', 2532), ('requirements', 2533), ('politifact', 2534), ('outrageous', 2535), ('filibuster', 2536), ('guaranteeing', 2537), ('brad', 2538), ('schimel', 2539), ('forcing', 2540), ('childrenpoverty', 2541), ('initiative', 2542), ('patients', 2543), ('demanded', 2544), ('deep', 2545), ('cures', 2546), ('armies', 2547), ('surrounding', 2548), ('totaling', 2549), ('dark', 2550), ('ii', 2551), ('identifiable', 2552), ('authorize', 2553), ('delays', 2554), ('processing', 2555), ('paytoplay', 2556), ('alligator', 2557), ('germany', 2558), ('kendrick', 2559), ('meek', 2560), ('pennsylvania', 2561), ('del', 2562), ('counting', 2563), ('machines', 2564), ('chances', 2565), ('apparently', 2566), ('recruiting', 2567), ('mistake', 2568), ('imposed', 2569), ('fee', 2570), ('twenty', 2571), ('proudly', 2572), ('features', 2573), ('anderson', 2574), ('criminals', 2575), ('offer', 2576), ('repeated', 2577), ('introduced', 2578), ('opposing', 2579), ('mccrory', 2580), ('fallen', 2581), ('potential', 2582), ('adopting', 2583), ('diabetes', 2584), ('endorse', 2585), ('ix', 2586), ('nh', 2587), ('cited', 2588), ('carolinas', 2589), ('featured', 2590), ('chesterfield', 2591), ('finland', 2592), ('recount', 2593), ('nevada', 2594), ('44', 2595), ('indication', 2596), ('condemn', 2597), ('wrongly', 2598), ('infected', 2599), ('1994', 2600), ('mosque', 2601), ('confirms', 2602), ('68', 2603), ('adam', 2604), ('hasner', 2605), ('f', 2606), ('reuses', 2607), ('bureaucracy', 2608), ('parole', 2609), ('links', 2610), ('lake', 2611), ('erie', 2612), ('contains', 2613), ('streetcar', 2614), ('930', 2615), ('forty', 2616), ('1928', 2617), ('supporters', 2618), ('elderly', 2619), ('skip', 2620), ('las', 2621), ('vegas', 2622), ('meetings', 2623), ('eleven', 2624), ('cameras', 2625), ('46', 2626), ('inspections', 2627), ('eggs', 2628), ('ok', 2629), ('fraudulent', 2630), ('falk', 2631), ('fortune', 2632), ('bet', 2633), ('classified', 2634), ('server', 2635), ('drones', 2636), ('mission', 2637), ('hpv', 2638), ('wasnt', 2639), ('restrictions', 2640), ('exploited', 2641), ('possibly', 2642), ('cast', 2643), ('confirmed', 2644), ('vice', 2645), ('dade', 2646), ('ought', 2647), ('460', 2648), ('1986', 2649), ('embryonic', 2650), ('dead', 2651), ('successful', 2652), ('highway', 2653), ('auditor', 2654), ('louie', 2655), ('shooting', 2656), ('movie', 2657), ('bidder', 2658), ('pentagons', 2659), ('vetoed', 2660), ('declined', 2661), ('breast', 2662), ('portion', 2663), ('secondlargest', 2664), ('maximum', 2665), ('fails', 2666), ('waters', 2667), ('holder', 2668), ('footage', 2669), ('scouts', 2670), ('promotes', 2671), ('margaret', 2672), ('founder', 2673), ('betty', 2674), ('naral', 2675), ('proabortion', 2676), ('divorce', 2677), ('robberies', 2678), ('earmark', 2679), ('affected', 2680), ('paris', 2681), ('acted', 2682), ('vaccinations', 2683), ('surgeries', 2684), ('healthcaremedicare', 2685), ('grade', 2686), ('figures', 2687), ('jr', 2688), ('pull', 2689), ('representative', 2690), ('comments', 2691), ('hearing', 2692), ('halftrue', 2693), ('newsmaker', 2694), ('pantsfire', 2695), ('if', 2696), ('froze', 2697), ('assembly', 2698), ('try', 2699), ('ukraine', 2700), ('domes', 2701), ('talking', 2702), ('12yearold', 2703), ('laughing', 2704), ('seventy', 2705), ('66', 2706), ('97', 2707), ('repaid', 2708), ('profit', 2709), ('toothpaste', 2710), ('nascar', 2711), ('dropout', 2712), ('lgbt', 2713), ('often', 2714), ('boxer', 2715), ('hunt', 2716), ('obese', 2717), ('entirely', 2718), ('delegation', 2719), ('gambling', 2720), ('1990s', 2721), ('cory', 2722), ('1989', 2723), ('145', 2724), ('referral', 2725), ('notorious', 2726), ('injury', 2727), ('temporary', 2728), ('petraeus', 2729), ('corrections', 2730), ('deputies', 2731), ('patrol', 2732), ('electricity', 2733), ('turbines', 2734), ('suicide', 2735), ('hey', 2736), ('thomas', 2737), ('patriot', 2738), ('indianas', 2739), ('cartels', 2740), ('juarez', 2741), ('concerns', 2742), ('rapid', 2743), ('subsequent', 2744), ('misdemeanor', 2745), ('enacting', 2746), ('documents', 2747), ('treating', 2748), ('motorists', 2749), ('buses', 2750), ('produced', 2751), ('worse', 2752), ('story', 2753), ('academies', 2754), ('reach', 2755), ('goals', 2756), ('married', 2757), ('rarer', 2758), ('runs', 2759), ('wild', 2760), ('panther', 2761), ('preventable', 2762), ('walked', 2763), ('polar', 2764), ('vortex', 2765), ('ribboncutting', 2766), ('healthcaregov', 2767), ('seeking', 2768), ('beckhams', 2769), ('amounted', 2770), ('fifteen', 2771), ('assets', 2772), ('happening', 2773), ('cruz', 2774), ('pregnancy', 2775), ('ponzi', 2776), ('packs', 2777), ('braves', 2778), ('beneficiaries', 2779), ('exclude', 2780), ('penalty', 2781), ('always', 2782), ('brotherhood', 2783), ('stated', 2784), ('ship', 2785), ('cops', 2786), ('shift', 2787), ('neighborhoods', 2788), ('harvest', 2789), ('indicating', 2790), ('opponents', 2791), ('akin', 2792), ('peanut', 2793), ('brewpubs', 2794), ('borrowed', 2795), ('cover', 2796), ('homeowners', 2797), ('39', 2798), ('brennan', 2799), ('limitation', 2800), ('drone', 2801), ('responded', 2802), ('derivatives', 2803), ('knew', 2804), ('sexting', 2805), ('bain', 2806), ('learning', 2807), ('stateowned', 2808), ('donors', 2809), ('handgun', 2810), ('11th', 2811), ('nowhere', 2812), ('volcano', 2813), ('collecting', 2814), ('promise', 2815), ('loopholes', 2816), ('sending', 2817), ('urban', 2818), ('gardening', 2819), ('uniforms', 2820), ('overdose', 2821), ('beef', 2822), ('collapsed', 2823), ('catastrophic', 2824), ('speech', 2825), ('barnes', 2826), ('catholic', 2827), ('jewish', 2828), ('protestant', 2829), ('pornography', 2830), ('western', 2831), ('sue', 2832), ('hotel', 2833), ('burden', 2834), ('needed', 2835), ('landrieu', 2836), ('persons', 2837), ('disabilities', 2838), ('worried', 2839), ('weather', 2840), ('skipped', 2841), ('chosen', 2842), ('prime', 2843), ('minister', 2844), ('benjamin', 2845), ('netanyahu', 2846), ('losses', 2847), ('unique', 2848), ('meals', 2849), ('1913', 2850), ('purchasing', 2851), ('taliban', 2852), ('osama', 2853), ('bin', 2854), ('laden', 2855), ('advertising', 2856), ('capitals', 2857), ('boston', 2858), ('guest', 2859), ('ayotte', 2860), ('understand', 2861), ('defect', 2862), ('110', 2863), ('humans', 2864), ('arab', 2865), ('warners', 2866), ('chamber', 2867), ('aflcio', 2868), ('ibm', 2869), ('technology', 2870), ('smart', 2871), ('product', 2872), ('raid', 2873), ('methods', 2874), ('surveys', 2875), ('whove', 2876), ('135', 2877), ('veto', 2878), ('immediately', 2879), ('stolen', 2880), ('film', 2881), ('television', 2882), ('8000', 2883), ('ratio', 2884), ('sometime', 2885), ('festival', 2886), ('target', 2887), ('northeast', 2888), ('october', 2889), ('outbreak', 2890), ('row', 2891), ('reward', 2892), ('jerseys', 2893), ('partisan', 2894), ('contrary', 2895), ('atms', 2896), ('experiencing', 2897), ('toilet', 2898), ('accountability', 2899), ('regulation', 2900), ('fill', 2901), ('authored', 2902), ('designated', 2903), ('slashing', 2904), ('predecessors', 2905), ('acting', 2906), ('nelson', 2907), ('post', 2908), ('promote', 2909), ('hovde', 2910), ('92', 2911), ('parental', 2912), ('length', 2913), ('uncompensated', 2914), ('flat', 2915), ('illnesses', 2916), ('residential', 2917), ('facilities', 2918), ('effective', 2919), ('privacy', 2920), ('prosecutor', 2921), ('secede', 2922), ('pricing', 2923), ('boardwalk', 2924), ('secondary', 2925), ('question', 2926), ('others', 2927), ('math', 2928), ('field', 2929), ('doubt', 2930), ('conventional', 2931), ('gi', 2932), ('neighborhood', 2933), ('47th', 2934), ('marquette', 2935), ('wasted', 2936), ('disclose', 2937), ('bullets', 2938), ('stage', 2939), ('agreements', 2940), ('dozen', 2941), ('stays', 2942), ('hivaids', 2943), ('driver', 2944), ('export', 2945), ('2004', 2946), ('kerry', 2947), ('graham', 2948), ('extending', 2949), ('tenure', 2950), ('arkansas', 2951), ('antibusiness', 2952), ('important', 2953), ('edward', 2954), ('snowden', 2955), ('bulbs', 2956), ('compact', 2957), ('fluorescent', 2958), ('pop', 2959), ('lilly', 2960), ('ledbetter', 2961), ('secure', 2962), ('mansion', 2963), ('milwaukees', 2964), ('barretts', 2965), ('ants', 2966), ('league', 2967), ('zoning', 2968), ('discriminating', 2969), ('extraordinary', 2970), ('overturning', 2971), ('boost', 2972), ('elena', 2973), ('published', 2974), ('articles', 2975), ('administrative', 2976), ('eliminating', 2977), ('connection', 2978), ('productions', 2979), ('1200', 2980), ('blocking', 2981), ('moment', 2982), ('panhandling', 2983), ('mayors', 2984), ('lease', 2985), ('overwhelming', 2986), ('path', 2987), ('degree', 2988), ('citations', 2989), ('tennessee', 2990), ('play', 2991), ('resign', 2992), ('managed', 2993), ('quote', 2994), ('waste', 2995), ('cotton', 2996), ('cranston', 2997), ('value', 2998), ('span', 2999), ('minerals', 3000), ('hardearned', 3001), ('ultimately', 3002), ('consecutive', 3003), ('reagans', 3004), ('awarded', 3005), ('elementary', 3006), ('singlepayer', 3007), ('barbecue', 3008), ('fewest', 3009), ('u', 3010), ('subpoena', 3011), ('sequestration', 3012), ('wrong', 3013), ('stopandfrisk', 3014), ('tripoli', 3015), ('foster', 3016), ('agrees', 3017), ('universal', 3018), ('codes', 3019), ('absolutely', 3020), ('murdered', 3021), ('olympic', 3022), ('haslam', 3023), ('harvard', 3024), ('guarantee', 3025), ('bonuses', 3026), ('potentially', 3027), ('churches', 3028), ('consistent', 3029), ('positive', 3030), ('worldclass', 3031), ('28000', 3032), ('leads', 3033), ('barrels', 3034), ('homicide', 3035), ('physician', 3036), ('era', 3037), ('accept', 3038), ('admissions', 3039), ('exams', 3040), ('burke', 3041), ('partition', 3042), ('announcement', 3043), ('playing', 3044), ('1978', 3045), ('join', 3046), ('critical', 3047), ('connie', 3048), ('mack', 3049), ('retirees', 3050), ('recruiters', 3051), ('electronic', 3052), ('cigarettes', 3053), ('fannie', 3054), ('freddie', 3055), ('de', 3056), ('effectively', 3057), ('walk', 3058), ('specific', 3059), ('exploding', 3060), ('pryor', 3061), ('protein', 3062), ('february', 3063), ('homicides', 3064), ('trumps', 3065), ('whats', 3066), ('liberia', 3067), ('behavior', 3068), ('odd', 3069), ('pick', 3070), ('discretion', 3071), ('anybodys', 3072), ('targeting', 3073), ('21st', 3074), ('kindergarten', 3075), ('vladimir', 3076), ('putin', 3077), ('fellow', 3078), ('surface', 3079), ('alaskas', 3080), ('nominees', 3081), ('ratios', 3082), ('southern', 3083), ('unfair', 3084), ('socially', 3085), ('cigarette', 3086), ('aimed', 3087), ('kitzhaber', 3088), ('organic', 3089), ('usda', 3090), ('crop', 3091), ('environment', 3092), ('embargo', 3093), ('smog', 3094), ('referendum', 3095), ('tent', 3096), ('investigated', 3097), ('jimmy', 3098), ('fundraiser', 3099), ('loses', 3100), ('husbands', 3101), ('highly', 3102), ('gallup', 3103), ('700', 3104), ('30s', 3105), ('creators', 3106), ('army', 3107), ('environmentally', 3108), ('output', 3109), ('toomey', 3110), ('doc', 3111), ('located', 3112), ('51', 3113), ('clunkers', 3114), ('nationwide', 3115), ('roadside', 3116), ('bruce', 3117), ('matter', 3118), ('improved', 3119), ('beach', 3120), ('sentences', 3121), ('sullivan', 3122), ('worlds', 3123), ('dorm', 3124), ('125', 3125), ('contained', 3126), ('penny', 3127), ('sworn', 3128), ('earths', 3129), ('boss', 3130), ('casino', 3131), ('exempt', 3132), ('teams', 3133), ('industries', 3134), ('captured', 3135), ('previously', 3136), ('retired', 3137), ('martial', 3138), ('faced', 3139), ('jones', 3140), ('womans', 3141), ('wife', 3142), ('was', 3143), ('neutrality', 3144), ('earners', 3145), ('youd', 3146), ('lanes', 3147), ('conflict', 3148), ('zack', 3149), ('prostitutes', 3150), ('redefine', 3151), ('meltdown', 3152), ('event', 3153), ('ross', 3154), ('performing', 3155), ('twentyfive', 3156), ('tallahassee', 3157), ('onto', 3158), ('corn', 3159), ('rebecca', 3160), ('bradley', 3161), ('direction', 3162), ('justified', 3163), ('capitol', 3164), ('letting', 3165), ('atheists', 3166), ('fuel', 3167), ('although', 3168), ('employs', 3169), ('rescue', 3170), ('sponsor', 3171), ('implemented', 3172), ('temperature', 3173), ('sense', 3174), ('nathan', 3175), ('ends', 3176), ('resulting', 3177), ('farmland', 3178), ('independence', 3179), ('vegan', 3180), ('californias', 3181), ('pockets', 3182), ('beaches', 3183), ('clinic', 3184), ('helping', 3185), ('mobile', 3186), ('rent', 3187), ('taj', 3188), ('mahal', 3189), ('courthouse', 3190), ('teaches', 3191), ('4th', 3192), ('perdue', 3193), ('democracy', 3194), ('ethical', 3195), ('opinions', 3196), ('course', 3197), ('1970s', 3198), ('beginning', 3199), ('weight', 3200), ('1979', 3201), ('middleincome', 3202), ('written', 3203), ('anybody', 3204), ('restaurant', 3205), ('consumers', 3206), ('wade', 3207), ('contribute', 3208), ('creates', 3209), ('conflicts', 3210), ('defends', 3211), ('sarah', 3212), ('librarian', 3213), ('doggett', 3214), ('prosperity', 3215), ('turnout', 3216), ('ham', 3217), ('bread', 3218), ('judge', 3219), ('tested', 3220), ('jobstransportation', 3221), ('governorelect', 3222), ('congresseconomy', 3223), ('citygovernmenteconomytransportation', 3224), ('trolley', 3225), ('abortionhealthcare', 3226), ('aspirin', 3227), ('bankruptcycorporationseconomyfederalbudgetfinancialregulationtaxes', 3228), ('sets', 3229), ('resolving', 3230), ('purpose', 3231), ('sees', 3232), ('fit', 3233), ('crimecriminaljusticeelectionsmessagemachine2012', 3234), ('santorum', 3235), ('crimecriminaljustice', 3236), ('explosion', 3237), ('educationjobs', 3238), ('category', 3239), ('fouryear', 3240), ('congressionalrulesfederalbudgethealthcare', 3241), ('foodsafetymarketregulation', 3242), ('seattle', 3243), ('carts', 3244), ('alcoholtransportation', 3245), ('abc', 3246), ('privatization', 3247), ('overpass', 3248), ('tysons', 3249), ('healthcarejobsworkers', 3250), ('economyfederalbudgettaxes', 3251), ('federalbudgetincometaxes', 3252), ('gamed', 3253), ('povertyveterans', 3254), ('correctionsandupdatesfederalbudgetimmigration', 3255), ('request', 3256), ('secures', 3257), ('medicaidmessagemachine2012', 3258), ('economyfamilieshungerwelfare', 3259), ('candidatesbiographyimmigrationpoverty', 3260), ('colonia', 3261), ('healthcaretaxesworkers', 3262), ('substantial', 3263), ('crimecriminaljusticehomelandsecuritymilitarypublicsafetyterrorism', 3264), ('incentivized', 3265), ('militarization', 3266), ('precincts', 3267), ('campaignfinancegunssupremecourt', 3268), ('wipe', 3269), ('flies', 3270), ('10th', 3271), ('educationhealthcarestatebudget', 3272), ('electionsstates', 3273), ('kean', 3274), ('economyhungerjobspoverty', 3275), ('energyfederalbudgetoilspillpunditstaxes', 3276), ('debatesgunsvotingrecord', 3277), ('toy', 3278), ('sellers', 3279), ('corporationsenergyenvironmentmessagemachine2012stimulus', 3280), ('lights', 3281), ('federalbudgetpensions', 3282), ('postal', 3283), ('candidatesbiographycrimecriminaljusticehistory', 3284), ('energyfederalbudgetgovernmentefficiencynuclearoilspillscience', 3285), ('presidentially', 3286), ('immigrationurban', 3287), ('rebuild', 3288), ('inner', 3289), ('debtstatebudgetstatefinancestaxes', 3290), ('trenton', 3291), ('154', 3292), ('parking', 3293), ('wins', 3294), ('gyms', 3295), ('taxed', 3296), ('architect', 3297), ('corzines', 3298), ('backwards', 3299), ('foreignpolicyterrorism', 3300), ('hezbollah', 3301), ('venezuela', 3302), ('imminent', 3303), ('citygovernmentcorporationshistorytechnology', 3304), ('tweet', 3305), ('childrendiversityfamiliesgaysandlesbiansmarriage', 3306), ('assertions', 3307), ('heterosexual', 3308), ('shattered', 3309), ('agricultureeconomyhistory', 3310), ('bushadministrationcrimeguns', 3311), ('deficitfederalbudgetjobstaxes', 3312), ('debteconomystatefinances', 3313), ('federalbudgetmedicaidmedicaresocialsecurity', 3314), ('eclipse', 3315), ('foreignpolicygovernmentefficiencyhistoryterrorism', 3316), ('ambassador', 3317), ('christopher', 3318), ('governmentregulationpublicsafetymarketregulation', 3319), ('belts', 3320), ('foreignpolicyisraelwater', 3321), ('gasprices', 3322), ('chu', 3323), ('964', 3324), ('reelected', 3325), ('climatechangepollsscience', 3326), ('57', 3327), ('co2', 3328), ('governmentefficiencyhealthcarepublichealthveterans', 3329), ('hopes', 3330), ('occur', 3331), ('immigrationmilitaryterrorism', 3332), ('confirm', 3333), ('housingimmigration', 3334), ('occupants', 3335), ('childreneducationgaysandlesbianssexuality', 3336), ('allied', 3337), ('pushing', 3338), ('stafford', 3339), ('congressdrugslegalissues', 3340), ('narcotic', 3341), ('alcoholcrimepublicsafetystatestransportation', 3342), ('roads', 3343), ('evey', 3344), ('animalsenvironmentfoodsafetygovernmentregulationpublichealth', 3345), ('eat', 3346), ('freshwater', 3347), ('campaignfinanceenvironment', 3348), ('leased', 3349), ('hide', 3350), ('economyjobslaborstatebudget', 3351), ('membership', 3352), ('optional', 3353), ('climatechangeeconomyenergyenvironmentjobspublichealthmarketregulationwater', 3354), ('shale', 3355), ('floridahealthcaremarketregulation', 3356), ('environmentgovernmentefficiencywater', 3357), ('salmon', 3358), ('handles', 3359), ('saltwater', 3360), ('historyjobaccomplishments', 3361), ('abortionwomen', 3362), ('electionstransparency', 3363), ('herserverof', 3364), ('economyinfrastructuretradetransportation', 3365), ('campaignfinancecongresselectionscampaignadvertising', 3366), ('club', 3367), ('debtfederalbudgethealthcaremedicaidmedicaresocialsecurity', 3368), ('permanently', 3369), ('appropriated', 3370), ('drugspoverty', 3371), ('gunshistory', 3372), ('ownership', 3373), ('automatic', 3374), ('weaspons', 3375), ('homelandsecurity', 3376), ('kelli', 3377), ('ward', 3378), ('agricultureanimalsconsumersafetydrugsenvironmentfoodsafetypublichealthscience', 3379), ('antibiotics', 3380), ('economypundits', 3381), ('legalissueswomenworkers', 3382), ('protecting', 3383), ('chinaforeignpolicynewhampshire2012', 3384), ('soon', 3385), ('englishspeaking', 3386), ('bipartisanshipfederalbudget', 3387), ('agricultureimmigrationjobslabor', 3388), ('strawberry', 3389), ('rough', 3390), ('veteransvotingrecord', 3391), ('criminaljusticeeducation', 3392), ('healthcarejobaccomplishmentsjobs', 3393), ('historyhumanrightslegalissuesreligiontaxes', 3394), ('lyndon', 3395), ('taxexempt', 3396), ('censuseconomy', 3397), ('eighth', 3398), ('populous', 3399), ('educationjobaccomplishmentsjobsstatebudget', 3400), ('afghanistanfoodsafetyiraq', 3401), ('wars', 3402), ('crimeimmigrationpublicsafety', 3403), ('named', 3404), ('safest', 3405), ('capandtradeclimatechangecorporationsenergyenvironmentforeignpolicymessagemachineoilspill', 3406), ('handout', 3407), ('educationimmigrationmessagemachine2012', 3408), ('attend', 3409), ('electionspundits10newstampabay', 3410), ('impossible', 3411), ('deadline', 3412), ('educationsportsstatebudget', 3413), ('floridaamendmentshealthcaremarijuana', 3414), ('refills', 3415), ('chinaforeignpolicynuclearterrorism', 3416), ('energyoilspill', 3417), ('agricultureanimalspublichealth', 3418), ('giant', 3419), ('snail', 3420), ('meningitis', 3421), ('healthcaremessagemachine2012', 3422), ('citygovernmentgovernmentregulationpublichealth', 3423), ('gulp', 3424), ('afghanistaneconomyiraqjobsmessagemachine2012', 3425), ('candidatesbiographysmallbusiness', 3426), ('furniture', 3427), ('frames', 3428), ('popculture', 3429), ('popularity', 3430), ('debut', 3431), ('cardson', 3432), ('netflix', 3433), ('economyeducationgovernmentregulationjobssmallbusinessstatestaxes', 3434), ('butt', 3435), ('educated', 3436), ('rankings', 3437), ('crimecriminaljusticedrugslegalissues', 3438), ('competition', 3439), ('streets', 3440), ('childreneducationgovernmentregulationstatebudgetstates', 3441), ('electionsfinancialregulationcampaignadvertising', 3442), ('financiers', 3443), ('educationguns', 3444), ('opt', 3445), ('animalscountygovernmentlegalissuesstatebudgettransportation', 3446), ('zoo', 3447), ('corporationslaborunionsworkers', 3448), ('mahlon', 3449), ('encouraging', 3450), ('boycott', 3451), ('incomejobsstatebudgetstatefinancestaxes', 3452), ('governmentefficiencyjobsmessagemachinestatebudget', 3453), ('busted', 3454), ('campaignfinancecorrectionsandupdatesweather', 3455), ('soar', 3456), ('cassidy', 3457), ('campaignfinanceeconomyjobaccomplishments', 3458), ('dewine', 3459), ('madoff', 3460), ('predatory', 3461), ('lenders', 3462), ('crimemessagemachine', 3463), ('syndicate', 3464), ('extorting', 3465), ('ganleys', 3466), ('fbis', 3467), ('award', 3468), ('civilrightselectionsstatestransportation', 3469), ('sauk', 3470), ('wednesday', 3471), ('economyhealthcarehousingpublichealth', 3472), ('300k', 3473), ('suicides', 3474), ('childrenhealthcarepundits', 3475), ('repeat', 3476), ('statefinances', 3477), ('offseason', 3478), ('drawing', 3479), ('wouldbe', 3480), ('educationlaborstatebudgetstatefinances', 3481), ('uri', 3482), ('immigrationlegalissuesmarijuana', 3483), ('trespassing', 3484), ('mules', 3485), ('countybudgetcountygovernmentdebttransportation', 3486), ('renegotiated', 3487), ('contribution', 3488), ('portlandmilwaukie', 3489), ('federalbudgetmedicaresocialsecurity', 3490), ('economyincomeworkers', 3491), ('citygovernmentcrimecriminaljustice', 3492), ('afghanistanmilitaryreligion', 3493), ('acknowledged', 3494), ('seizing', 3495), ('burning', 3496), ('bibles', 3497), ('educationstates', 3498), ('9th', 3499), ('64', 3500), ('ged', 3501), ('candidatesbiographyelectionsfinancialregulationmessagemachine', 3502), ('childrenhungerpoverty', 3503), ('groupfeeding', 3504), ('claimsthat', 3505), ('starvation', 3506), ('corporationseconomy', 3507), ('clearly', 3508), ('gaming', 3509), ('citybudgeteconomystatebudgettaxes', 3510), ('beverage', 3511), ('fourthhighest', 3512), ('educationtechnology', 3513), ('lack', 3514), ('electrical', 3515), ('outlets', 3516), ('abortionsupremecourtwomen', 3517), ('crimemessagemachinereligion', 3518), ('scientology', 3519), ('massages', 3520), ('federalbudgetpovertysocialsecuritytaxeswealth', 3521), ('solvent', 3522), ('2033', 3523), ('extend', 3524), ('candidatesbiographycorporationsgaysandlesbiansimmigration', 3525), ('scotts', 3526), ('partners', 3527), ('playboy', 3528), ('networking', 3529), ('geared', 3530), ('foreignpolicyreligionabcnewsweek', 3531), ('church', 3532), ('synagogues', 3533), ('forbidden', 3534), ('correctionsandupdateslegalissues', 3535), ('pled', 3536), ('healthcarehistorylegalissuessocialsecurity', 3537), ('bushadministrationgunscampaignadvertisingsupremecourt', 3538), ('selfdefense', 3539), ('incomejobswomen', 3540), ('areaswho', 3541), ('outearning', 3542), ('spacetourism', 3543), ('shuttle', 3544), ('dayton', 3545), ('censusdiversitypopulation', 3546), ('pensionspublicserviceretirementworkers', 3547), ('hybrid', 3548), ('popculturepundits', 3549), ('followers', 3550), ('830000', 3551), ('jobsmessagemachine2014', 3552), ('healthcareunions', 3553), ('enforcing', 3554), ('punditstaxes', 3555), ('cay', 3556), ('johnston', 3557), ('richest', 3558), ('retirementsocialsecurity', 3559), ('alternative', 3560), ('operates', 3561), ('participants', 3562), ('laborpublicserviceretirementstatefinances', 3563), ('statistically', 3564), ('lawenforcement', 3565), ('israelterrorism', 3566), ('castigated', 3567), ('settlements', 3568), ('rockets', 3569), ('rained', 3570), ('congressdrugseconomyjobs', 3571), ('headquarter', 3572), ('congresscongressionalrules', 3573), ('educationhealthcarepublicsafetystatebudgettaxes', 3574), ('environmentmessagemachine', 3575), ('draining', 3576), ('everglades', 3577), ('economyfederalbudgetjobstaxes', 3578), ('correctionsandupdatesenvironmentsports', 3579), ('provided', 3580), ('habitat', 3581), ('poachers', 3582), ('economyjobsveterans', 3583), ('alcoholpublichealth', 3584), ('guardians', 3585), ('regardless', 3586), ('terrorismtransportation', 3587), ('attention', 3588), ('pale', 3589), ('significance', 3590), ('correctionsandupdatestrade', 3591), ('economyfederalbudgethealthcare', 3592), ('bled', 3593), ('childrenmarriage', 3594), ('climatechangetransportation', 3595), ('environmentrecreation', 3596), ('doyle', 3597), ('dnr', 3598), ('mismanaged', 3599), ('herd', 3600), ('dwindled', 3601), ('campaignfinancevotingrecord', 3602), ('hard', 3603), ('debtdeficitincomestatebudgetstatefinancestaxes', 3604), ('homelandsecuritymarketregulation', 3605), ('federalbudgethealthcaremedicaidstatebudget', 3606), ('crimecriminaljusticehistorypublicsafetypublicservice', 3607), ('constables', 3608), ('educationstatebudgetstates', 3609), ('reciprocity', 3610), ('homelandsecurityterrorismabcnewsweek', 3611), ('initial', 3612), ('isolated', 3613), ('civilrightselectionslegalissues', 3614), ('712', 3615), ('participated', 3616), ('congressdeficiteconomyfederalbudgetgovernmentefficiency', 3617), ('congressforeignpolicyjobaccomplishmentspatriotism', 3618), ('chuck', 3619), ('hagels', 3620), ('character', 3621), ('patriotism', 3622), ('bravery', 3623), ('defending', 3624), ('environmentgovernmentregulationpublichealth', 3625), ('prematurely', 3626), ('candidatesbiographyeducation', 3627), ('finish', 3628), ('dropping', 3629), ('foreignpolicyimmigrationreligion', 3630), ('refugee', 3631), ('candidatesbiographycrime', 3632), ('kilmartin', 3633), ('witness', 3634), ('governmentregulationhealthcarejobaccomplishmentsmedicarepublichealth', 3635), ('governmentcontrolled', 3636), ('beyond', 3637), ('governmentefficiencyjobaccomplishmentsworkers', 3638), ('crimestatebudget', 3639), ('224', 3640), ('countygovernmentelectionsinfrastructure', 3641), ('sellwood', 3642), ('languished', 3643), ('secured', 3644), ('corporationsdeficitfederalbudgettaxes', 3645), ('financialregulationforeignpolicytrade', 3646), ('override', 3647), ('campaignfinancecongressionalrules', 3648), ('content', 3649), ('speakers', 3650), ('animalscorrectionsandupdateshealthcarestimulus', 3651), ('spay', 3652), ('neuter', 3653), ('antiobesity', 3654), ('correctionsandupdatesfederalbudgethealthcaremedicare', 3655), ('julia', 3656), ('6350', 3657), ('incomelaborwomenworkers', 3658), ('childrenforeignpolicyhealthcarepublichealth', 3659), ('jobaccomplishmentsstates', 3660), ('distance', 3661), ('interviewers', 3662), ('campaignfinancedrugselections', 3663), ('twentyseven', 3664), ('pharmaceutical', 3665), ('federalbudgethealthcaremedicaidmedicarepovertypublichealth', 3666), ('energyenvironmentgaspricespundits', 3667), ('soaring', 3668), ('capandtradeclimatechangeenergyenvironmentscience', 3669), ('94', 3670), ('nature', 3671), ('electionsjobaccomplishmentspopculture', 3672), ('jan', 3673), ('debra', 3674), ('medina', 3675), ('remainder', 3676), ('jobspovertyworkers', 3677), ('725', 3678), ('correctionsandupdatesgunsterrorism', 3679), ('crimecriminaljusticedrugsgovernmentregulationpublichealth', 3680), ('debtdeficiteconomyfederalbudgetmessagemachine2012', 3681), ('civilrightsconsumersafetycorporationsgovernmentregulationhealthcarelegalissuespublichealth', 3682), ('device', 3683), ('immunity', 3684), ('injuries', 3685), ('childrenfamilieshealthcareimmigrationterrorism', 3686), ('lbj', 3687), ('candidatesbiographyforeignpolicymessagemachine2012', 3688), ('plouffe', 3689), ('piles', 3690), ('joint', 3691), ('congresseconomyelectionsjobaccomplishmentsjobs', 3692), ('federalbudgethungerwelfare', 3693), ('pledge', 3694), ('bipartisanshipcongressionalrules', 3695), ('filibustered', 3696), ('debateseconomyincomejobslaborworkers', 3697), ('thirdlowest', 3698), ('corporationsfederalbudgetjobs', 3699), ('lowwage', 3700), ('corporationssupremecourt', 3701), ('reversed', 3702), ('floodgates', 3703), ('climatechangeenvironmentpublichealthwater', 3704), ('dioxide', 3705), ('emitted', 3706), ('chemicals', 3707), ('energyfinancialregulationgasprices', 3708), ('speculators', 3709), ('childrengaysandlesbiansmarriage', 3710), ('irrefutable', 3711), ('abortioncorrectionsandupdates', 3712), ('federalbudgethealthcarepublichealth', 3713), ('employersponsored', 3714), ('candidatesbiographyjobs', 3715), ('lemieux', 3716), ('candidatesbiographyhomelandsecurityimmigrationlegalissuestransportation', 3717), ('ones', 3718), ('groping', 3719), ('corporationsfederalbudgettaxestechnologytransportation', 3720), ('bailed', 3721), ('controls', 3722), ('educationimmigrationstates', 3723), ('energymessagemachine2012', 3724), ('childrenmedicaid', 3725), ('births', 3726), ('childrenfamilieslaborunions', 3727), ('economypublicserviceretirementstatebudgetworkers', 3728), ('bipartisanshiphealthcaremedicaid', 3729), ('hampshires', 3730), ('enrolled', 3731), ('legalissuesmilitary', 3732), ('commanders', 3733), ('orders', 3734), ('consumersafetycrimecriminaljusticedrugseconomygovernmentregulationhistorymarijuanapublichealthpublicsafetyrecreationmarketregulationsmallbusinessstatefinancestaxes', 3735), ('primarily', 3736), ('agriculturedeficit', 3737), ('aprovision', 3738), ('corporationsenergyinfrastructure', 3739), ('utility', 3740), ('firstenergys', 3741), ('kyrgyzstan', 3742), ('iceland', 3743), ('citybudgetcitygovernmenthousingtaxes', 3744), ('candidatesbiographyobamabirthcertificate', 3745), ('gunspublichealth', 3746), ('scare', 3747), ('midterm', 3748), ('candidatesbiographyfederalbudgetmedicare', 3749), ('ran', 3750), ('environmenthousingstatesweather', 3751), ('reimbursements', 3752), ('corporationsfederalbudgetretirement', 3753), ('insures', 3754), ('debatesforeignpolicyterrorism', 3755), ('bankruptcydebtfederalbudgetfinancialregulationretirement', 3756), ('healthcarepublichealthscience', 3757), ('cured', 3758), ('sheen', 3759), ('comoros', 3760), ('goats', 3761), ('arthritis', 3762), ('economyjobspoverty', 3763), ('economyenergyenvironmentfloridatransportation', 3764), ('criminaljusticeelections', 3765), ('restoring', 3766), ('historystatefinancesstatestaxes', 3767), ('highesttaxed', 3768), ('debtdeficitfederalbudgethealthcaremedicaid', 3769), ('drastically', 3770), ('healthcaremarijuana', 3771), ('teenager', 3772), ('recommendation', 3773), ('economystatebudgetstatefinancesstates', 3774), ('downgrades', 3775), ('diversitylegalissues', 3776), ('hawaiian', 3777), ('medicaidtaxes', 3778), ('gillespies', 3779), ('enforced', 3780), ('religionterrorism', 3781), ('dearborn', 3782), ('advancement', 3783), ('allah', 3784), ('debteducationmessagemachine2014', 3785), ('correctionsandupdateseconomyjobs', 3786), ('incomewealth', 3787), ('blackwhite', 3788), ('apartheid', 3789), ('baseballeconomyflorida', 3790), ('rays', 3791), ('judged', 3792), ('professional', 3793), ('economypovertywomen', 3794), ('taxeswater', 3795), ('abortioncandidatesbiographyhealthcarelegalissueswomen', 3796), ('anthony', 3797), ('gemma', 3798), ('campaignfinancehistory', 3799), ('outspent', 3800), ('campaignfinancemessagemachine2012', 3801), ('menendez', 3802), ('candidatesbiographyeconomyincomejobaccomplishmentstaxestransparencywealth', 3803), ('disclosure', 3804), ('humanrightsislamnuclear', 3805), ('persecutes', 3806), ('christians', 3807), ('annies', 3808), ('candidatesbiographyelectionsnewhampshire2012', 3809), ('publicsafetytransportationunions', 3810), ('trimet', 3811), ('103', 3812), ('respond', 3813), ('nontransit', 3814), ('agricultureeconomy', 3815), ('farming', 3816), ('corporationseconomysmallbusiness', 3817), ('childrenimmigrationpopulation', 3818), ('bipartisanshipjobs', 3819), ('jobaccomplishmentsmessagemachine2012', 3820), ('coowner', 3821), ('celilo', 3822), ('chinaenergyforeignpolicy', 3823), ('supplies', 3824), ('corporationseconomyincomejobsworkers', 3825), ('healthcarelegalissuestaxes', 3826), ('imprisonment', 3827), ('fines', 3828), ('hefty', 3829), ('economytransportation', 3830), ('accessible', 3831), ('militarynuclear', 3832), ('arsenal', 3833), ('equipment', 3834), ('environmentjobsmarketregulation', 3835), ('bipartisanshipcampaignfinancecandidatesbiographydebates', 3836), ('disgraced', 3837), ('gordon', 3838), ('drugshealthcaremarijuana', 3839), ('parkinsons', 3840), ('glaucoma', 3841), ('associate', 3842), ('economyincomejobslaborworkers', 3843), ('everywhere', 3844), ('paychecks', 3845), ('foreignpolicymilitarynuclearterrorism', 3846), ('rouhani', 3847), ('medicaremessagemachineretirement', 3848), ('healthcarepublichealthstates', 3849), ('thirteen', 3850), ('applicants', 3851), ('governmentefficiencyjobaccomplishmentsmessagemachine2012campaignadvertising', 3852), ('traveling', 3853), ('sexualitytechnology', 3854), ('smartphones', 3855), ('candidatesbiographychildrenethicsfamilieslegalissues', 3856), ('exboyfriend', 3857), ('lawyers', 3858), ('regular', 3859), ('economypopulationpoverty', 3860), ('crimeeducation', 3861), ('investigating', 3862), ('tech', 3863), ('blacksburg', 3864), ('pollstaxes', 3865), ('energygovernmentregulationnewhampshire2012', 3866), ('efficiency', 3867), ('mandates', 3868), ('complied', 3869), ('pensionsstatebudgettaxesunions', 3870), ('federalbudgethealthcaremedicarepundits', 3871), ('15000', 3872), ('healthcaresportswomen', 3873), ('concussions', 3874), ('cheerleading', 3875), ('statebudgetstatefinancestaxeswelfare', 3876), ('thom', 3877), ('tillis', 3878), ('capandtradeclimatechangehealthcare', 3879), ('socialized', 3880), ('citybudgetcitygovernmentcorrectionsandupdatesworkers', 3881), ('sixfigure', 3882), ('pollssports', 3883), ('childrenfamiliesgaysandlesbians', 3884), ('adoption', 3885), ('corporationshealthcare', 3886), ('civilrightsdiversityflorida', 3887), ('yoho', 3888), ('threefifths', 3889), ('crimelaborlegalissuessupremecourtunions', 3890), ('prohibitions', 3891), ('outlawing', 3892), ('federalbudgetforeignpolicy', 3893), ('economyhealthcarejobaccomplishments', 3894), ('hikes', 3895), ('correctionsandupdateseducationfederalbudgethealthcare', 3896), ('candidatesbiographyethicsmessagemachinemilitaryveterans', 3897), ('marine', 3898), ('vietnam', 3899), ('deficitfederalbudgetpundits', 3900), ('corporationsjobs', 3901), ('sheets', 3902), ('educationunions', 3903), ('administrator', 3904), ('federalbudgetnewhampshire2012', 3905), ('countybudgetcountygovernmenteducationtaxes', 3906), ('standpoint', 3907), ('disabilitypensionsretirementstatefinancesstates', 3908), ('economyjobsmessagemachine2012taxes', 3909), ('outsource', 3910), ('candidatesbiographyhealthcaremedicare', 3911), ('prohibiting', 3912), ('economystates', 3913), ('crimecriminaljusticesexualitywomen', 3914), ('foreignpolicymessagemachinemilitarytechnology', 3915), ('raese', 3916), ('laser', 3917), ('sky', 3918), ('debatesfederalbudgetmilitary', 3919), ('criminaljusticegaysandlesbiansstatebudgetwomen', 3920), ('convene', 3921), ('electionsimmigrationmessagemachine2012', 3922), ('kurt', 3923), ('browning', 3924), ('childrenfederalbudgethealthcarepublichealthscienceveteranswomen', 3925), ('ann', 3926), ('kuster', 3927), ('blind', 3928), ('eye', 3929), ('fda', 3930), ('institutes', 3931), ('deficiteconomyfederalbudgetjobs', 3932), ('borrow', 3933), ('entrepreneurs', 3934), ('agriculturegovernmentregulation', 3935), ('chores', 3936), ('correctionsandupdatescrimetaxes', 3937), ('corporationseconomywealth', 3938), ('60s', 3939), ('congresscrimepensionsretirement', 3940), ('ethicsfederalbudgetvotingrecordworkers', 3941), ('straightforward', 3942), ('governmentefficiencystatebudgetstatefinancestransportation', 3943), ('brandnew', 3944), ('messagemachine2012stimulusvotingrecord', 3945), ('foreignpolicynuclear', 3946), ('congressdiversityelectionsstateswomen', 3947), ('childreneducationhealthcarejobsurban', 3948), ('truancy', 3949), ('governmentefficiency', 3950), ('treasurers', 3951), ('candidatesbiographyjobstrade', 3952), ('economyhistoryjobstaxes', 3953), ('existence', 3954), ('governmentefficiencylaborstatebudget', 3955), ('zippo', 3956), ('alcoholmarketregulationsmallbusiness', 3957), ('craft', 3958), ('floridahealthcarehousing', 3959), ('preventing', 3960), ('candidatesbiographyfederalbudgethealthcarepublichealth', 3961), ('animalschildrencrimepublicsafety', 3962), ('serial', 3963), ('killers', 3964), ('abusing', 3965), ('economyforeignpolicyiraqmilitary', 3966), ('britain', 3967), ('abortionfamiliesfederalbudgetstatebudget', 3968), ('fy', 3969), ('327653', 3970), ('528', 3971), ('hyde', 3972), ('debatesethics', 3973), ('scandals', 3974), ('corporationsincomeoccupywallstreettaxes', 3975), ('bushadministrationcongressforeignpolicyiraqmilitaryvotingrecord', 3976), ('afghanistanforeignpolicymilitary', 3977), ('succeeded', 3978), ('federalbudgethistorymedicareretirement', 3979), ('robbed', 3980), ('candidatesbiographyeconomypensionsretirementstatefinanceswealthworkers', 3981), ('gina', 3982), ('raimondo', 3983), ('portfolio', 3984), ('bankruptcyeconomyfederalbudgetfinancialregulationjobaccomplishmentsmarketregulation', 3985), ('stabilizing', 3986), ('1980s', 3987), ('foreignpolicypublichealth', 3988), ('instability', 3989), ('recipient', 3990), ('economyfederalbudgetincomepopulationwealth', 3991), ('energyethicsjobaccomplishmentsmessagemachine2012', 3992), ('economyjobsstimulus', 3993), ('abortioncandidatesbiography', 3994), ('blake', 3995), ('rocap', 3996), ('debateshealthcare', 3997), ('congresstaxestechnologytransparency', 3998), ('erased', 3999), ('crimecriminaljusticeeconomy', 4000), ('recidivism', 4001), ('foreignpolicypopculture', 4002), ('waved', 4003), ('flags', 4004), ('crimeelectionsimmigrationstates', 4005), ('noncitizen', 4006), ('victory', 4007), ('childreneconomyeducationfamilieshealthcarehungerincomejobslabormarriagepovertywomenworkers', 4008), ('moms', 4009), ('undereducated', 4010), ('starving', 4011), ('candidatesbiographyelectionshistorypolls', 4012), ('afghanistaniraqmilitaryabcnewsweek', 4013), ('invade', 4014), ('oilspillstatebudget', 4015), ('crists', 4016), ('photoop', 4017), ('deficitfederalbudgetweather', 4018), ('offsets', 4019), ('congressgovernmentefficiencygovernmentregulationsmallbusiness', 4020), ('candidatesbiographyfinancialregulationtaxes', 4021), ('fiduciary', 4022), ('electionsmessagemachine2012', 4023), ('noncitizens', 4024), ('governmentregulationtaxes', 4025), ('principally', 4026), ('plot', 4027), ('lerner', 4028), ('appointee', 4029), ('civilrightscongresshomelandsecurityprivacyterrorism', 4030), ('nsa', 4031), ('environmentmarketregulationwater', 4032), ('abide', 4033), ('caution', 4034), ('disturb', 4035), ('bodies', 4036), ('puddle', 4037), ('governmentefficiencynewhampshire2012', 4038), ('onefourth', 4039), ('corporationsmedicaidwelfareworkers', 4040), ('messagemachine2012', 4041), ('rigells', 4042), ('civilrightsgaysandlesbianslegalissuesmarriage', 4043), ('lesbian', 4044), ('june', 4045), ('candidatesbiographylegalissues', 4046), ('thesis', 4047), ('criticized', 4048), ('plutocratic', 4049), ('thugs', 4050), ('shackles', 4051), ('crimedrugselections', 4052), ('abortionhealthcaresocialsecurity', 4053), ('laborlegalissuesstatebudget', 4054), ('embroiled', 4055), ('fleeing', 4056), ('budgetrepair', 4057), ('messagemachineretirementstatebudgetworkers', 4058), ('jerry', 4059), ('powers', 4060), ('economywomen', 4061), ('federalbudgetmedicaremessagemachine', 4062), ('523', 4063), ('candidatesbiographyhousing', 4064), ('involve', 4065), ('exterior', 4066), ('economygovernmentefficiencygovernmentregulationtaxes', 4067), ('healthcarehistorypunditssocialsecurity', 4068), ('correctionsandupdateseducationmarketregulation', 4069), ('removes', 4070), ('educationvotingrecord', 4071), ('sununu', 4072), ('countybudgetcountygovernmentstates', 4073), ('candidatesbiographywealth', 4074), ('demise', 4075), ('healthcaremedicaremessagemachine2012', 4076), ('gill', 4077), ('economyhealthcaremedicaidmedicare', 4078), ('educationgunspublicsafetystates', 4079), ('statesupported', 4080), ('duties', 4081), ('jobsmessagemachine', 4082), ('environmentoilspill', 4083), ('supply', 4084), ('childreneconomyfamilies', 4085), ('downturn', 4086), ('bipartisanshipimmigration', 4087), ('invitation', 4088), ('candidatesbiographyobamabirthcertificatereligion', 4089), ('citygovernmentcountygovernmentcriminaljusticejobaccomplishmentslegalissuespublicservice', 4090), ('corrupted', 4091), ('legalissuesreligion', 4092), ('accommodation', 4093), ('grow', 4094), ('beard', 4095), ('reasons', 4096), ('afghanistanhumanrightsiraqterrorism', 4097), ('cia', 4098), ('psychologists', 4099), ('torture', 4100), ('publicservice', 4101), ('freshman', 4102), ('ordinary', 4103), ('elective', 4104), ('sleep', 4105), ('might', 4106), ('pose', 4107), ('debteconomyhistoryhousing', 4108), ('excited', 4109), ('devastated', 4110), ('scoop', 4111), ('cheap', 4112), ('citygovernmentdiversitywomen', 4113), ('racially', 4114), ('ethnically', 4115), ('genderwise', 4116), ('diversityeconomyworkers', 4117), ('economyjobsstatebudgetstatefinances', 4118), ('89', 4119), ('censuscitygovernmentcountygovernmentpopulation', 4120), ('oak', 4121), ('grove', 4122), ('capandtradeclimatechangeeconomyenergy', 4123), ('18th', 4124), ('historymessagemachine2012publicsafetystatebudgetstatefinancesstates', 4125), ('congresselectionsredistricting', 4126), ('relatives', 4127), ('gaysandlesbianslegalissuesmarriagereligionsupremecourt', 4128), ('scholars', 4129), ('marriages', 4130), ('childrencrimeeducation', 4131), ('12000', 4132), ('agricultureeconomystatebudget', 4133), ('71', 4134), ('candidatesbiographynuclearvotingrecord', 4135), ('debteducationfederalbudgettaxes', 4136), ('deductible', 4137), ('countybudgetcountygovernmenteconomyjobaccomplishmentsjobs', 4138), ('falks', 4139), ('homelandsecurityisraelnuclearterrorism', 4140), ('homelandsecuritytechnology', 4141), ('practices', 4142), ('economypovertystates', 4143), ('outstripped', 4144), ('1959', 4145), ('agricultureanimalsconsumersafetyeconomyenvironmentgovernmentregulationpublichealthsciencetechnology', 4146), ('organisms', 4147), ('gmos', 4148), ('environmentgaspricesmessagemachine2012', 4149), ('animalsscience', 4150), ('crabs', 4151), ('crimecriminaljusticedrugsgovernmentregulationgunspublicsafety', 4152), ('skyrocketed', 4153), ('criminaljusticeguns', 4154), ('crimejobaccomplishmentscampaignadvertising', 4155), ('convictions', 4156), ('allegation', 4157), ('electionsjobaccomplishmentslegalissuessupremecourt', 4158), ('gunsmarketregulation', 4159), ('camper', 4160), ('federalbudgethealthcarestatebudget', 4161), ('implement', 4162), ('educationstatebudgetstatefinancesstates', 4163), ('perpupil', 4164), ('electionsethics', 4165), ('crimecriminaljusticetransportation', 4166), ('marta', 4167), ('civilrightsgunsreligion', 4168), ('freed', 4169), ('ku', 4170), ('klux', 4171), ('klan', 4172), ('jobsstates', 4173), ('volvo', 4174), ('spurned', 4175), ('3455', 4176), ('promising', 4177), ('peach', 4178), ('deficitfederalbudgetjobs', 4179), ('federalbudgetforeignpolicyhealthcare', 4180), ('worldleading', 4181), ('commitment', 4182), ('federalbudgetgovernmentefficiencyhealthcaretaxes', 4183), ('16500', 4184), ('policing', 4185), ('bipartisanshipcandidatesbiographyvotingrecord', 4186), ('76', 4187), ('economyretirement', 4188), ('britains', 4189), ('401ks', 4190), ('candidatesbiographystatebudgetstatefinances', 4191), ('upgraded', 4192), ('alcoholgovernmentregulation', 4193), ('direct', 4194), ('shipment', 4195), ('mouse', 4196), ('click', 4197), ('energyenvironmentmarketregulation', 4198), ('halt', 4199), ('gases', 4200), ('bipartisanshipcongresshistory', 4201), ('foreignpolicyisraelterrorism', 4202), ('undermine', 4203), ('saudis', 4204), ('citybudgetcitygovernmentlaborpensionspublicsafetytaxesunions', 4205), ('historysupremecourt', 4206), ('overrule', 4207), ('branchesof', 4208), ('educationstatebudgettaxes', 4209), ('1700', 4210), ('infinite', 4211), ('promises', 4212), ('healthcaremedicareretirement', 4213), ('ration', 4214), ('wasteful', 4215), ('historystateswomen', 4216), ('comprise', 4217), ('lawmaking', 4218), ('144', 4219), ('181', 4220), ('diversityhistorystates', 4221), ('compromise', 4222), ('statehouse', 4223), ('appearance', 4224), ('sovereignty', 4225), ('deficitfederalbudgethealthcaremedicare', 4226), ('economyhistoryjobaccomplishments', 4227), ('immigrationtransportationvotingrecord', 4228), ('educationwomen', 4229), ('rural', 4230), ('illiterate', 4231), ('correctionsandupdateshomelandsecurityimmigrationtransportation', 4232), ('inspected', 4233), ('candidatesbiographyfederalbudgethealthcaremessagemachine2012taxes', 4234), ('legalissuespunditssotomayornominationsupremecourt', 4235), ('justicedesignate', 4236), ('sneaky', 4237), ('unsigned', 4238), ('chinaforeignpolicymilitarynuclear', 4239), ('jobslegalissueswomen', 4240), ('toughen', 4241), ('grothman', 4242), ('cleaning', 4243), ('economygamblingjobsstatebudgetstatefinancestaxes', 4244), ('quonset', 4245), ('coffers', 4246), ('produces', 4247), ('stimulustaxes', 4248), ('educationlaborworkers', 4249), ('44th', 4250), ('2nd', 4251), ('citybudgetjobaccomplishments', 4252), ('downgraded', 4253), ('bbb', 4254), ('junkbond', 4255), ('countygovernmentretirementsocialsecurityworkers', 4256), ('economymessagemachinetrade', 4257), ('49000', 4258), ('91000', 4259), ('candidatesbiographywomen', 4260), ('staffers', 4261), ('environmentnaturaldisastersweather', 4262), ('forest', 4263), ('fires', 4264), ('forests', 4265), ('environmentalists', 4266), ('popculturesports', 4267), ('outdrawn', 4268), ('arenas', 4269), ('followed', 4270), ('surpassed', 4271), ('debatesjobaccomplishmentspublicsafetyterrorism', 4272), ('financialregulationgovernmentregulationtaxes', 4273), ('complying', 4274), ('economyfinancialregulationpatriotismmarketregulation', 4275), ('fireworks', 4276), ('jobsmessagemachine2012', 4277), ('celebrity', 4278), ('grads', 4279), ('federalbudgetforeignpolicyhistorymessagemachinetrade', 4280), ('portmans', 4281), ('exploded', 4282), ('oversaw', 4283), ('spree', 4284), ('congresstransportation', 4285), ('energyenvironmentforeignpolicy', 4286), ('drilled', 4287), ('congressforeignpolicymilitary', 4288), ('1983', 4289), ('beirut', 4290), ('barracks', 4291), ('bombing', 4292), ('key', 4293), ('educationfamilies', 4294), ('spring', 4295), ('civilrightscrimecriminaljustice', 4296), ('conviction', 4297), ('citygovernmentgovernmentefficiency', 4298), ('relation', 4299), ('historywomen', 4300), ('viciously', 4301), ('abused', 4302), ('countybudgetdebttaxes', 4303), ('federalbudgethistorymilitary', 4304), ('superiority', 4305), ('older', 4306), ('civilrightsgaysandlesbians', 4307), ('diverting', 4308), ('conversion', 4309), ('crimegunsterrorism', 4310), ('procedure', 4311), ('notified', 4312), ('unionsworkers', 4313), ('burkes', 4314), ('madison', 4315), ('ignore', 4316), ('201516', 4317), ('environmentoilspillstates', 4318), ('inaction', 4319), ('marsh', 4320), ('healthcaremedicaremessagemachine', 4321), ('sestak', 4322), ('gut', 4323), ('854489', 4324), ('jeopardizing', 4325), ('childreneconomyfamiliesfederalbudgetgovernmentefficiencyhealthcarehousinghumanrightshungerincomemedicaidpovertypublichealthstatebudgetstatefinancestaxesurbanveteranswelfarewomen', 4326), ('averaged', 4327), ('jobsunions', 4328), ('numerous', 4329), ('generate', 4330), ('familiesmarriage', 4331), ('divorces', 4332), ('environmentfederalbudgetoilspill', 4333), ('dedicate', 4334), ('steer', 4335), ('jobaccomplishmentsstatebudgetstatefinancestaxes', 4336), ('165', 4337), ('healthcaresmallbusiness', 4338), ('educationstatefinancestaxestransportation', 4339), ('rides', 4340), ('chinamilitary', 4341), ('submarines', 4342), ('foreignpolicyiraqterrorism', 4343), ('climatechangeenvironment', 4344), ('educationstimulus', 4345), ('plug', 4346), ('correctionsandupdatesmedicare', 4347), ('sebelius', 4348), ('inform', 4349), ('policyholders', 4350), ('economyhealthcaremessagemachine2014', 4351), ('economyfederalbudgetnewhampshire2012marketregulation', 4352), ('1960s', 4353), ('consuming', 4354), ('trendline', 4355), ('cease', 4356), ('publicserviceretirementstatefinancesworkers', 4357), ('childrenfamiliesincomepovertywelfare', 4358), ('works', 4359), ('crimecriminaljusticeeducationgunspublicsafety', 4360), ('fatality', 4361), ('civilrightslaborstatebudget', 4362), ('segregated', 4363), ('educationhealthcarelegalissueswomen', 4364), ('dental', 4365), ('alcoholcitygovernment', 4366), ('malfeasance', 4367), ('economyfederalbudgethistoryjobstaxes', 4368), ('rebounding', 4369), ('economyhistoryabcnewsweek', 4370), ('bipartisanshippopculturepundits', 4371), ('trusted', 4372), ('healthcaremessagemachine', 4373), ('animalsgovernmentregulationguns', 4374), ('shoot', 4375), ('bears', 4376), ('floor', 4377), ('window', 4378), ('candidatesbiographyterrorism', 4379), ('ignored', 4380), ('consulate', 4381), ('healthcarestatebudgetabcnewsweek', 4382), ('historylegalissuesmessagemachinesupremecourt', 4383), ('ethicspundits', 4384), ('tucson', 4385), ('thrive', 4386), ('logo', 4387), ('slogan', 4388), ('medicaremessagemachine', 4389), ('reopen', 4390), ('darn', 4391), ('doughnut', 4392), ('hole', 4393), ('infrastructurestatebudgettransportation', 4394), ('dovilla', 4395), ('\x90', 4396), ('congressdiversityreligion', 4397), ('278', 4398), ('cantors', 4399), ('candidatesbiographymessagemachine2012newhampshire2012taxestransparency', 4400), ('traditionally', 4401), ('candidatesbiographycongressforeignpolicyhistoryterrorism', 4402), ('citybudgetcitygovernmentethics', 4403), ('randi', 4404), ('shades', 4405), ('contributors', 4406), ('formula', 4407), ('racetrack', 4408), ('economyjobaccomplishmentsjobssmallbusiness', 4409), ('46th', 4410), ('capandtradeclimatechangeenergymessagemachine2012', 4411), ('corporationsfinancialregulationmarketregulationabcnewsweektransportation', 4412), ('motors', 4413), ('climatechangeenvironmentweather', 4414), ('destructive', 4415), ('storm', 4416), ('foreignpolicymilitarynuclear', 4417), ('susan', 4418), ('rice', 4419), ('conceded', 4420), ('uranium', 4421), ('enrichment', 4422), ('debteconomyfamiliesfinancialregulationincomepovertywealthworkers', 4423), ('relying', 4424), ('highcost', 4425), ('payday', 4426), ('pawn', 4427), ('shop', 4428), ('cashing', 4429), ('statebudgetstimulus', 4430), ('happily', 4431), ('blaming', 4432), ('gunsprivacy', 4433), ('applications', 4434), ('dishonorably', 4435), ('discharged', 4436), ('substances', 4437), ('mind', 4438), ('stalking', 4439), ('crimegunsstateswomen', 4440), ('privatesale', 4441), ('handguns', 4442), ('housingtourismtrade', 4443), ('candidatesbiographycrimecriminaljusticelegalissues', 4444), ('vacuum', 4445), ('cleaners', 4446), ('salesman', 4447), ('animalsclimatechangeeconomyenergyenvironmenthistorypopulationpublichealthscienceweather', 4448), ('acidic', 4449), ('floridahealthcare', 4450), ('floridahealthcaremedicaid', 4451), ('healthcaremessagemachine2014women', 4452), ('cheaper', 4453), ('udalls', 4454), ('censusredistricting', 4455), ('nonanglo', 4456), ('chinaforeignpolicymilitaryspace', 4457), ('practicing', 4458), ('blow', 4459), ('satellites', 4460), ('electionsmarketregulation', 4461), ('no', 4462), ('healthcaremedicaidstatebudget', 4463), ('257000', 4464), ('jobstaxesvotingrecord', 4465), ('jobkilling', 4466), ('paperwork', 4467), ('economyoilspilltourism', 4468), ('panhandle', 4469), ('highestever', 4470), ('bed', 4471), ('collections', 4472), ('abortionpolls', 4473), ('bit', 4474), ('lender', 4475), ('governmentefficiencyjobaccomplishmentsmessagemachine2012', 4476), ('treasurer', 4477), ('economyhealthcarehousingincomeretirementtaxes', 4478), ('congressgovernmentregulationhealthcare', 4479), ('goldplated', 4480), ('historypopulation', 4481), ('millennials', 4482), ('electorate', 4483), ('debatesenergy', 4484), ('pointed', 4485), ('marijuanapublichealthtransportation', 4486), ('legalization', 4487), ('recreational', 4488), ('agriculturecampaignfinanceethicsmessagemachine', 4489), ('dorman', 4490), ('grace', 4491), ('brags', 4492), ('federalbudgetmedicaremessagemachine2012socialsecurity', 4493), ('civilrightsgunsterrorism', 4494), ('boarding', 4495), ('airplane', 4496), ('buying', 4497), ('dynamite', 4498), ('ak47', 4499), ('abortionhealthcareimmigration', 4500), ('covers', 4501), ('foreignpolicyhumanrightslegalissues', 4502), ('economyincomemessagemachine2012', 4503), ('energyenvironmentgovernmentregulationhealthcarepublichealthmarketregulation', 4504), ('electionsimmigrationpoverty', 4505), ('backdoor', 4506), ('economynewhampshire2012workers', 4507), ('1996', 4508), ('crimereligionterrorism', 4509), ('racial', 4510), ('supremacist', 4511), ('messagemachinestatebudget', 4512), ('plale', 4513), ('childreneducationhealthcarehousingpublichealthwelfare', 4514), ('35000', 4515), ('climatechangeenergyenvironmentgovernmentregulation', 4516), ('limitedly', 4517), ('historysports', 4518), ('explosive', 4519), ('scored', 4520), ('correctionsandupdateshealthcarelegalissues', 4521), ('faceless', 4522), ('lifesustaining', 4523), ('conscious', 4524), ('deficitstatebudgetstatefinances', 4525), ('jobstaxes', 4526), ('inherited', 4527), ('bankruptcymarketregulation', 4528), ('correctionsandupdatesdebtfinancialregulationhealthcaretaxes', 4529), ('correctionsandupdatesfederalbudgethealthcaretaxes', 4530), ('corporationseconomyfinancialregulationpundits', 4531), ('federalbudgetpollstaxes', 4532), ('citybudgetcitygovernmentmessagemachine2012taxes', 4533), ('educationlabor', 4534), ('awardwinning', 4535), ('megan', 4536), ('sampson', 4537), ('hungerveterans', 4538), ('charities', 4539), ('heads', 4540), ('childreneducationgovernmentregulation', 4541), ('38th', 4542), ('proficiency', 4543), ('alcoholchildrendrugsmarijuana', 4544), ('seem', 4545), ('indicate', 4546), ('minors', 4547), ('easy', 4548), ('federalbudgetiraqmedicaremilitarytaxes', 4549), ('citygovernmentworkers', 4550), ('projection', 4551), ('municipal', 4552), ('boundaries', 4553), ('healthcarepublichealthstatebudgetterrorism', 4554), ('bankruptcycandidatesbiography', 4555), ('pantsonfire', 4556), ('correctionsandupdatescrime', 4557), ('stops', 4558), ('whatever', 4559), ('congresscongressionalrulesgovernmentefficiency', 4560), ('placed', 4561), ('economyjobaccomplishmentsmessagemachine', 4562), ('messagemachine2014women', 4563), ('candidateterri', 4564), ('lynn', 4565), ('abortiongovernmentregulationlegalissuesstates', 4566), ('cahoots', 4567), ('civilrightsdiversity', 4568), ('comply', 4569), ('wedding', 4570), ('cakes', 4571), ('congressionalruleslegalissues', 4572), ('confirming', 4573), ('childrenpublichealth', 4574), ('seeing', 4575), ('healthier', 4576), ('deficitfederalbudgetobamabirthcertificate', 4577), ('stories', 4578), ('certificate', 4579), ('drowned', 4580), ('historymilitary', 4581), ('eisenhower', 4582), ('saluted', 4583), ('consumersafetycorrectionsandupdatespublichealth', 4584), ('chai', 4585), ('distributed', 4586), ('watereddown', 4587), ('hivaidsdrugs', 4588), ('subsaharan', 4589), ('candidatesbiography', 4590), ('dunnam', 4591), ('educationfederalbudgethealthcarescienceabcnewsweek', 4592), ('climatechangeforeignpolicy', 4593), ('negotiated', 4594), ('officially', 4595), ('crimecriminaljusticediversity', 4596), ('foreignpolicyisraelmilitary', 4597), ('educationfinancialregulation', 4598), ('forgiving', 4599), ('foreignpolicyiraqmilitaryterrorism', 4600), ('useless', 4601), ('fighters', 4602), ('electionsredistricting', 4603), ('constituent', 4604), ('mine', 4605), ('sciencestates', 4606), ('lawmaker', 4607), ('pushes', 4608), ('glow', 4609), ('humanjellyfish', 4610), ('hybrids', 4611), ('citygovernmentoccupywallstreet', 4612), ('economyfederalbudgetgovernmentefficiencystatefinancesstimulus', 4613), ('dubious', 4614), ('electionspundits', 4615), ('educationprivacy', 4616), ('energyabcnewsweek', 4617), ('barton', 4618), ('gavel', 4619), ('chairmanship', 4620), ('afghanistanhumanrightslegalissuesmilitaryterrorismabcnewsweek', 4621), ('fights', 4622), ('organization', 4623), ('immigrationmilitary', 4624), ('enlist', 4625), ('gunspublicsafety', 4626), ('bloomberg', 4627), ('guardsmen', 4628), ('challenge', 4629), ('suspend', 4630), ('crimegovernmentefficiency', 4631), ('dna', 4632), ('lab', 4633), ('environmentforeignpolicytransportation', 4634), ('750000', 4635), ('economylaborlegalissueswomen', 4636), ('candidatesbiographycriminaljusticeethics', 4637), ('accuses', 4638), ('fitzgerald', 4639), ('correctionsandupdateseconomysmallbusiness', 4640), ('starting', 4641), ('animalscorrectionsandupdatescrimeguns', 4642), ('happens', 4643), ('concealandcarry', 4644), ('healthcarehistorypublichealthsciencewater', 4645), ('fluoridation', 4646), ('nazi', 4647), ('ghettos', 4648), ('pacify', 4649), ('abortionhumanrightsvotingrecord', 4650), ('personhood', 4651), ('conforming', 4652), ('rulings', 4653), ('citybudgetcitygovernmentethicslegalissues', 4654), ('martinez', 4655), ('stuck', 4656), ('2465750', 4657), ('candidatesbiographycrimegunsjobaccomplishmentscampaignadvertisingpublicsafety', 4658), ('candidatesbiographyhealthcarejobaccomplishmentsmessagemachinestatebudgetvotingrecord', 4659), ('liberals', 4660), ('obamastyle', 4661), ('votingrecord', 4662), ('maurice', 4663), ('ferre', 4664), ('986', 4665), ('chinahistoryjobaccomplishmentsnuclear', 4666), ('capandtradeclimatechangeeconomy', 4667), ('spray', 4668), ('bordentown', 4669), ('regional', 4670), ('crimedrugshealthcaremarijuana', 4671), ('cathy', 4672), ('jordan', 4673), ('dragged', 4674), ('swat', 4675), ('hooligans', 4676), ('gaysandlesbiansmarriagevotingrecord', 4677), ('correctionsandupdateseducationstatebudget', 4678), ('ben', 4679), ('chafin', 4680), ('votingto', 4681), ('palace', 4682), ('chinatrade', 4683), ('whereas', 4684), ('homelandsecurityterrorism', 4685), ('substitutes', 4686), ('manmade', 4687), ('incomesmallbusinesstaxes', 4688), ('surtax', 4689), ('businessmen', 4690), ('economygamblingstatebudgetstatefinances', 4691), ('connecticuts', 4692), ('slot', 4693), ('candidatesbiographyelectionsmedicaresocialsecurity', 4694), ('chose', 4695), ('vicepresidential', 4696), ('mate', 4697), ('chinajobsmessagemachine2012workers', 4698), ('chrysler', 4699), ('italians', 4700), ('jeeps', 4701), ('childrenfederalbudget', 4702), ('invest', 4703), ('citybudgetcitygovernmentfederalbudgetrecreation', 4704), ('deficiteconomyfederalbudgettaxes', 4705), ('floridaimmigration', 4706), ('entered', 4707), ('overstayed', 4708), ('visas', 4709), ('childreneconomyfamiliesjobspoverty', 4710), ('1010', 4711), ('laborlegalissuesstatebudgetunionsworkers', 4712), ('overhauling', 4713), ('gunsmilitary', 4714), ('marines', 4715), ('citygovernmentelectionsstates', 4716), ('abortiongovernmentregulationhealthcarepublichealthreligion', 4717), ('childreneducationfederalbudgetjobspovertytaxes', 4718), ('agricultureenvironment', 4719), ('shelter', 4720), ('monsanto', 4721), ('syngenta', 4722), ('biotech', 4723), ('federalbudgetmedicaremessagemachine2012', 4724), ('nelsons', 4725), ('jobsmessagemachine2012stimulus', 4726), ('martin', 4727), ('heinrich', 4728), ('disabilityjobsstatesworkers', 4729), ('ablebodied', 4730), ('maine', 4731), ('bipartisanshipcandidatesbiographyelectionspunditsredistricting', 4732), ('tracked', 4733), ('fled', 4734), ('crimecriminaljusticevotingrecord', 4735), ('laurie', 4736), ('monnes', 4737), ('deficitfederalbudgetnewhampshire2012', 4738), ('candidatesbiographyhealthcaremessagemachine2012', 4739), ('blueprint', 4740), ('economyhistoryjobaccomplishmentsjobslaborstatesworkers', 4741), ('inall', 4742), ('candidatesbiographyeconomypensionsretirementworkers', 4743), ('underperformed', 4744), ('caprio', 4745), ('immigrationtransportationwelfare', 4746), ('obamacars', 4747), ('motorcycles', 4748), ('scooters', 4749), ('bankruptcycorporationseconomytransparency', 4750), ('anonymous', 4751), ('citybudgeteducationgovernmentefficiency', 4752), ('clock', 4753), ('bushadministrationcivilrights', 4754), ('motion', 4755), ('censure', 4756), ('systemic', 4757), ('wiretaps', 4758), ('retirementstatebudgetworkers', 4759), ('educationincomejobs', 4760), ('welders', 4761), ('philosophers', 4762), ('citybudgetlaborunions', 4763), ('countybudgetcountygovernment', 4764), ('childrenfamilieshomelandsecurityimmigration', 4765), ('pathway', 4766), ('deliver', 4767), ('squat', 4768), ('debteconomyeducationfinancialregulation', 4769), ('energynuclearpublicsafetypundits', 4770), ('abortionjobswomen', 4771), ('abortionhealthcarestatebudgetstatefinanceswomen', 4772), ('abortioncongressionalruleseconomyfamiliesgaysandlesbiansgovernmentefficiencygunslaboroccupywallstreetreligionworkers', 4773), ('topics', 4774), ('congressincomewomen', 4775), ('civilrightsgaysandlesbiansmarriage', 4776), ('governmentefficiencylaborunions', 4777), ('stateswomen', 4778), ('maggie', 4779), ('candidatesbiographyeducationstatebudgetstates', 4780), ('41st', 4781), ('civilrightseconomy', 4782), ('egging', 4783), ('spitting', 4784), ('policemens', 4785), ('protests', 4786), ('candidatesbiographymedicaresocialsecurity', 4787), ('unwinding', 4788), ('welfaretowork', 4789), ('deficitjobsmessagemachine2012military', 4790), ('candidatesbiographytechnology', 4791), ('statebudgetstatesworkers', 4792), ('correctionsandupdatescrimecriminaljustice', 4793), ('childrenfamiliesfederalbudgetmedicaidmedicaresocialsecurityveterans', 4794), ('childrencorrectionsandupdatesfamiliesgaysandlesbians', 4795), ('jeb', 4796), ('equality', 4797), ('candidatesbiographyhealthcarescience', 4798), ('huckabee', 4799), ('infomercials', 4800), ('drugsmarijuanarecreationscience', 4801), ('potent', 4802), ('yesteryear', 4803), ('educationgaysandlesbiansgovernmentregulationsexuality', 4804), ('hb2threaten', 4805), ('title', 4806), ('looked', 4807), ('corporationsdebteconomyenergyenvironmentfederalbudgetgaspricesgovernmentregulationinfrastructure', 4808), ('rigs', 4809), ('environmentgovernmentregulationnewhampshire2012', 4810), ('landfill', 4811), ('nashua', 4812), ('federalbudgetpublicsafety', 4813), ('obamaphones', 4814), ('candidatesbiographyforeignpolicy', 4815), ('stoned', 4816), ('adulteryand', 4817), ('congressionalruleseducationfederalbudgethealthcare', 4818), ('overcharging', 4819), ('historypopculturepundits', 4820), ('appearances', 4821), ('federalbudgetjobaccomplishments', 4822), ('senelect', 4823), ('johnsons', 4824), ('caucus', 4825), ('corporationseconomygaysandlesbiansstatefinancesstates', 4826), ('needle', 4827), ('iota', 4828), ('candidatesbiographyhistory', 4829), ('aphotograph', 4830), ('21yearold', 4831), ('childrencitybudgetdeficiteducationstatebudgettaxes', 4832), ('punish', 4833), ('charter', 4834), ('countygovernmenttaxes', 4835), ('gecker', 4836), ('vehicle', 4837), ('debteconomyfederalbudgetmessagemachine2012', 4838), ('windmills', 4839), ('economyhungerpoverty', 4840), ('hungry', 4841), ('civilrightsgovernmentregulationguns', 4842), ('correctionsandupdateseconomyhistoryincomejobaccomplishmentstransparencywealth', 4843), ('terrible', 4844), ('russian', 4845), ('oligarchs', 4846), ('borrows', 4847), ('federalbudgetnaturaldisasterswaterweather', 4848), ('smallbusinesswomen', 4849), ('womenowned', 4850), ('governmentregulationhealthcare', 4851), ('expect', 4852), ('civilrightscrimecriminaljusticedrugsmarijuana', 4853), ('countygovernmentelections', 4854), ('bankruptcyeconomyhousingmessagemachine', 4855), ('economystatebudgetstatefinancestaxes', 4856), ('counselors', 4857), ('woods', 4858), ('foreignpolicyhumanrightslabortrade', 4859), ('malaysia', 4860), ('indentured', 4861), ('servants', 4862), ('passports', 4863), ('slavelike', 4864), ('congressionalrulesforeignpolicyhumanrightsislamisraelreligion', 4865), ('sentencing', 4866), ('pastor', 4867), ('capps', 4868), ('alcoholcorporationstaxes', 4869), ('miller', 4870), ('brewing', 4871), ('anheuserbusch', 4872), ('abortioncandidatesbiographycorrectionsandupdates', 4873), ('conveyed', 4874), ('stance', 4875), ('educationgovernmentefficiencylaborstatebudgetunions', 4876), ('wisconsinmilwaukee', 4877), ('secondlowest', 4878), ('socialsecurity', 4879), ('jolly', 4880), ('correctionsandupdatesjobs', 4881), ('candidatesbiographycorporationsjobaccomplishmentsmessagemachineworkers', 4882), ('hp', 4883), ('carly', 4884), ('fiorina', 4885), ('religionabcnewsweek', 4886), ('cultural', 4887), ('citybudget', 4888), ('hesecured', 4889), ('deficiteconomysocialsecurity', 4890), ('gunstaxes', 4891), ('debatesmessagemachine2014stimulustransportation', 4892), ('candidatesbiographychildrencongresscriminaljusticehistoryhumanrightswomen', 4893), ('underground', 4894), ('network', 4895), ('civilrightsdiversityfamiliesgaysandlesbiansmarriage', 4896), ('infrastructurejobstaxestransportation', 4897), ('economyjobsstatebudgetstatefinancesstatestaxes', 4898), ('jennifer', 4899), ('granholm', 4900), ('congressvotingrecord', 4901), ('campaignfinance', 4902), ('standalone', 4903), ('gamblingreligionvotingrecord', 4904), ('coalition', 4905), ('citybudgetcitygovernmentwater', 4906), ('educationgovernmentefficiency', 4907), ('tape', 4908), ('childreneducationmessagemachine2014statebudget', 4909), ('sizes', 4910), ('electionsforeignpolicyhistoryhomelandsecurity', 4911), ('cyberattacks', 4912), ('kremlin', 4913), ('civilrightscrimecriminaljusticemarijuana', 4914), ('probation', 4915), ('enslaved', 4916), ('1850', 4917), ('governmentefficiencyjobaccomplishmentsstatebudget', 4918), ('107', 4919), ('candidatesbiographyelectionsmessagemachine', 4920), ('jailed', 4921), ('abramoff', 4922), ('cw', 4923), ('environmentrecreationwater', 4924), ('abcnewsweek', 4925), ('answer', 4926), ('hypotheticals', 4927), ('afghanistanmilitary', 4928), ('deadliest', 4929), ('crimecriminaljusticejobslaborlegalissues', 4930), ('5600', 4931), ('electionslaborworkers', 4932), ('healthcaremessagemachine2012votingrecord', 4933), ('economyfinancialregulationmessagemachine', 4934), ('funneled', 4935), ('quarters', 4936), ('nobid', 4937), ('immigrationpublicsafetyvotingrecord', 4938), ('citybudgetcrimefederalbudget', 4939), ('historyhomelandsecurityimmigrationlegalissuespundits', 4940), ('childrenforeignpolicyisraelmilitary', 4941), ('haskilled', 4942), ('palestinian', 4943), ('bushadministrationelections', 4944), ('nixon', 4945), ('agricultureveterans', 4946), ('todd', 4947), ('staples', 4948), ('woo', 4949), ('suggesting', 4950), ('incomesportswomen', 4951), ('countygovernmenthistoryrecreation', 4952), ('economyhealthcare', 4953), ('abortioncensuschildrenfamilieshealthcarepublichealthwomen', 4954), ('unintended', 4955), ('gaspricestaxes', 4956), ('economytourism', 4957), ('conventions', 4958), ('laborstatebudgeturban', 4959), ('resemble', 4960), ('crimeimmigrationstatebudgetstatefinances', 4961), ('achieved', 4962), ('21500', 4963), ('tons', 4964), ('narcotics', 4965), ('confiscated', 4966), ('agriculturemarketregulation', 4967), ('retail', 4968), ('sites', 4969), ('proliferation', 4970), ('electionslegalissuesstates', 4971), ('6200', 4972), ('electionday', 4973), ('proved', 4974), ('countybudgetstatebudgettaxes', 4975), ('chinamessagemachine2012', 4976), ('candidatesbiographycorporations', 4977), ('godfathers', 4978), ('pizza', 4979), ('commonsense', 4980), ('principles', 4981), ('abortionsexuality', 4982), ('invades', 4983), ('peoples', 4984), ('choices', 4985), ('historyjobslabor', 4986), ('eighthour', 4987), ('40hour', 4988), ('henry', 4989), ('ford', 4990), ('israelpublicsafety', 4991), ('baltimore', 4992), ('trained', 4993), ('mossad', 4994), ('shin', 4995), ('correctionsandupdatespopulationtransportation', 4996), ('thousand', 4997), ('publichealthstatebudget', 4998), ('50000000', 4999), ('correctionsandupdatesethicsforeignpolicytechnology', 5000), ('material', 5001), ('marked', 5002), ('whilesecretary', 5003), ('censuseconomyfamilies', 5004), ('afghanistanforeignpolicyiraqlegalissuesmilitaryterrorism', 5005), ('battlefield', 5006), ('debteconomyeducation', 5007), ('corporationsdeficiteconomyfinancialregulationgovernmentregulationoccupywallstreettaxes', 5008), ('16000year', 5009), ('ceogoldman', 5010), ('blankfein', 5011), ('16000hour', 5012), ('militarypundits', 5013), ('pete', 5014), ('hoekstra', 5015), ('tweeted', 5016), ('whereabouts', 5017), ('topsecret', 5018), ('climatechangeenergyenvironmentjobs', 5019), ('familieshealthcare', 5020), ('vaccinated', 5021), ('electionsjobslaborunions', 5022), ('exempting', 5023), ('drugshealthcaremedicare', 5024), ('thompsons', 5025), ('prohibits', 5026), ('climatechangeenergyenvironmentstatesweather', 5027), ('windfarm', 5028), ('childreneducationstatebudgetstatefinances', 5029), ('healthcaremedicaidtaxes', 5030), ('insure', 5031), ('economyfinancialregulationhealthcarepundits', 5032), ('bennett', 5033), ('rutah', 5034), ('legalissues', 5035), ('1540', 5036), ('nominations', 5037), ('defeated', 5038), ('foreignpolicylegalissuesmilitaryterrorism', 5039), ('dick', 5040), ('cheney', 5041), ('unable', 5042), ('outstanding', 5043), ('warrants', 5044), ('crimehealthcare', 5045), ('broward', 5046), ('punditsvotingrecord', 5047), ('keith', 5048), ('olbermann', 5049), ('socialsecuritytaxes', 5050), ('congressfederalbudget', 5051), ('murrayryan', 5052), ('dividedgovernment', 5053), ('candidatesbiographyeconomyforeignpolicyhistoryjobaccomplishmentstrade', 5054), ('deleted', 5055), ('agricultureconsumersafetyenvironmentpublichealthwater', 5056), ('faucet', 5057), ('agriculturefederalbudgetpovertywelfare', 5058), ('healthcaremessagemachine2012science', 5059), ('campaignfinancecandidatesbiographyforeignpolicymilitarycampaignadvertising', 5060), ('kings', 5061), ('oman', 5062), ('yemen', 5063), ('infrastructurestatestransportation', 5064), ('6600', 5065), ('20012012', 5066), ('citygovernmentwater', 5067), ('flint', 5068), ('burlington', 5069), ('vt', 5070), ('clean', 5071), ('messagemachine2012taxesvotingrecord', 5072), ('pants', 5073), ('chinaforeignpolicyjobaccomplishmentsjobsmessagemachine2012trade', 5074), ('chinas', 5075), ('citybudgetincomevotingrecord', 5076), ('jessica', 5077), ('economymilitaryveterans', 5078), ('900000', 5079), ('ethicslegalissuesstatebudgettransparency', 5080), ('diverted', 5081), ('misuse', 5082), ('crimepunditsreligion', 5083), ('gohmert', 5084), ('blamed', 5085), ('theater', 5086), ('judeochristian', 5087), ('recreation', 5088), ('healthcaremilitary', 5089), ('economyhousingjobaccomplishmentsstatebudget', 5090), ('40000', 5091), ('markets', 5092), ('energyenvironmentstatefinances', 5093), ('lucky', 5094), ('immigrationwelfare', 5095), ('legalized', 5096), ('climatechangeenergyenvironmentmessagemachine2014', 5097), ('pocketing', 5098), ('welfareworkers', 5099), ('5800', 5100), ('correctionsandupdateseconomy', 5101), ('decline', 5102), ('fakenewspopculture', 5103), ('actor', 5104), ('dwayne', 5105), ('rock', 5106), ('wore', 5107), ('tshirt', 5108), ('stating', 5109), ('kneel', 5110), ('immigrationpopulation', 5111), ('healthcaretaxeswomen', 5112), ('economyrecreationsportsstatebudgettaxes', 5113), ('5050', 5114), ('publicprivate', 5115), ('split', 5116), ('financing', 5117), ('debthealthcaretaxes', 5118), ('fortysix', 5119), ('civilrightseducationhistory', 5120), ('fatherinlaw', 5121), ('linwood', 5122), ('holton', 5123), ('integrated', 5124), ('abortionfederalbudgethealthcare', 5125), ('electionspopulation', 5126), ('citybudgetcitygovernmentcountybudgetcountygovernmentcrimestatebudget', 5127), ('contributing', 5128), ('campaignfinanceethicsgovernmentregulation', 5129), ('childreneducationpoverty', 5130), ('environmentjobsmessagemachine2012recreation', 5131), ('gillnets', 5132), ('inland', 5133), ('taxesterrorism', 5134), ('multiyear', 5135), ('governmentregulationguns', 5136), ('wear', 5137), ('bracelets', 5138), ('afterthefactcivilrightscriminaljusticepublicsafetytransparency', 5139), ('nclaw', 5140), ('camera', 5141), ('broad', 5142), ('filmed', 5143), ('crimecriminaljusticeimmigration', 5144), ('abortionchildrenwomen', 5145), ('extremely', 5146), ('premature', 5147), ('survive', 5148), ('healthy', 5149), ('healthcarehousingstatebudget', 5150), ('abortionchildrensexuality', 5151), ('usas', 5152), ('curriculum', 5153), ('sanger', 5154), ('friedan', 5155), ('icons', 5156), ('emulate', 5157), ('crimeeconomygambling', 5158), ('governmentregulationincomejobsmarketregulationsmallbusinessunionswomenworkers', 5159), ('elses', 5160), ('retaliated', 5161), ('federalbudgetstatebudget', 5162), ('educationgunsmarketregulation', 5163), ('dildo', 5164), ('ebolapublichealth', 5165), ('outbreaks', 5166), ('globe', 5167), ('confines', 5168), ('educationsportstechnology', 5169), ('studentathletes', 5170), ('ipad', 5171), ('immigrationmessagemachinesocialsecurity', 5172), ('congresspublichealthterrorism', 5173), ('pray', 5174), ('debatesfederalbudgetgovernmentefficiency', 5175), ('electionsethicsstates', 5176), ('mris', 5177), ('forth', 5178), ('extensively', 5179), ('mittromney', 5180), ('9874json', 5181), ('barelytrue', 5182), ('edgillespie', 5183), ('strategist', 5184), ('3072json', 5185), ('146', 5186), ('governmentefficiencytransparency', 5187), ('newsmax', 5188), ('magazine', 5189), ('solicitation', 5190), ('2436json', 5191), ('repeating', 5192), ('prek', 5193), ('3rd', 5194), ('alexsink', 5195), ('cites', 5196), ('9721json', 5197), ('advised', 5198), ('barrel', 5199), ('trigger', 5200), ('crimecriminaljusticegunslegalissues', 5201), ('greaterwisconsinpoliticalfund', 5202), ('3627json', 5203), ('ronaldrenuart', 5204), ('11900json', 5205), ('bernies', 5206), ('vermont', 5207), ('pbs', 5208), ('4611json', 5209), ('under', 5210), ('almond', 5211), ('dmv', 5212), ('civilrightshomelandsecurityimmigrationpublicsafetytransportationworkers', 5213), ('davidquiroa', 5214), ('prresident', 5215), ('guatemalanamerican', 5216), ('alliance', 5217), ('3168json', 5218), ('censuscrimeeducationhealthcareimmigrationstatebudgettaxes', 5219), ('terrygorman', 5220), ('radio', 5221), ('6832json', 5222), ('assure', 5223), ('congressionalrulesfederalbudgetmilitary', 5224), ('waynepowell', 5225), ('5893json', 5226), ('jamieradtke', 5227), ('statement', 5228), ('3304json', 5229), ('laborstatebudget', 5230), ('jefffitzgerald', 5231), ('channel', 5232), ('1638json', 5233), ('correctionsandupdateshistorymilitary', 5234), ('crimea', 5235), ('1954', 5236), ('republic', 5237), ('congresscongressionalrulesfederalbudget', 5238), ('803700', 5239), ('abortionmedicare', 5240), ('craig', 5241), ('huey', 5242), ('candidatesbiographyeducationstatebudget', 5243), ('2487', 5244), ('countybudget', 5245), ('millionplus', 5246), ('abortionpundits', 5247), ('tiller', 5248), ('merely', 5249), ('reporting', 5250), ('prolifers', 5251), ('abortioncongresscongressionalrulesfederalbudget', 5252), ('climatechangecongressionalrulesenvironmentfederalbudgetinfrastructure', 5253), ('israelterrorismtransportation', 5254), ('candidatesbiographycorrectionsandupdatescriminaljusticewomen', 5255), ('raping', 5256), ('occasions', 5257), ('abortionchildrengovernmentregulationhealthcarehumanrightslegalissuespollspublichealthreligionwomen', 5258), ('deficiteconomytaxes', 5259), ('pres', 5260), ('transactions', 5261), ('povertywomen', 5262), ('abject', 5263), ('criminaljusticepundits', 5264), ('executions', 5265), ('bipartisanshipcongressvotingrecord', 5266), ('lotterystatebudgetstatefinances', 5267), ('played', 5268), ('returned', 5269), ('watersheds', 5270), ('federalbudgetmessagemachine2012veteransvotingrecord', 5271), ('governmentefficiencyhistoryjobaccomplishmentsvotingrecord', 5272), ('oclock', 5273), ('latenight', 5274), ('corporationsfederalbudgetfinancialregulation', 5275), ('legalissuespopculture', 5276), ('toothbrush', 5277), ('candidatesbiographysports', 5278), ('educationmessagemachine', 5279), ('alabamas', 5280), ('crimegaysandlesbians', 5281), ('recognized', 5282), ('educationgamblingstatefinances', 5283), ('staterun', 5284), ('healthcaremarketregulation', 5285), ('obamashealth', 5286), ('crimecriminaljusticehomelandsecurityimmigration', 5287), ('encountered', 5288), ('identified', 5289), ('customs', 5290), ('detained', 5291), ('deportation', 5292), ('politically', 5293), ('correct', 5294), ('alcoholmarketregulation', 5295), ('consumption', 5296), ('handinhand', 5297), ('healthcaremedicaremessagemachineretirement', 5298), ('altogether', 5299), ('historyreligion', 5300), ('movement', 5301), ('electionsjobaccomplishmentspolls', 5302), ('economygunssports', 5303), ('nineday', 5304), ('healthcarepublichealthstatebudget', 5305), ('psychological', 5306), ('distress', 5307), ('economyhistoryhousingmarketregulation', 5308), ('rooted', 5309), ('gee', 5310), ('hope', 5311), ('collapse', 5312), ('correctionsandupdateshealthcarepublichealthstates', 5313), ('357', 5314), ('congresselectionsstimulus', 5315), ('correctionsandupdateshomelandsecurityimmigration', 5316), ('reaching', 5317), ('deportations', 5318), ('whelan', 5319), ('1920s', 5320), ('abortioncongresssexuality', 5321), ('gardner', 5322), ('championed', 5323), ('eightyear', 5324), ('crusade', 5325), ('educationjobsstatebudget', 5326), ('201315', 5327), ('federalbudgetstimulus', 5328), ('payasyougo', 5329), ('entitlement', 5330), ('etc', 5331), ('it', 5332), ('consumersafetyfinancialregulationgovernmentregulation', 5333), ('cordrays', 5334), ('marks', 5335), ('educationlegalissuesmessagemachinestates', 5336), ('economyjobstaxes', 5337), ('186740', 5338), ('61050', 5339), ('censuspundits', 5340), ('federalbudgetstimulustaxes', 5341), ('campaignfinancecandidatesbiographyethics', 5342), ('julin', 5343), ('castro', 5344), ('sevenfigure', 5345), ('smells', 5346), ('dirty', 5347), ('gift', 5348), ('mikal', 5349), ('watts', 5350), ('donor', 5351), ('statebudgetstatefinances', 5352), ('campaignfinancecandidatesbiography', 5353), ('cigars', 5354), ('economypunditswelfare', 5355), ('immigrationabcnewsweekworkers', 5356), ('gutted', 5357), ('candidatesbiographycriminaljusticeforeignpolicygovernmentregulationhomelandsecuritymilitarytechnologytransparency', 5358), ('capandtradeclimatechangeenvironmentpundits', 5359), ('cooling', 5360), ('economyfederalbudgetmessagemachinetaxes', 5361), ('18000', 5362), ('pelosis', 5363), ('downtown', 5364), ('laborpublicsafety', 5365), ('untrained', 5366), ('lakefront', 5367), ('economyenergyenvironment', 5368), ('woonsocket', 5369), ('garbageburning', 5370), ('nose', 5371), ('fueled', 5372), ('censusfamiliesincomepovertywealth', 5373), ('poorest', 5374), ('subscription', 5375), ('energyenvironment', 5376), ('turbine', 5377), ('flicker', 5378), ('shadows', 5379), ('blades', 5380), ('seizures', 5381), ('educationgunshealthcarepublichealth', 5382), ('attempt', 5383), ('animalspolls', 5384), ('nj', 5385), ('hunting', 5386), ('candidatesbiographydiversitycampaignadvertising', 5387), ('sheila', 5388), ('jackson', 5389), ('wrinkly', 5390), ('whiteaged', 5391), ('hasbeens', 5392), ('spot', 5393), ('planet', 5394), ('historypatriotism', 5395), ('paine', 5396), ('civilrightsgaysandlesbiansmarriagereligion', 5397), ('laborlegalissuesunions', 5398), ('environmentgovernmentregulationjobs', 5399), ('republicansponsored', 5400), ('jobaccomplishmentswelfare', 5401), ('ended', 5402), ('afghanistancrimeimmigration', 5403), ('blocks', 5404), ('economyfederalbudgetjobstransportation', 5405), ('buffett', 5406), ('611000', 5407), ('ebolaforeignpolicypublichealthscience', 5408), ('160000', 5409), ('hazmat', 5410), ('prompting', 5411), ('anticipating', 5412), ('bankruptcyhealthcare', 5413), ('bankruptcies', 5414), ('crimecriminaljusticedrugsgunslegalissues', 5415), ('drugsmarijuana', 5416), ('usage', 5417), ('synthetic', 5418), ('dramatically', 5419), ('instances', 5420), ('bodily', 5421), ('candidatesbiographytaxes', 5422), ('overexaggerated', 5423), ('satisfied', 5424), ('foreignpolicyhistorylabortransportationunions', 5425), ('seriously', 5426), ('controllers', 5427), ('educationpublicsafety', 5428), ('passing', 5429), ('climatechangepublichealth', 5430), ('medicaresocialsecurity', 5431), ('disputed', 5432), ('trimming', 5433), ('chinatechnology', 5434), ('countybudgetpublicsafety', 5435), ('dekalb', 5436), ('graduated', 5437), ('ambitious', 5438), ('censusfamilieshistorymarriagepopulationwomen', 5439), ('bipartisanshipmedicaidstatebudget', 5440), ('debtdeficitfederalbudgetmessagemachine2012', 5441), ('neumann', 5442), ('jobaccomplishmentsstatebudget', 5443), ('reining', 5444), ('crimediversitygambling', 5445), ('reservations', 5446), ('corporationsmessagemachine2012statebudgettaxes', 5447), ('civilrightscrimeelectionspundits', 5448), ('dismissal', 5449), ('intimidation', 5450), ('countybudgetcountygovernmenttaxes', 5451), ('decrease', 5452), ('crimedrugshealthcarepublichealth', 5453), ('opioids', 5454), ('accidental', 5455), ('gunshomelandsecurity', 5456), ('foreignpolicyabcnewsweek', 5457), ('strides', 5458), ('foreignpolicynuclearterrorismtrade', 5459), ('watered', 5460), ('governmentefficiencystatebudgetstates', 5461), ('federalbudgethealthcaremedicaresocialsecurity', 5462), ('brendan', 5463), ('doherty', 5464), ('1960', 5465), ('regard', 5466), ('challenges', 5467), ('physically', 5468), ('occupations', 5469), ('gaysandlesbiansreligionworkers', 5470), ('discriminates', 5471), ('daycare', 5472), ('childrenfamiliesgaysandlesbianshumanrightslegalissuesmarriagesciencesexualitysupremecourt', 5473), ('outlive', 5474), ('fertility', 5475), ('electionshistoryreligion', 5476), ('letters', 5477), ('climatechangeweather', 5478), ('proves', 5479), ('ice', 5480), ('melting', 5481), ('jobaccomplishmentsveterans', 5482), ('terrorismwomen', 5483), ('lures', 5484), ('kittens', 5485), ('nutella', 5486), ('healthcarepunditstechnology', 5487), ('betatested', 5488), ('abortionpunditswomen', 5489), ('item', 5490), ('vaginal', 5491), ('sportsstatebudgettaxes', 5492), ('gunshealthcarepublicsafetypundits', 5493), ('commissioned', 5494), ('inconvenient', 5495), ('facts', 5496), ('harmed', 5497), ('attackers', 5498), ('effectiveness', 5499), ('buybacks', 5500), ('candidatesbiographyeconomy', 5501), ('hewlettpackard', 5502), ('economyfamiliespatriotismscience', 5503), ('pong', 5504), ('invaders', 5505), ('iphone', 5506), ('debtweather', 5507), ('tag', 5508), ('4200', 5509), ('economyfinancialregulationmarketregulationabcnewsweek', 5510), ('totaled', 5511), ('childrencrimetechnology', 5512), ('campaignstyle', 5513), ('frame', 5514), ('educationhealthcarepublichealth', 5515), ('physicians', 5516), ('graduates', 5517), ('forprofit', 5518), ('climatechangeenvironmentpollsscience', 5519), ('criminaljusticejobaccomplishmentsstatebudgetstatefinances', 5520), ('candidatesbiographyforeignpolicyimmigrationterrorism', 5521), ('debatesforeignpolicyimmigrationpublicsafetyterrorism', 5522), ('vetted', 5523), ('countygovernmentpublichealthtaxes', 5524), ('coos', 5525), ('citygovernmentsports', 5526), ('lasted', 5527), ('campaignfinancecrime', 5528), ('teamed', 5529), ('felon', 5530), ('smear', 5531), ('foreignpolicyhumanrights', 5532), ('normalization', 5533), ('liberation', 5534), ('historytaxes', 5535), ('frankly', 5536), ('alcoholcriminaljusticestatebudgettransportation', 5537), ('threshold', 5538), ('drunkendriving', 5539), ('crimeimmigrationvotingrecord', 5540), ('pembroke', 5541), ('pines', 5542), ('detention', 5543), ('ranches', 5544), ('candidatesbiographyelectionsredistricting', 5545), ('correctionsandupdateselectionscampaignadvertising', 5546), ('countybudgetjobaccomplishments', 5547), ('federalbudgethealthcare', 5548), ('burns', 5549), ('candidatesbiographystatebudget', 5550), ('republicancandidate', 5551), ('animalsconsumersafetygovernmentregulationpublicsafety', 5552), ('lap', 5553), ('80pound', 5554), ('mph', 5555), ('2400pound', 5556), ('punch', 5557), ('baseballjobssports', 5558), ('9241', 5559), ('295', 5560), ('healthcaremedicaidmedicaretaxes', 5561), ('gaysandlesbianslegalissuesreligion', 5562), ('liberty', 5563), ('exemption', 5564), ('afghanistanfederalbudgetiraqmilitary', 5565), ('accounted', 5566), ('worldwide', 5567), ('accounts', 5568), ('candidatesbiographydiversityelectionshistory', 5569), ('latina', 5570), ('debtdeficiteconomyeducationgovernmentefficiencyjobsmessagemachine2014campaignadvertisingpensionssmallbusinessstatebudgetstatefinancestaxesunions', 5571), ('seth', 5572), ('magaziner', 5573), ('abortioncrimetaxeswomen', 5574), ('sean', 5575), ('duffy', 5576), ('audits', 5577), ('federalbudgetgovernmentefficiencymilitaryveterans', 5578), ('disrupt', 5579), ('economymessagemachine2012poverty', 5580), ('climatechangeenvironmentscience', 5581), ('tide', 5582), ('gauges', 5583), ('measuring', 5584), ('1930', 5585), ('inches', 5586), ('campaignfinancecrimecriminaljusticeelectionsethicslegalissues', 5587), ('doe', 5588), ('exceptions', 5589), ('messagemachine2012transportation', 5590), ('atlantans', 5591), ('commuting', 5592), ('260', 5593), ('citygovernmenteconomyjobaccomplishmentstransportation', 5594), ('audit', 5595), ('energyenvironmentabcnewsweek', 5596), ('batteries', 5597), ('crimecriminaljusticetaxes', 5598), ('jasper', 5599), ('economygaysandlesbiansmilitary', 5600), ('alcoholgovernmentregulationlegalissues', 5601), ('breweries', 5602), ('pint', 5603), ('premise', 5604), ('sixpack', 5605), ('debateselections', 5606), ('createda', 5607), ('maximize', 5608), ('economylaborunions', 5609), ('foreignpolicyisraelpunditsterrorism', 5610), ('declare', 5611), ('candidatesbiographywomenworkers', 5612), ('218029', 5613), ('153014', 5614), ('20102014', 5615), ('congresseconomyjobstaxes', 5616), ('governmentefficiencyhistorystatebudgetstatefinances', 5617), ('submitted', 5618), ('earliest', 5619), ('homelandsecuritymilitarypatriotismstates', 5620), ('installations', 5621), ('deficitfederalbudgethealthcaretaxes', 5622), ('correctionsandupdateseconomyjobspoverty', 5623), ('exceeds', 5624), ('averageand', 5625), ('countybudgethealthcare', 5626), ('sit', 5627), ('squad', 5628), ('mentally', 5629), ('ill', 5630), ('sometimes', 5631), ('federalbudgetjobs', 5632), ('isaksons', 5633), ('chambliss', 5634), ('citybudgetcitygovernmentpublicsafety', 5635), ('kathie', 5636), ('tovo', 5637), ('invests', 5638), ('paramedics', 5639), ('federalbudgetmilitaryabcnewsweek', 5640), ('federalbudgetveterans', 5641), ('abortiondebates', 5642), ('videos', 5643), ('formed', 5644), ('fetus', 5645), ('beating', 5646), ('legs', 5647), ('alive', 5648), ('correctionsandupdatesforeignpolicyreligion', 5649), ('notices', 5650), ('economymessagemachine2012', 5651), ('corzine', 5652), ('smartest', 5653), ('corporationsjobstaxes', 5654), ('governmentregulationjobaccomplishmentslaborstatebudgetunionsworkers', 5655), ('diversityfinancialregulationhousing', 5656), ('applicant', 5657), ('credentials', 5658), ('foreignpolicygovernmentregulationtechnology', 5659), ('intends', 5660), ('economyenergyinfrastructuretaxestransportationurban', 5661), ('addition', 5662), ('hillsborough', 5663), ('countygovernmentfoodsafetygovernmentregulationhealthcare', 5664), ('wash', 5665), ('butterandjelly', 5666), ('alcoholstatefinances', 5667), ('controlling', 5668), ('spaceweather', 5669), ('warmer', 5670), ('mars', 5671), ('canada', 5672), ('debtdeficitfederalbudgethistorypundits', 5673), ('woodrow', 5674), ('wilson', 5675), ('bushadministrationfederalbudgetpunditstaxes', 5676), ('pensionsstatebudgettaxes', 5677), ('floridamarketregulation', 5678), ('infrastructurestimulustransportation', 5679), ('civilrightsterrorism', 5680), ('geographic', 5681), ('healthcaremedicarestatebudget', 5682), ('corporationseconomyfinancialregulation', 5683), ('electionslegalissuespublicservice', 5684), ('crimelegalissues', 5685), ('jb', 5686), ('hollen', 5687), ('kratz', 5688), ('candidatesbiographyjobaccomplishments', 5689), ('bright', 5690), ('horizons', 5691), ('lady', 5692), ('rightly', 5693), ('citygovernment', 5694), ('amanda', 5695), ('fritz', 5696), ('manages', 5697), ('candidatesbiographyeducationsports', 5698), ('interceptions', 5699), ('tackles', 5700), ('defensive', 5701), ('energystatebudget', 5702), ('luther', 5703), ('olsen', 5704), ('foodsafetygovernmentefficiency', 5705), ('regulates', 5706), ('swiss', 5707), ('civilrightselectionspundits', 5708), ('energyenvironmentgovernmentregulationjobs', 5709), ('obamaclinton', 5710), ('crimegunspublichealth', 5711), ('purchases', 5712), ('economysmallbusiness', 5713), ('entrepreneurship', 5714), ('1st', 5715), ('educationjobaccomplishmentsstatebudget', 5716), ('capped', 5717), ('federalbudgettransportation', 5718), ('candidatesbiographyethicsmessagemachinesmallbusinesstradetransparency', 5719), ('connections', 5720), ('defaulted', 5721), ('chinacrimecriminaljusticeforeignpolicyinfrastructure', 5722), ('dialogues', 5723), ('100plus', 5724), ('addressed', 5725), ('cybersecurity', 5726), ('bankruptcyfinancialregulationprivacy', 5727), ('monitoring', 5728), ('knowledge', 5729), ('storing', 5730), ('homelandsecurityiraqislammilitaryterrorism', 5731), ('westerners', 5732), ('corporationseconomyjobaccomplishmentsmessagemachine2012taxes', 5733), ('abortioncivilrightshistoryhumanrights', 5734), ('abraham', 5735), ('slavery', 5736), ('runaway', 5737), ('climatechangeenvironmentmessagemachine2012', 5738), ('aligned', 5739), ('historyiraqmilitarypolls', 5740), ('candidatesbiographyethicscampaignadvertisingpolls', 5741), ('battleground', 5742), ('untrustworthy', 5743), ('ebolahealthcarepublichealth', 5744), ('jazzercise', 5745), ('massage', 5746), ('redirected', 5747), ('crimecriminaljusticemessagemachine2014women', 5748), ('stronger', 5749), ('actthan', 5750), ('chinaeconomyforeignpolicyhomelandsecuritymilitarysmallbusinesstrade', 5751), ('manufactured', 5752), ('healthcaremessagemachine2012taxes', 5753), ('imposes', 5754), ('countybudgetcountygovernmentpublicsafetystatebudgettaxes', 5755), ('curry', 5756), ('drugsgunshealthcarepublichealth', 5757), ('120', 5758), ('outnumber', 5759), ('agriculturechildreneducation', 5760), ('cafeterias', 5761), ('laborpunditsstatebudgetabcnewsweek', 5762), ('perquisites', 5763), ('housingmessagemachine2012', 5764), ('values', 5765), ('energyenvironmentgovernmentregulation', 5766), ('earthquake', 5767), ('youngstown', 5768), ('correctionsandupdatesforeignpolicyhistoryhumanrightsmilitary', 5769), ('100yearold', 5770), ('norm', 5771), ('educationmessagemachine2012', 5772), ('legalissuesterrorism', 5773), ('crimediversityhomelandsecurityterrorism', 5774), ('mosques', 5775), ('bernardino', 5776), ('deficiteconomypunditsabcnewsweek', 5777), ('crimefederalbudgethealthcaremessagemachine', 5778), ('viagra', 5779), ('molesters', 5780), ('publicservicestatebudget', 5781), ('woefully', 5782), ('underpaid', 5783), ('childrenvotingrecord', 5784), ('convict', 5785), ('housingnewhampshire2012', 5786), ('abortionhealthcarereligionstates', 5787), ('childrencrime', 5788), ('molesting', 5789), ('candidatesbiographycitybudgetcitygovernmenttransportation', 5790), ('austinincluding', 5791), ('jobaccomplishmentsstatebudgettaxes', 5792), ('legalissuesstates', 5793), ('correctionsandupdatesethics', 5794), ('falkland', 5795), ('environmentmessagemachinestimulus', 5796), ('marshall', 5797), ('dmacon', 5798), ('debatesdeficitfederalbudget', 5799), ('deficithistorystatefinancesstates', 5800), ('civilrightsgaysandlesbiansreligionworkers', 5801), ('teddy', 5802), ('kennedys', 5803), ('federalbudgetmedicaremilitarysocialsecurityabcnewsweek', 5804), ('jobslegalissuespunditsworkers', 5805), ('electionspolls', 5806), ('jobaccomplishmentsmessagemachine', 5807), ('hodges', 5808), ('badly', 5809), ('botched', 5810), ('gunsvotingrecord', 5811), ('energyenvironmentoilspillpunditsabcnewsweek', 5812), ('federalbudgetstatebudgettaxes', 5813), ('clothing', 5814), ('debatesdebtdeficiteconomyjobaccomplishmentscampaignadvertisingpensionsunionsworkers', 5815), ('pensionsstatebudgetstatefinancesstates', 5816), ('selffunded', 5817), ('environmentgovernmentregulationpublichealthpublicsafetyscience', 5818), ('hydrogen', 5819), ('sulfide', 5820), ('genocide', 5821), ('correctionsandupdatesenvironment', 5822), ('floridaamendments', 5823), ('campaignfinancecampaignadvertising', 5824), ('prostitution', 5825), ('allegedly', 5826), ('renaccis', 5827), ('corporationsjobstaxestradeworkers', 5828), ('debatesmedicare', 5829), ('baseballcitybudgetcountybudget', 5830), ('marlins', 5831), ('incurred', 5832), ('tourists', 5833), ('visiting', 5834), ('resident', 5835), ('campaignfinanceworkers', 5836), ('implied', 5837), ('rebates', 5838), ('directed', 5839), ('climatechangecongresseconomyenergyenvironmentmarketregulationvotingrecord', 5840), ('supportive', 5841), ('schrader', 5842), ('explain', 5843), ('legalissuespatriotism', 5844), ('violatedfederal', 5845), ('scuba', 5846), ('diving', 5847), ('gunssupremecourt', 5848), ('childrencrimecriminaljusticesexuality', 5849), ('recovered', 5850), ('disabilitymedicaidstatebudget', 5851), ('communitybased', 5852), ('candidatesbiographyenvironmenthomelandsecuritymessagemachineterrorism', 5853), ('boxers', 5854), ('civilrightsgaysandlesbianshousinglegalissuessexualityworkers', 5855), ('transgender', 5856), ('criminaljusticedebates', 5857), ('isunder', 5858), ('militaryvotingrecord', 5859), ('kurds', 5860), ('foreignpolicyhomelandsecuritymilitary', 5861), ('citygovernmentcorrectionsandupdates', 5862), ('draw', 5863), ('healthcarelegalissueswomen', 5864), ('electionspunditsterrorism', 5865), ('ayers', 5866), ('candidatesbiographypovertyreligion', 5867), ('preaches', 5868), ('keeper', 5869), ('aunt', 5870), ('kenya', 5871), ('debatesforeignpolicyisrael', 5872), ('climatechangeenvironmentforeignpolicy', 5873), ('polluted', 5874), ('governmentregulationhealthcaremessagemachine2012marketregulation', 5875), ('bipartisanshipcandidatesbiographyhistoryjobaccomplishments', 5876), ('cosponsor', 5877), ('economygovernmentefficiencymessagemachinetaxes', 5878), ('suffered', 5879), ('economyenergyenvironmentjobslabor', 5880), ('childrencrimejobaccomplishments', 5881), ('22000', 5882), ('protocol', 5883), ('addresses', 5884), ('downloaded', 5885), ('correctionsandupdatesfederalbudgethunger', 5886), ('appropriations', 5887), ('deprived', 5888), ('federalbudgetgovernmentefficiencyhealthcare', 5889), ('congressdiversity', 5890), ('diverse', 5891), ('medicareretirement', 5892), ('12500', 5893), ('candidatesbiographyjobaccomplishmentsstatebudgettaxes', 5894), ('stone', 5895), ('statebudgetstatefinancestaxes', 5896), ('816', 5897), ('economyfinancialregulationincome', 5898), ('foreignpolicypoverty', 5899), ('jobaccomplishmentstransportation', 5900), ('attracting', 5901), ('animalscrimeenvironmentgovernmentregulationgunspunditsrecreationmarketregulation', 5902), ('baiting', 5903), ('dem', 5904), ('technologytransparency', 5905), ('crimefamilieswomen', 5906), ('spouse', 5907), ('afghanistangovernmentefficiencyiraqmilitary', 5908), ('environmentpunditsrecreationscience', 5909), ('fishing', 5910), ('foreignpolicyhistorypunditsreligionterrorism', 5911), ('conspirators', 5912), ('ayman', 5913), ('zawahiri', 5914), ('khalid', 5915), ('sheikh', 5916), ('mohammed', 5917), ('gunsislamterrorism', 5918), ('solution', 5919), ('childrendisabilityhealthcarehistorymedicaresocialsecurity', 5920), ('widows', 5921), ('orphans', 5922), ('candidatesbiographymessagemachine2012', 5923), ('kb', 5924), ('toys', 5925), ('herald', 5926), ('disgusting', 5927), ('agricultureeconomygovernmentregulationjobslaborunionsworkers', 5928), ('debtdeficitfederalbudget', 5929), ('congresslaborvotingrecordwomen', 5930), ('immigrationtransportation', 5931), ('sharp', 5932), ('workrelated', 5933), ('english', 5934), ('countygovernmentcrimecriminaljusticelegalissuespublicsafety', 5935), ('deferred', 5936), ('firstoffenders', 5937), ('candidatesbiographydeficitstatebudgetstatefinancestaxes', 5938), ('turns', 5939), ('chinacorporationsfederalbudgetpublicserviceworkers', 5940), ('enterprise', 5941), ('childrenhealthcarepublichealth', 5942), ('unrecognized', 5943), ('congenital', 5944), ('electionslegalissues', 5945), ('sudafed', 5946), ('palin', 5947), ('buchanan', 5948), ('healthcarehistorytaxes', 5949), ('obamacarerepresents', 5950), ('electionsmilitary', 5951), ('distinction', 5952), ('availability', 5953), ('versus', 5954), ('citybudgetcitygovernmentpopulation', 5955), ('capandtradeclimatechangeenergyenvironmentmessagemachine2012', 5956), ('climatechangeeconomyenergygovernmentregulationmarketregulation', 5957), ('educationhealthcaretaxes', 5958), ('foreignpolicypunditsabcnewsweek', 5959), ('abortioncampaignfinanceelectionsfederalbudgethealthcarewomen', 5960), ('healthcarevotingrecord', 5961), ('campaignfinancecorporationsabcnewsweektransparency', 5962), ('entities', 5963), ('federalbudgethealthcaretechnology', 5964), ('corporationsdiversity', 5965), ('apples', 5966), ('cook', 5967), ('crimehistorypublicsafety', 5968), ('gunstechnology', 5969), ('federalbudgetmessagemachine2012taxesvotingrecord', 5970), ('correctionsandupdatesislamterrorism', 5971), ('pakistan', 5972), ('viewisis', 5973), ('favorably', 5974), ('unfortunately', 5975), ('economyfederalbudgetpundits', 5976), ('gross', 5977), ('childreneducationfloridaamendments', 5978), ('foreignpolicymessagemachine2012militaryterrorism', 5979), ('hollywood', 5980), ('invited', 5981), ('briefing', 5982), ('drugseducationmarijuana', 5983), ('surveyed', 5984), ('correctionsandupdatesfinancialregulationstatefinances', 5985), ('jobspunditsworkers', 5986), ('weak', 5987), ('underemployed', 5988), ('incomelegalissuesworkers', 5989), ('macys', 5990), ('urging', 5991), ('childrencrimecriminaljusticelegalissuestransportationurban', 5992), ('juveniles', 5993), ('juvenile', 5994), ('jobslabormilitarywealth', 5995), ('tyler', 5996), ('mcpherson', 5997), ('studio', 5998), ('candidatesbiographycongressfloridaforeignpolicyhistoryjobaccomplishmentscampaignadvertisingvotingrecord', 5999), ('attendance', 6000), ('candidatesbiographycitygovernmentcrimecriminaljustice', 6001), ('mayoral', 6002), ('adler', 6003), ('active', 6004), ('hall', 6005), ('deficitfederalbudgetmilitary', 6006), ('corporationseconomylaboroccupywallstreetworkers', 6007), ('475', 6008), ('censusimmigration', 6009), ('anglos', 6010), ('ten', 6011), ('homelandsecuritypublicsafety', 6012), ('keene', 6013), ('militarygrade', 6014), ('armored', 6015), ('citing', 6016), ('pumpkin', 6017), ('debateseducationstates', 6018), ('candidatesbiographycriminaljusticejobaccomplishmentslegalissuescampaignadvertisingpublicservicetechnology', 6019), ('misconduct', 6020), ('disgrace', 6021), ('embarrassment', 6022), ('crimegunslegalissues', 6023), ('unequivocally', 6024), ('groundlaw', 6025), ('bankruptcyeconomyhistoryhousingincomejobaccomplishmentspunditsabcnewsweekworkers', 6026), ('worsened', 6027), ('congressjobaccomplishmentspolls', 6028), ('economyincomejobswomen', 6029), ('familieshealthcareworkers', 6030), ('familieshealthcarereligion', 6031), ('countybudgettransportation', 6032), ('precinct', 6033), ('quick', 6034), ('desirable', 6035), ('ebolagovernmentefficiencypublicsafety', 6036), ('supplier', 6037), ('prepared', 6038), ('activated', 6039), ('economygovernmentregulationstatebudgetstatestaxes', 6040), ('instituted', 6041), ('humanrightsimmigrationisraelabcnewsweek', 6042), ('dubai', 6043), ('imported', 6044), ('agricultureimmigrationworkers', 6045), ('growers', 6046), ('dairies', 6047), ('crimehomelandsecuritylegalissuesterrorismabcnewsweek', 6048), ('miranda', 6049), ('educationstatebudgetstatefinances', 6050), ('130000', 6051), ('merit', 6052), ('debatesiraq', 6053), ('destabilize', 6054), ('bipartisanshipcitygovernmenteducationelections', 6055), ('clerk', 6056), ('uwgreen', 6057), ('afraid', 6058), ('bias', 6059), ('economyhistorytaxes', 6060), ('uninterrupted', 6061), ('religionwomen', 6062), ('clintoninsists', 6063), ('encounters', 6064), ('educationsports', 6065), ('heres', 6066), ('schiano', 6067), ('rutgers', 6068), ('marijuanawelfare', 6069), ('morgan', 6070), ('carroll', 6071), ('clubs', 6072), ('dispensaries', 6073), ('debatesisrael', 6074), ('candidatesbiographyethicsfinancialregulationlegalissues', 6075), ('nationsbank', 6076), ('bipartisanshipcampaignfinancecandidatesbiography', 6077), ('alameel', 6078), ('congressdebatesredistricting', 6079), ('crimeimmigration', 6080), ('candidatesbiographycitybudgetcitygovernmentmessagemachine2012', 6081), ('frankel', 6082), ('marble', 6083), ('shower', 6084), ('economyjobsmessagemachine2012', 6085), ('65000', 6086), ('ambitions', 6087), ('dnc', 6088), ('chairman', 6089), ('governorship', 6090), ('corporationsincomestatebudgettaxeswealth', 6091), ('610', 6092), ('1400', 6093), ('educationretirement', 6094), ('98000', 6095), ('53000', 6096), ('bipartisanshipdebatesmedicare', 6097), ('endorses', 6098), ('romneyryan', 6099), ('drugsflorida', 6100), ('affiliated', 6101), ('educationhealthcare', 6102), ('pe', 6103), ('corporationseconomyfinancialregulationmarketregulation', 6104), ('guarantees', 6105), ('agriculturegasprices', 6106), ('statestransportation', 6107), ('pip', 6108), ('foreignpolicymilitarysupremecourtterrorism', 6109), ('merrick', 6110), ('garland', 6111), ('federalbudgethistorypovertypundits', 6112), ('1965', 6113), ('untold', 6114), ('budged', 6115), ('energyenvironmentoilspillabcnewsweek', 6116), ('to', 6117), ('segments', 6118), ('sand', 6119), ('barriers', 6120), ('citybudgetcitygovernment', 6121), ('supporting', 6122), ('phillys', 6123), ('schoolchildren', 6124), ('handling', 6125), ('looming', 6126), ('legalissuessupremecourt', 6127), ('consumersafetygovernmentregulationpublicsafetymarketregulation', 6128), ('consultation', 6129), ('input', 6130), ('buckyballs', 6131), ('abortioncandidatesbiographymessagemachine', 6132), ('healthcarehousingincometaxes', 6133), ('civilrightsdisabilityincomejobsmarketregulationworkers', 6134), ('countybudgetenvironment', 6135), ('jackow', 6136), ('2020', 6137), ('censuseconomyhousingpopulation', 6138), ('ethicslegalissuespublicservicestates', 6139), ('coopers', 6140), ('drugsfederalbudgetmessagemachine2012stimulus10newstampabay', 6141), ('144541', 6142), ('monkeys', 6143), ('react', 6144), ('afterthefactalcoholdrugsmarijuana', 6145), ('editor', 6146), ('jobslaborstatebudget', 6147), ('educationgaysandlesbians', 6148), ('educational', 6149), ('promoting', 6150), ('glbt', 6151), ('lifestyle', 6152), ('federalbudgetmessagemachine2012taxes', 6153), ('consumersafetylegalissues', 6154), ('correctionsandupdateshealthcarepublichealth', 6155), ('vaccination', 6156), ('campaignfinancecandidatesbiographyelectionshistorycampaignadvertising', 6157), ('lyin', 6158), ('histv', 6159), ('stations', 6160), ('jobaccomplishmentsmessagemachine2012statebudgetstatefinances', 6161), ('debatesdrugshealthcaresexualitywomen', 6162), ('housingpovertyworkers', 6163), ('employed', 6164), ('correctionsandupdateseducationgunsmilitaryveterans', 6165), ('handing', 6166), ('pamphlet', 6167), ('titled', 6168), ('attempts', 6169), ('correctionsandupdatesstimulustaxes', 6170), ('economyjobstaxesworkers', 6171), ('soup', 6172), ('kitchens', 6173), ('deficiteconomyhealthcare', 6174), ('abortionstates', 6175), ('minor', 6176), ('tattoo', 6177), ('marketregulation', 6178), ('houses', 6179), ('weighs', 6180), ('340', 6181), ('immigrationvotingrecord', 6182), ('deport', 6183), ('dreamers', 6184), ('healthcarepublichealthstatebudgetstatefinances', 6185), ('absorbed', 6186), ('economynewhampshire2012statestaxes', 6187), ('utah', 6188), ('numberone', 6189), ('agricultureconsumersafety', 6190), ('careless', 6191), ('attitude', 6192), ('traced', 6193), ('citygovernmentguns', 6194), ('buyback', 6195), ('memphis', 6196), ('homelandsecuritylegalissuesmilitaryterrorism', 6197), ('190', 6198), ('childrenfamiliesstatebudget', 6199), ('custody', 6200), ('economyhistoryjobs', 6201), ('economyhistoryworkers', 6202), ('have', 6203), ('considerably', 6204), ('censuscriminaljusticeeducation', 6205), ('gunshealthcareprivacy', 6206), ('newest', 6207), ('unconstitutionally', 6208), ('professionals', 6209), ('violate', 6210), ('hipaa', 6211), ('justification', 6212), ('confiscation', 6213), ('candidatesbiographyjobaccomplishmentspolls', 6214), ('rated', 6215), ('factuallychallenged', 6216), ('alcoholcandidatesbiography', 6217), ('rosemary', 6218), ('lehmberg', 6219), ('bottles', 6220), ('vodka', 6221), ('candidatesbiographyforeignpolicyhistorystates', 6222), ('capandtradeclimatechangeabcnewsweek', 6223), ('candidatesbiographydebates', 6224), ('megyn', 6225), ('moderator', 6226), ('electionsethicsgaysandlesbiansgunslegalissuesmarriage', 6227), ('richards', 6228), ('enforce', 6229), ('tourismweather', 6230), ('exists', 6231), ('pillars', 6232), ('alcoholdrugsmarijuanapublichealthstates', 6233), ('incomejobsworkers', 6234), ('debtfederalbudgettaxesvotingrecord', 6235), ('climatechangefederalbudgetscienceweather', 6236), ('forecasting', 6237), ('islamspace', 6238), ('nasa', 6239), ('flights', 6240), ('candidatesbiographychildren', 6241), ('wrotei', 6242), ('crimeimmigrationabcnewsweek', 6243), ('economygamblingjobsmessagemachine2012statebudgetstatefinancesworkers', 6244), ('twin', 6245), ('river', 6246), ('candidatesbiographysmallbusinessstimulus', 6247), ('sittons', 6248), ('650000', 6249), ('labortaxes', 6250), ('equity', 6251), ('nurses', 6252), ('educationscience', 6253), ('anthropology', 6254), ('engineering', 6255), ('climatechangeenvironmentpolls', 6256), ('wisdom', 6257), ('historypunditsveterans', 6258), ('critics', 6259), ('educationstatebudget', 6260), ('campaignfinanceenergyoilspillpundits', 6261), ('federalbudgetgovernmentefficiencynewhampshire2012transportation', 6262), ('limousines', 6263), ('73', 6264), ('economyhistoryjobsstates', 6265), ('climatechangeenergygasprices', 6266), ('circle', 6267), ('economystatebudgettaxes', 6268), ('childreneducationstatefinances', 6269), ('educationhealthcareimmigrationvotingrecord', 6270), ('dewhursts', 6271), ('energygovernmentregulationnuclearmarketregulation', 6272), ('publicsafetystatebudgettransportation', 6273), ('rebuilt', 6274), ('interchange', 6275), ('candidatesbiographyethicsmessagemachinestatebudgettaxes', 6276), ('32000', 6277), ('spiral', 6278), ('staircase', 6279), ('ethicstaxes', 6280), ('matt', 6281), ('bevin', 6282), ('candidatesbiographycorrectionsandupdatestaxes', 6283), ('bermuda', 6284), ('disclosures', 6285), ('corporationseconomyjobslaborworkers', 6286), ('environmentguns', 6287), ('smelter', 6288), ('jobstourism', 6289), ('educationreligion', 6290), ('skipping', 6291), ('countygovernmentstatefinancesstates', 6292), ('congresswealth', 6293), ('ballooned', 6294), ('healthcareprivacytechnology', 6295), ('hidden', 6296), ('users', 6297), ('waive', 6298), ('reasonable', 6299), ('candidatesbiographymilitary', 6300), ('debatenot', 6301), ('federalbudgethealthcaremedicareretirement', 6302), ('design', 6303), ('afghanistanhistory', 6304), ('vietnams', 6305), ('exchanging', 6306), ('alcoholdrugsmarijuana', 6307), ('citygovernmentcivilrightscountygovernmentgaysandlesbianssexuality', 6308), ('existed', 6309), ('publichealthscience', 6310), ('fluoride', 6311), ('statebudgettaxes', 6312), ('leaner', 6313), ('bipartisanshipjobaccomplishmentsvotingrecord', 6314), ('differed', 6315), ('candidatesbiographyeconomyjobaccomplishmentsjobs', 6316), ('doughnuts', 6317), ('830', 6318), ('educationforeignpolicypublichealthwomen', 6319), ('chance', 6320), ('childrencrimefamilies', 6321), ('dropouts', 6322), ('economyforeignpolicyhistorytrade', 6323), ('curbing', 6324), ('1930s', 6325), ('painful', 6326), ('healthcaremedicareprivacypublichealth', 6327), ('retirementsocialsecuritystatebudget', 6328), ('faces', 6329), ('longterm', 6330), ('healthcareincomelaborpublichealthworkers', 6331), ('goods', 6332), ('globally', 6333), ('candidatesbiographycorrectionsandupdatesvotingrecord', 6334), ('thensen', 6335), ('socialsecuritytaxesvotingrecord', 6336), ('holidays', 6337), ('animalsenvironment', 6338), ('450000', 6339), ('lions', 6340), ('mid70s', 6341), ('governmentregulationgunshistorypublicsafetypundits', 6342), ('cspan', 6343), ('sitin', 6344), ('incomestatebudget', 6345), ('jobslegalissuessmallbusiness', 6346), ('litigation', 6347), ('affecting', 6348), ('campaignfinancecorporationscorrectionsandupdateselections', 6349), ('jobaccomplishmentslegalissuesmessagemachine', 6350), ('educationhealthcaremessagemachine2012', 6351), ('autism', 6352), ('foreignpolicyiraqisraelmilitary', 6353), ('reveal', 6354), ('britishand', 6355), ('criminaljusticeeducationstatebudget', 6356), ('energymarketregulation', 6357), ('lamp', 6358), ('cfl', 6359), ('abortioncongress', 6360), ('katko', 6361), ('immigrationislam', 6362), ('pew', 6363), ('oppressive', 6364), ('economyenergyenvironmentjobs', 6365), ('candidatesbiographycivilrightssupremecourtwomen', 6366), ('correctionsandupdateseconomyjobsmessagemachine2012', 6367), ('homelandsecurityimmigrationmilitary', 6368), ('energytrade', 6369), ('oilproducing', 6370), ('selfimposed', 6371), ('exporting', 6372), ('crude', 6373), ('educationflorida', 6374), ('averages', 6375), ('campaignfinancestatefinancestaxes', 6376), ('177', 6377), ('economyimmigrationpublicsafetymarketregulation', 6378), ('bipartisanshipelectionsredistricting', 6379), ('candidatesbiographyeconomyjobaccomplishments', 6380), ('remodeling', 6381), ('publicsafetypublicservice', 6382), ('economyincomejobaccomplishments', 6383), ('citygovernmenthousingpovertypublicsafetyurban', 6384), ('renaissance', 6385), ('countygovernmentpublichealth', 6386), ('kid', 6387), ('ate', 6388), ('sugar', 6389), ('stimulusvotingrecord', 6390), ('barrow', 6391), ('950k', 6392), ('genetic', 6393), ('makeup', 6394), ('healthcarestimulusvotingrecord', 6395), ('phil', 6396), ('13m', 6397), ('familiesfederalbudgetmilitary', 6398), ('fantasy', 6399), ('citygovernmenteconomygovernmentregulationmarketregulationstates', 6400), ('rhodemap', 6401), ('healthcarelegalissuessupremecourt', 6402), ('throws', 6403), ('democratically', 6404), ('militarycampaignadvertisingpatriotism', 6405), ('browns', 6406), ('educationforeignpolicywomen', 6407), ('healthcaretaxes', 6408), ('weidner', 6409), ('oregons', 6410), ('incometaxes', 6411), ('perks', 6412), ('\u202abernie\u202c', 6413), ('jobaccomplishmentskagannominationlegalissuespunditssupremecourt', 6414), ('scholarly', 6415), ('procedural', 6416), ('drugsfederalbudgethealthcaresexuality', 6417), ('175587', 6418), ('japanese', 6419), ('quail', 6420), ('electionsabcnewsweek', 6421), ('hawaii', 6422), ('they', 6423), ('throwing', 6424), ('incumbents', 6425), ('correctionsandupdatesenergymessagemachine2012oilspill', 6426), ('healthcarestatestransparency', 6427), ('opportunities', 6428), ('countybudgetgovernmentefficiency', 6429), ('downsizing', 6430), ('pinellas', 6431), ('ms', 6432), ('latvala', 6433), ('economyenvironmentgovernmentregulationjobs', 6434), ('acknowledging', 6435), ('epas', 6436), ('burdensome', 6437), ('corporationsenergyoilspillpunditssciencetechnology', 6438), ('025', 6439), ('rd', 6440), ('candidatesbiographygovernmentregulationhealthcarereligion', 6441), ('socialists', 6442), ('economyeducationincome', 6443), ('bachelors', 6444), ('thereafter', 6445), ('publichealthtrade', 6446), ('threaten', 6447), ('indias', 6448), ('pharmacy', 6449), ('developing', 6450), ('medicines', 6451), ('candidatesbiographydebtdeficiteconomyjobs', 6452), ('menendezs', 6453), ('debteducation', 6454), ('upon', 6455), ('jobaccomplishmentsstatefinancesvotingrecord', 6456), ('spotty', 6457), ('historypatriotismstates', 6458), ('founders', 6459), ('motto', 6460), ('economyenergyhistoryjobsworkers', 6461), ('proving', 6462), ('citybudgetjobaccomplishmentscampaignadvertisingtaxes', 6463), ('crimepoverty', 6464), ('citygovernmenteconomyincome', 6465), ('inequality', 6466), ('correctionsandupdatesfederalbudgethealthcaremedicareabcnewsweek', 6467), ('animalseconomyenvironmentmarketregulation', 6468), ('menhaden', 6469), ('plummeted', 6470), ('infrastructuretransportation', 6471), ('incomejobslaborwomenworkers', 6472), ('calling', 6473), ('senseless', 6474), ('educationfamiliesimmigrationmilitary', 6475), ('willing', 6476), ('citybudgetdiversitylegalissues', 6477), ('31000', 6478), ('visitors', 6479), ('surviving', 6480), ('tickets', 6481), ('messagemachine2014votingrecord', 6482), ('ninetyseven', 6483), ('correctionsandupdatescrimecriminaljusticeguns', 6484), ('type', 6485), ('frequency', 6486), ('agricultureanimals', 6487), ('coyotes', 6488), ('economyimmigrationretirementsocialsecurity', 6489), ('wantsillegal', 6490), ('collectingsocial', 6491), ('federalbudgetpopculture', 6492), ('warcraft', 6493), ('candidatesbiographypensions', 6494), ('candidatesbiographyeducationfederalbudgetjobaccomplishments', 6495), ('healthcaremarriage', 6496), ('laborwomen', 6497), ('federalbudgetforeignpolicyhistory', 6498), ('foreigners', 6499), ('drugsimmigrationterrorism', 6500), ('qaeda', 6501), ('jobspundits', 6502), ('mcmansion', 6503), ('lagged', 6504), ('connecticut', 6505), ('obvious', 6506), ('comparison', 6507), ('crimepopulationpublicsafetyurban', 6508), ('96', 6509), ('countybudgetsportstaxes', 6510), ('dolphins', 6511), ('hungerpovertypunditswelfare', 6512), ('abortionforeignpolicygunshealthcarehumanrightsimmigrationislamreligionterrorism', 6513), ('ebolamessagemachine2014publichealthvotingrecord', 6514), ('saystom', 6515), ('preparing', 6516), ('pandemics', 6517), ('childrencrimecriminaljustice', 6518), ('notifications', 6519), ('offender', 6520), ('childrenfamilieshealthcaremedicaidmedicare', 6521), ('countybudgetcountygovernmentcrimeelections', 6522), ('providing', 6523), ('candidatesbiographycongressfederalbudgethomelandsecurityvotingrecordweather', 6524), ('fema', 6525), ('economypunditsretirement', 6526), ('guessed', 6527), ('corporationsenergyenvironmentgovernmentefficiencymessagemachine2012stimulus', 6528), ('abortionhealthcarestates', 6529), ('afghanistanenergy', 6530), ('corporationsenergyenvironmentethicsgovernmentefficiencyoilspillmarketregulation', 6531), ('showered', 6532), ('regulators', 6533), ('gifts', 6534), ('write', 6535), ('campaignfinancecandidatesbiographycongresstaxesworkers', 6536), ('federalbudgetfinancialregulationinfrastructuremessagemachine2012stimulustransportation', 6537), ('correctionsandupdateseconomyenergyiraq', 6538), ('rises', 6539), ('cent', 6540), ('abillion', 6541), ('dollarsout', 6542), ('candidatesbiographyethicsstatebudget', 6543), ('xiv', 6544), ('economystimulusworkers', 6545), ('consumersafetycrime', 6546), ('accident', 6547), ('federalbudgethistoryjobaccomplishments', 6548), ('1997', 6549), ('censuspopulation', 6550), ('34yearolds', 6551), ('energyenvironmentguns', 6552), ('rack', 6553), ('chevrolet', 6554), ('volt', 6555), ('economyjobaccomplishmentsjobspovertyworkers', 6556), ('onehundred', 6557), ('federalbudgethealthcareterrorism', 6558), ('responders', 6559), ('corporationseconomyjobstaxes', 6560), ('gaysandlesbiansreligion', 6561), ('warren', 6562), ('federalbudgetsports', 6563), ('kingston', 6564), ('baseballcountybudgetdebteconomylegalissues', 6565), ('educationpunditsstatebudget', 6566), ('snack', 6567), ('humanrightsislamreligion', 6568), ('genital', 6569), ('mutilation', 6570), ('maher', 6571), ('alcoholcrimecriminaljusticepublicsafetytransportation', 6572), ('005', 6573), ('glass', 6574), ('dinner', 6575), ('federalbudgethealthcarehistorymedicarepollspublichealth', 6576), ('truman', 6577), ('economyjobaccomplishmentsstatefinances', 6578), ('star', 6579), ('climatechangeenergygaspricesmessagemachine2012', 6580), ('economynewhampshire2012', 6581), ('weekend', 6582), ('groceries', 6583), ('educationstatebudgetstimulus', 6584), ('environmentpublichealth', 6585), ('mosquito', 6586), ('governmentregulationjobspovertywelfare', 6587), ('abortionhealthcaremessagemachine2014publichealthtechnologywomen', 6588), ('transvaginal', 6589), ('electionscampaignadvertising', 6590), ('victors', 6591), ('correctionsandupdatesdebt', 6592), ('electionsimmigrationpopulation', 6593), ('hispanics', 6594), ('demographic', 6595), ('campaignfinancecandidatesbiographyelectionsmessagemachine', 6596), ('gores', 6597), ('economyfederalbudgetstimulustaxes', 6598), ('crimeurban', 6599), ('panhandlingrelated', 6600), ('offenses', 6601), ('78', 6602), ('capandtradeclimatechangejobs', 6603), ('historyhomelandsecuritylegalissuestransparency', 6604), ('delete', 6605), ('33000', 6606), ('climatechangeeconomyenergyjobs', 6607), ('outpaces', 6608), ('economyhealthcarejobsoccupywallstreetpovertyworkers', 6609), ('congressionalrulesfederalbudget', 6610), ('unless', 6611), ('bankruptcycandidatesbiographytaxes', 6612), ('bipartisanshipcongressenvironmentfederalbudgethomelandsecurityinfrastructurepublicsafetyvotingrecordweather', 6613), ('arkansan', 6614), ('congresselections', 6615), ('legalissuessocialsecurity', 6616), ('foreignpolicylegalissues', 6617), ('acceptance', 6618), ('gold', 6619), ('medal', 6620), ('medicaidreligionretirement', 6621), ('kosher', 6622), ('healthcaremessagemachine2014military', 6623), ('educationgovernmentefficiencyjobs', 6624), ('crimecriminaljusticesupremecourt', 6625), ('responding', 6626), ('assertion', 6627), ('criminaljusticegovernmentregulationgunspublichealth', 6628), ('occurring', 6629), ('licensed', 6630), ('economyincomeretirementsocialsecuritywealth', 6631), ('110th', 6632), ('federalbudgethealthcareveterans', 6633), ('slashed', 6634), ('environmentfinancialregulationgovernmentregulationprivacy', 6635), ('spying', 6636), ('innocent', 6637), ('booker', 6638), ('correctionsandupdatesdiversityjobs', 6639), ('resume', 6640), ('nameresume', 6641), ('historymilitaryreligion', 6642), ('declared', 6643), ('jefferson', 6644), ('transportationunions', 6645), ('rebidding', 6646), ('metrorail', 6647), ('guaranteed', 6648), ('foreignpolicyislamterrorism', 6649), ('jihadists', 6650), ('swim', 6651), ('pool', 6652), ('stimulusworkers', 6653), ('floridastates', 6654), ('kriseman', 6655), ('ineffective', 6656), ('krisemans', 6657), ('abortioncorrectionsandupdatesmessagemachine2012', 6658), ('healthcaremedicarenewhampshire2012', 6659), ('occupywallstreet', 6660), ('foreignpolicy', 6661), ('by', 6662), ('jobaccomplishmentsjobstaxes', 6663), ('economyfederalbudgetjobs', 6664), ('debtmedicaidstatebudgetstatefinancestaxes', 6665), ('energyfederalbudgetstimulus', 6666), ('candidatesbiographychildrencrime', 6667), ('stephen', 6668), ('webber', 6669), ('feet', 6670), ('childcare', 6671), ('playgrounds', 6672), ('coaches', 6673), ('jobaccomplishmentstaxesvotingrecord', 6674), ('inaccuracies', 6675), ('multiple', 6676), ('unincorporated', 6677), ('animalsgunsmessagemachine2012publicsafety', 6678), ('healthcarejobs', 6679), ('afghanistaniraqveterans', 6680), ('post911', 6681), ('civilrightsgaysandlesbiansmarriagepolls', 6682), ('economymedicaid', 6683), ('bipartisanshipgovernmentefficiency', 6684), ('solitary', 6685), ('healthcaremessagemachine2012publichealth', 6686), ('thanked', 6687), ('conducting', 6688), ('educationfederalbudgetmarketregulationsciencetechnology', 6689), ('barrier', 6690), ('innovation', 6691), ('crimegunswomen', 6692), ('candidatesbiographyforeignpolicymessagemachine2012sports', 6693), ('winter', 6694), ('burma', 6695), ('economyenergyenvironmentinfrastructurestatebudgettransportation', 6696), ('ripta', 6697), ('fullest', 6698), ('candidatesbiographycrimecriminaljustice', 6699), ('verona', 6700), ('swanigan', 6701), ('drugsgovernmentregulationmarijuanamarketregulationscience', 6702), ('economyreligion', 6703), ('compliant', 6704), ('specializes', 6705), ('compliance', 6706), ('immigrationmessagemachine', 6707), ('richardsondenish', 6708), ('citybudgetcitygovernmenteconomyjobs', 6709), ('shore', 6710), ('crimecriminaljusticegunspublicsafety', 6711), ('finds', 6712), ('foreignpolicyhomelandsecurityhumanrightsimmigrationterrorism', 6713), ('550', 6714), ('screen', 6715), ('healthcarepollspublichealth', 6716), ('childrenconsumersafety', 6717), ('1980', 6718), ('documented', 6719), ('suction', 6720), ('entrapment', 6721), ('swimming', 6722), ('pools', 6723), ('spas', 6724), ('electionsgovernmentregulationlegalissuesstates', 6725), ('corporationsfinancialregulationgovernmentregulation', 6726), ('cromnibus', 6727), ('incredibly', 6728), ('economyincometaxeswealth', 6729), ('onetenth', 6730), ('federalbudgetjobaccomplishmentsmessagemachine2012military', 6731), ('civilrightscrimegaysandlesbiansreligion', 6732), ('hatecrimes', 6733), ('preach', 6734), ('homosexuality', 6735), ('congresscongressionalrulesethicslegalissuesvotingrecord', 6736), ('insider', 6737), ('trading', 6738), ('brave', 6739), ('citygovernmentpopulation', 6740), ('historyinfrastructurelegalissuestransportation', 6741), ('transferred', 6742), ('tollfree', 6743), ('baseballeconomyrecreationsports', 6744), ('economists', 6745), ('arising', 6746), ('franchises', 6747), ('naturaldisasters', 6748), ('fulfilled', 6749), ('assisting', 6750), ('energyhistoryoilspillmarketregulation', 6751), ('economyfederalbudgetgovernmentefficiencysmallbusinessstatebudgetstatefinancestaxesworkers', 6752), ('ornamental', 6753), ('holders', 6754), ('governmentefficiencypovertystatebudgettaxes', 6755), ('generosity', 6756), ('corporationseconomytaxes', 6757), ('bipartisanshipcongresshistorylegalissuessupremecourt', 6758), ('lasting', 6759), ('237', 6760), ('economystatebudgetstatefinancesworkers', 6761), ('citygovernmenteducationmessagemachine2012votingrecord', 6762), ('salem', 6763), ('educationjobaccomplishments', 6764), ('measured', 6765), ('amongst', 6766), ('familiesincome', 6767), ('57000', 6768), ('homelandsecurityimmigrationabcnewsweek', 6769), ('are', 6770), ('apprehensions', 6771), ('indicates', 6772), ('crossings', 6773), ('healthcaremedicare10newstampabay', 6774), ('crimehomelandsecurityimmigration', 6775), ('crimecriminaljusticelegalissuespublicsafetystates', 6776), ('drugspublicsafety', 6777), ('bipartisanshiphistory', 6778), ('gridlocked', 6779), ('foreignpolicyhistoryiraq', 6780), ('thatcher', 6781), ('hw', 6782), ('wobbly', 6783), ('energymessagemachine2012stimulus', 6784), ('payback', 6785), ('environmentgunshealthcarepublichealth', 6786), ('citybudgetstatebudgetunions', 6787), ('diversityhomelandsecuritylegalissuesreligion', 6788), ('profiling', 6789), ('energyoilspillmarketregulation', 6790), ('civilrightshistory', 6791), ('crowera', 6792), ('debthistorypundits', 6793), ('federalbudgetstatebudgetstatestaxes', 6794), ('sixth', 6795), ('crimegunshistorypublicsafety', 6796), ('victimsoutnumbers', 6797), ('bipartisanshipcandidatesbiography', 6798), ('railroad', 6799), ('williams', 6800), ('attended', 6801), ('gatherings', 6802), ('candidatesbiographyhomelandsecurity', 6803), ('crimepublicsafety', 6804), ('collected', 6805), ('kasim', 6806), ('reed', 6807), ('correctionsandupdatesfederalbudget', 6808), ('crimeelectionstechnology', 6809), ('abortioncandidatesbiographycivilrightslegalissueswomen', 6810), ('restrictive', 6811), ('infrastructurewater', 6812), ('candidatesbiographycrimehomelandsecuritypublicsafetystatebudgetstatefinances', 6813), ('lieutenant', 6814), ('detail', 6815), ('debateseducation', 6816), ('contests', 6817), ('messagemachine2012statebudgetstatefinancestaxes', 6818), ('shortfall', 6819), ('corporationseconomyhealthcarepublichealthworkers', 6820), ('animalsgamblingtourism', 6821), ('dogracing', 6822), ('governmentregulationstatestransportation', 6823), ('uber', 6824), ('rideshare', 6825), ('candidatesbiographymessagemachine', 6826), ('wwe', 6827), ('linda', 6828), ('mcmahon', 6829), ('tipping', 6830), ('ringside', 6831), ('distributing', 6832), ('steroids', 6833), ('wrestlers', 6834), ('candidatesbiographygovernmentefficiency', 6835), ('pawlenty', 6836), ('proactive', 6837), ('aggressive', 6838), ('countygovernmentjobsstatebudget', 6839), ('commonwealth', 6840), ('corporationssmallbusiness', 6841), ('referred', 6842), ('lego', 6843), ('insidious', 6844), ('conspiracy', 6845), ('abortionfamiliesgovernmentregulationhealthcarepublichealthpublicsafetyreligionsciencestatebudgetwomen', 6846), ('providers', 6847), ('afghanistaniraqmilitarystates', 6848), ('secondmost', 6849), ('heavily', 6850), ('deployed', 6851), ('electionslegalissuesredistricting', 6852), ('bribe', 6853), ('healthcarehistorymessagemachinestimulus', 6854), ('bennet', 6855), ('criminaljusticediversityeconomyjobs', 6856), ('overincarceration', 6857), ('healthcaremedicaidpoverty', 6858), ('animalscitygovernment', 6859), ('dccitycouncilpasseda', 6860), ('lawbanninglethalrat', 6861), ('trapping', 6862), ('healthcarelaborstatebudget', 6863), ('deficitlotterystatefinances', 6864), ('sufficient', 6865), ('reserves', 6866), ('tennessees', 6867), ('scholarship', 6868), ('jobsmessagemachine2012women', 6869), ('diversitypopulationterrorism', 6870), ('somali', 6871), ('delaware', 6872), ('economypovertyworkers', 6873), ('bushadministrationpundits', 6874), ('felt', 6875), ('hating', 6876), ('corporationsethicsmarketregulationabcnewsweek', 6877), ('lobbying', 6878), ('familiesforeignpolicyhomelandsecurityhumanrightsimmigrationterrorism', 6879), ('massively', 6880), ('abortionhealthcarevotingrecord', 6881), ('sununus', 6882), ('accessed', 6883), ('economyjobaccomplishmentssmallbusiness', 6884), ('establishment', 6885), ('debtstatebudgetstatefinances', 6886), ('201517', 6887), ('foreignpolicyisraelmessagemachine2012', 6888), ('iraqabcnewsweek', 6889), ('islammessagemachine2012', 6890), ('disputes', 6891), ('electionsjobaccomplishmentsstatebudget', 6892), ('correctionsandupdatesimmigration', 6893), ('sonia', 6894), ('animmigrant', 6895), ('healthcaresciencesports', 6896), ('absurd', 6897), ('establish', 6898), ('link', 6899), ('chronic', 6900), ('traumatic', 6901), ('encephalopathy', 6902), ('childrendebteconomyeducation', 6903), ('terrorismtransparency', 6904), ('concerned', 6905), ('wiretappings', 6906), ('briefings', 6907), ('appropriately', 6908), ('struck', 6909), ('punditsreligion', 6910), ('foreignpolicyhumanrightsiraqmilitaryterrorism', 6911), ('carpet', 6912), ('bomb', 6913), ('deficiteconomyjobs', 6914), ('presiding', 6915), ('hurtling', 6916), ('financialregulationforeignpolicywater', 6917), ('manila', 6918), ('philippines', 6919), ('845', 6920), ('subsidiary', 6921), ('partial', 6922), ('campaignfinanceethicsmessagemachine', 6923), ('correctionsandupdatesterrorism', 6924), ('healthcarepunditsterrorism', 6925), ('detainees', 6926), ('h1n1', 6927), ('economystimulustaxesworkers', 6928), ('baseballcorporationseconomystatefinancesvotingrecord', 6929), ('loaning', 6930), ('drugswelfare', 6931), ('electionspensionsretirementstatebudgetstatefinancestransparency', 6932), ('knopp', 6933), ('upheld', 6934), ('disabilitystatebudget', 6935), ('abilities', 6936), ('candidatesbiographyfederalbudget', 6937), ('stupid', 6938), ('corrected', 6939), ('bipartisanshipchildrencongresscorrectionsandupdatesdebtdeficitfamiliesfederalbudget', 6940), ('scramble', 6941), ('familiesgaysandlesbiansmarriagereligion', 6942), ('kindergartners', 6943), ('energytaxesvotingrecord', 6944), ('candidatesbiographychildreneducationfamiliesgovernmentregulationhistory', 6945), ('saysgary', 6946), ('libertarian', 6947), ('criminaljusticegovernmentregulationgunshistorysupremecourt', 6948), ('dowith', 6949), ('toddlers', 6950), ('incomepensionsretirementtaxes', 6951), ('taxfriendly', 6952), ('kagannominationmilitarysupremecourtabcnewsweek', 6953), ('various', 6954), ('childrenconsumersafetydrugslegalissuespublichealthpublicsafetyrecreationmarketregulationscience', 6955), ('penn', 6956), ('addictive', 6957), ('healthcarelegalissuesmessagemachine', 6958), ('deposition', 6959), ('invoked', 6960), ('dealings', 6961), ('columbiahca', 6962), ('chain', 6963), ('economyhousingmarketregulation', 6964), ('crimehistory', 6965), ('kkk', 6966), ('historystates', 6967), ('economyfederalbudgetsocialsecuritytaxes', 6968), ('crimecriminaljusticeeducation', 6969), ('leticia', 6970), ('putte', 6971), ('publicsafetyunions', 6972), ('eliminates', 6973), ('citygovernmenteconomy', 6974), ('300step', 6975), ('chinadeficiteconomy', 6976), ('governmentefficiencyhousing', 6977), ('upgrading', 6978), ('energyoilspillmarketregulationabcnewsweekworkers', 6979), ('recommendations', 6980), ('them', 6981), ('countygovernment', 6982), ('48m', 6983), ('dept', 6984), ('crimecriminaljusticelegalissuesstates', 6985), ('northeastern', 6986), ('firstdegree', 6987), ('murderers', 6988), ('deficitjobstaxes', 6989), ('experienced', 6990), ('climatechangeenergy', 6991), ('elimination', 6992), ('offset', 6993), ('countybudgetcountygovernmenteconomyjobsmessagemachine2012', 6994), ('privacyterrorism', 6995), ('usa', 6996), ('naturaldisastersweather', 6997), ('amounting', 6998), ('superstorm', 6999), ('newhampshire2012taxes', 7000), ('gingrichs', 7001), ('bipartisanshipvotingrecord', 7002), ('93', 7003), ('abortionmessagemachine2012', 7004), ('included', 7005), ('childreneducation', 7006), ('underprivileged', 7007), ('meatbased', 7008), ('depriving', 7009), ('candidatesbiographyhistoryimmigration', 7010), ('nikki', 7011), ('haley', 7012), ('childrenfamiliespovertywelfare', 7013), ('correctionsandupdatesfederalbudgetpunditswealth', 7014), ('highestincome', 7015), ('ring', 7016), ('jobaccomplishmentsstatebudgetstatefinances', 7017), ('535', 7018), ('exact', 7019), ('afghanistanbushadministrationiraqterrorism', 7020), ('chinaeconomylabortradeworkers', 7021), ('familiesgaysandlesbians', 7022), ('traditional', 7023), ('defined', 7024), ('crimegunsterrorismtransportation', 7025), ('healthcarejobspundits', 7026), ('crimetransportation', 7027), ('blasio', 7028), ('subway', 7029), ('soared', 7030), ('jobaccomplishmentstaxes', 7031), ('economystatebudgetstatestaxes', 7032), ('outmigration', 7033), ('newsgoogle', 7034), ('des', 7035), ('moines', 7036), ('energyforeignpolicyoilspill', 7037), ('350', 7038), ('enemies', 7039), ('climatechangehomelandsecurityterrorism', 7040), ('dedicated', 7041), ('combating', 7042), ('radicalizing', 7043), ('debteducationtaxes', 7044), ('load', 7045), ('doubles', 7046), ('ebolaforeignpolicymilitarypublichealth', 7047), ('deployment', 7048), ('economyjobspovertywealth', 7049), ('sutton', 7050), ('riot', 7051), ('gaysandlesbianshealthcaresexuality', 7052), ('lifeby', 7053), ('economyiraqstimulus', 7054), ('historyretirementsocialsecurityworkers', 7055), ('sportstaxes', 7056), ('candidatesbiographydebatescampaignadvertising', 7057), ('carcieri', 7058), ('congressimmigration', 7059), ('obamareid', 7060), ('bipartisanshipcongressjobaccomplishments', 7061), ('105', 7062), ('afghanistanethicshumanrightsiraqmilitaryterrorism', 7063), ('saddam', 7064), ('10year', 7065), ('relationship', 7066), ('campaignfinancecongresscongressionalrulesjobaccomplishmentsmessagemachine2012campaignadvertising', 7067), ('james', 7068), ('supposedly', 7069), ('champion', 7070), ('womenworkers', 7071), ('earnonly', 7072), ('79', 7073), ('economyjobsnewhampshire2012poverty', 7074), ('slipped', 7075), ('citybudgetcitygovernmentdebtdeficiteconomyjobaccomplishmentslegalissuestransparency', 7076), ('locked', 7077), ('finances', 7078), ('smallbusinessworkers', 7079), ('religionsupremecourtcolbertreport', 7080), ('crimegovernmentregulationguns', 7081), ('chicago', 7082), ('stringent', 7083), ('taxesabcnewsweek', 7084), ('educationincome', 7085), ('educationimmigrationislam', 7086), ('worrying', 7087), ('bullying', 7088), ('muslimsand', 7089), ('economyforeignpolicytechnology', 7090), ('cubans', 7091), ('telecommunications', 7092), ('educationincomewealth', 7093), ('top25', 7094), ('congresspolls', 7095), ('incomejobsstatestaxes', 7096), ('battered', 7097), ('censusjobs', 7098), ('skew', 7099), ('season', 7100), ('hungerpopulation', 7101), ('valle', 7102), ('bastrop', 7103), ('closest', 7104), ('floridataxes', 7105), ('candidatesbiographyforeignpolicypolls', 7106), ('childreneducationgunspublicsafety', 7107), ('bankruptcyeconomygovernmentregulationhousingpovertystates', 7108), ('bipartisanshipeconomy', 7109), ('goal', 7110), ('energynuclearwater', 7111), ('rivers', 7112), ('hydropower', 7113), ('messagemachine2012campaignadvertisingtaxes', 7114), ('economyforeignpolicy', 7115), ('idiots', 7116), ('crimecriminaljusticedrugsmarijuananewhampshire2012', 7117), ('imprisoned', 7118), ('economystatestaxes', 7119), ('boats', 7120), ('spawned', 7121), ('publichealthveterans', 7122), ('economysports', 7123), ('mayweatherpacquiao', 7124), ('federalbudgetmessagemachine2012retirementsocialsecurity', 7125), ('game', 7126), ('candidatesbiographyeducationsmallbusinesstaxes', 7127), ('climatechangeenergyenvironmentgovernmentregulationhistoryscienceweather', 7128), ('temperatures', 7129), ('agricultureenvironmentwater', 7130), ('dumping', 7131), ('sewage', 7132), ('historypopculturereligionspace', 7133), ('holy', 7134), ('communion', 7135), ('financialregulationtaxes', 7136), ('candidatesbiographyeconomystatebudgettaxes', 7137), ('dave', 7138), ('brat', 7139), ('kaines', 7140), ('advisors', 7141), ('immigrationtrade', 7142), ('crimecriminaljusticegovernmentefficiencyjobaccomplishmentsstatefinances', 7143), ('900case', 7144), ('backlog', 7145), ('debtfederalbudgetmilitary', 7146), ('congressdecision', 7147), ('environmentpundits', 7148), ('prince', 7149), ('william', 7150), ('sound', 7151), ('pristine', 7152), ('environmentjobstransportation', 7153), ('invasive', 7154), ('species', 7155), ('ballast', 7156), ('campaignfinancecongressfinancialregulationgovernmentregulation', 7157), ('afghanistanislamreligionterrorism', 7158), ('sic', 7159), ('ideology', 7160), ('congressionalrulescriminaljustice', 7161), ('saysloretta', 7162), ('lynchs', 7163), ('economyfederalbudgettaxesabcnewsweek', 7164), ('citybudgetdebtjobaccomplishments', 7165), ('cumberland', 7166), ('deficitfederalbudgettrade', 7167), ('approaching', 7168), ('ireland', 7169), ('portugal', 7170), ('spain', 7171), ('chinaforeignpolicyterrorism', 7172), ('naval', 7173), ('exercises', 7174), ('someplace', 7175), ('jobswomenworkers', 7176), ('ebolahealthcareimmigrationpublichealth', 7177), ('amid', 7178), ('migrants', 7179), ('citygovernmentcivilrightsdiversitygovernmentregulationhousingpovertymarketregulationurbanwealth', 7180), ('patently', 7181), ('unjust', 7182), ('exclusive', 7183), ('retirementstatebudgetstatefinances', 7184), ('unsustainable', 7185), ('translates', 7186), ('813', 7187), ('electionsimmigrationpundits', 7188), ('jose', 7189), ('iraqmilitaryreligionterrorism', 7190), ('launched', 7191), ('strikes', 7192), ('oncountries', 7193), ('predominantly', 7194), ('childrenenvironmentpublichealth', 7195), ('butts', 7196), ('nicotine', 7197), ('electionsstatebudgetstatefinances', 7198), ('economyincome', 7199), ('adjust', 7200), ('federalbudgetpovertystateswelfare', 7201), ('junk', 7202), ('extent', 7203), ('abortioncongresshealthcare', 7204), ('crimehistoryimmigration', 7205), ('alcoholeducation', 7206), ('friday', 7207), ('criminaljusticeelectionsjobaccomplishments', 7208), ('citygovernmentjobaccomplishments', 7209), ('redskins', 7210), ('citybudgetcitygovernmentcountybudgetcountygovernmenttaxes', 7211), ('7800', 7212), ('crimeforeignpolicyguns', 7213), ('recorded', 7214), ('466', 7215), ('bipartisanshipstatebudget', 7216), ('taxestourism', 7217), ('5anight', 7218), ('hotelmotel', 7219), ('convention', 7220), ('publichealthmarketregulationstates', 7221), ('smoking', 7222), ('acute', 7223), ('myocardial', 7224), ('infarction', 7225), ('stroke', 7226), ('angina', 7227), ('implementation', 7228), ('criminaljusticehumanrightswomen', 7229), ('crimecriminaljusticepublicsafety', 7230), ('ciancis', 7231), ('congressionalrulescriminaljusticewomen', 7232), ('candidatesbiographydiversityimmigration', 7233), ('mexicans', 7234), ('federalbudgetpoverty', 7235), ('candidatesbiographycivilrightshousinglegalissues', 7236), ('breaking', 7237), ('correctionsandupdatesfoodsafetygovernmentregulation', 7238), ('crops', 7239), ('seed', 7240), ('floridaforeignpolicy', 7241), ('cuban', 7242), ('travel', 7243), ('economyenergyhistory', 7244), ('foreignpolicytrade', 7245), ('healthcareimmigrationpolls', 7246), ('educationtaxes', 7247), ('debatesforeignpolicy', 7248), ('apology', 7249), ('tour', 7250), ('climatechangeenergytransportation', 7251), ('caltech', 7252), ('civilrightselections', 7253), ('conversation', 7254), ('candidatesbiographyethicstransportation', 7255), ('wentworth', 7256), ('bending', 7257), ('21174396', 7258), ('economyjobspovertyworkers', 7259), ('correctionsandupdatesfloridataxestransportation', 7260), ('photos', 7261), ('prove', 7262), ('ridership', 7263), ('immigrationmessagemachine2012', 7264), ('childrenfamiliesgovernmentregulationhealthcare', 7265), ('4154', 7266), ('energyforeignpolicytrade', 7267), ('lent', 7268), ('explore', 7269), ('electionslegalissuesstatessupremecourt', 7270), ('energyfederalbudget', 7271), ('royalties', 7272), ('foroil', 7273), ('citygovernmentdebatesenvironmentgovernmentefficiency', 7274), ('garbage', 7275), ('candidatesbiographycapandtradeflorida', 7276), ('deficittaxes', 7277), ('1982', 7278), ('1984', 7279), ('1987', 7280), ('economygovernmentefficiencygovernmentregulationmarketregulationsmallbusiness', 7281), ('pitch', 7282), ('campaignfinanceethics', 7283), ('bushadministrationhousinghungerpovertywelfare', 7284), ('roof', 7285), ('animalscriminaljusticegovernmentregulation', 7286), ('11yearold', 7287), ('injured', 7288), ('woodpecker', 7289), ('cage', 7290), ('bird', 7291), ('candidatesbiographyhealthcarepundits', 7292), ('bankrupting', 7293), ('congresscorrectionsandupdatesincomemessagemachine2014wealth', 7294), ('mcconnellwhat', 7295), ('multimillionaire', 7296), ('economyreligionstimulus', 7297), ('antichristian', 7298), ('sundays', 7299), ('boy', 7300), ('campaignfinancelegalissuescampaignadvertising', 7301), ('housingretirement', 7302), ('legler', 7303), ('installing', 7304), ('sprinklers', 7305), ('generators', 7306), ('deficitfederalbudgettaxes', 7307), ('collects', 7308), ('economywomenworkers', 7309), ('correctionsandupdatesmarketregulation', 7310), ('healthcareabcnewsweek', 7311), ('rationing', 7312), ('civilrightscorporationsethicsfoodsafetypunditstransparency', 7313), ('distort', 7314), ('airwaves', 7315), ('crimeterrorismwomen', 7316), ('boyfriends', 7317), ('deficitfederalbudgetmedicaremilitarysocialsecurity', 7318), ('nonmilitary', 7319), ('campaignfinanceabcnewsweek', 7320), ('foreigncontrolled', 7321), ('floridaamendmentssupremecourt', 7322), ('express', 7323), ('questions', 7324), ('jobslabormessagemachine', 7325), ('hitler', 7326), ('stalin', 7327), ('mubarak', 7328), ('autocrats', 7329), ('healthcarepoverty', 7330), ('expense', 7331), ('jobslaborstates', 7332), ('healthcarenewhampshire2012abcnewsweek', 7333), ('ipab', 7334), ('pundits', 7335), ('mornings', 7336), ('historysupremecourtvotingrecord', 7337), ('unusual', 7338), ('confirmation', 7339), ('1940', 7340), ('economyfederalbudgetmilitary', 7341), ('nondefense', 7342), ('jfk', 7343), ('immigrationpovertysocialsecurityunionswomenworkers', 7344), ('1956', 7345), ('asylum', 7346), ('electionslegalissuestechnology', 7347), ('outlawed', 7348), ('electionstaxes', 7349), ('transparencywomen', 7350), ('historytechnology', 7351), ('economytaxes', 7352), ('cbo', 7353), ('60000014', 7354), ('90000027', 7355), ('citybudgetpublicsafety', 7356), ('lay', 7357), ('energyinfrastructure', 7358), ('est', 7359), ('bi', 7360), ('economyincometaxes', 7361), ('crimecriminaljusticedrugs', 7362), ('bath', 7363), ('salts', 7364), ('raccoons', 7365), ('candidatesbiographycorrectionsandupdateseducation', 7366), ('universityhave', 7367), ('ebolafederalbudgetpublichealth', 7368), ('landuse', 7369), ('foreignpolicyhealthcarepublichealth', 7370), ('cervical', 7371), ('screeningand', 7372), ('vaccines', 7373), ('abortionchildrenhealthcarepublichealthtechnologywomen', 7374), ('ultrasound', 7375), ('odds', 7376), ('foreignpolicyisraelnuclear', 7377), ('correctionsandupdateshealthcarelegalissuestaxes', 7378), ('unilaterally', 7379), ('blog', 7380), ('midlevel', 7381), ('bureaucrat', 7382), ('corporationsdebtdeficiteconomyfinancialregulationgovernmentregulationmessagemachine2012marketregulationstimulusvotingrecord', 7383), ('sheldon', 7384), ('rewarded', 7385), ('executives', 7386), ('congressfloridastates', 7387), ('senatebecause', 7388), ('laborstatebudgetworkers', 7389), ('reporters', 7390), ('bargainedfor', 7391), ('chinaforeignpolicytrade', 7392), ('imbalance', 7393), ('educationabcnewsweek', 7394), ('economyjobspovertywelfare', 7395), ('crimediversitypublicsafety', 7396), ('cop', 7397), ('candidatesbiographydisabilityelectionshistoryprivacytransparency', 7398), ('meets', 7399), ('debatessports', 7400), ('usual', 7401), ('dems', 7402), ('rig', 7403), ('foreignpolicyhistory', 7404), ('1968', 7405), ('patrice', 7406), ('lumumba', 7407), ('moscow', 7408), ('mahmoud', 7409), ('abbas', 7410), ('ali', 7411), ('khamenei', 7412), ('established', 7413), ('relationships', 7414), ('economyenergytrade', 7415), ('foreignpolicynewhampshire2012', 7416), ('kim', 7417), ('jongll', 7418), ('ahmadinejad', 7419), ('chavez', 7420), ('actors', 7421), ('citybudgetcitygovernmenthealthcarepublichealthwomen', 7422), ('campaignfinancetransparency', 7423), ('suspicion', 7424), ('abortionstatebudgetstatefinances', 7425), ('climatechangeenergyjobs', 7426), ('employ', 7427), ('foreignpolicypolls', 7428), ('corporationsgovernmentefficiencylegalissuesmilitarypunditsabcnewsweek', 7429), ('halliburton', 7430), ('defrauded', 7431), ('candidatesbiographyhistoryincomecampaignadvertisingtaxeswealth', 7432), ('alcoholanimalschildrencrime', 7433), ('untagged', 7434), ('correctionsandupdateshealthcare', 7435), ('shrink', 7436), ('correctionsandupdateseconomyfederalbudgethistorystimulus', 7437), ('accelerated', 7438), ('pace', 7439), ('precedent', 7440), ('candidatesbiographydrugshomelandsecurityimmigration', 7441), ('addiction', 7442), ('opioid', 7443), ('publicsafetytransportation', 7444), ('delta', 7445), ('animal', 7446), ('carriers', 7447), ('agricultureeconomytaxes', 7448), ('farms', 7449), ('inheritance', 7450), ('censuscivilrightscriminaljusticeoccupywallstreeturban', 7451), ('incarcerates', 7452), ('incarcerated', 7453), ('incomewomen', 7454), ('11000', 7455), ('vagina', 7456), ('wealthworkers', 7457), ('educationfoodsafetyabcnewsweek', 7458), ('freerange', 7459), ('nutritious', 7460), ('offerings', 7461), ('bipartisanshiphealthcarepublichealth', 7462), ('abortionhealthcarelegalissueswomen', 7463), ('vitro', 7464), ('fertilization', 7465), ('legalissuesmessagemachinepublicsafety', 7466), ('immigrationtaxes', 7467), ('medicaidstatebudget', 7468), ('capandtradeclimatechangeeconomygasprices', 7469), ('smallbusiness', 7470), ('carol', 7471), ('sheaporter', 7472), ('candidatesbiographymessagemachine2012newhampshire2012sports', 7473), ('energyenvironmentmilitary', 7474), ('gabrielle', 7475), ('giffords', 7476), ('emphasis', 7477), ('damaging', 7478), ('stabbing', 7479), ('clubbing', 7480), ('enemy', 7481), ('minimize', 7482), ('messagemachinemarketregulationsocialsecurity', 7483), ('deficitfederalbudgethealthcarestimulustaxes', 7484), ('taxcut', 7485), ('adds', 7486), ('childrenhistory', 7487), ('federalbudgethealthcaremedicaidmedicare', 7488), ('cliff', 7489), ('disproportionately', 7490), ('impacting', 7491), ('renal', 7492), ('crimelabor', 7493), ('islamterrorism', 7494), ('incl', 7495), ('antiabortion', 7496), ('antigov', 7497), ('diversityeconomyjobs', 7498), ('federalbudgethealthcaremedicare', 7499), ('governmentregulationgunsterrorism', 7500), ('citybudgetcitygovernmentincomejobspoverty', 7501), ('suburbs', 7502), ('economypunditsstimulus', 7503), ('fourthquarter', 7504), ('firsttime', 7505), ('buyer', 7506), ('electionspublicservice', 7507), ('economygovernmentregulationincomelaborpovertysmallbusinesswealthworkers', 7508), ('correctionsandupdateshealthcarepublichealthwomen', 7509), ('cutoff', 7510), ('130', 7511), ('economyfederalbudgettransportation', 7512), ('educationislam', 7513), ('grades', 7514), ('studying', 7515), ('congresscongressionalrulesenergyvotingrecord', 7516), ('citygovernmentcountygovernmenthealthcarepensionsstatebudgetstatefinancestaxes', 7517), ('environmenttransportation', 7518), ('correctionsandupdateshealthcarepollspundits', 7519), ('corporationsmessagemachinestatebudgettaxesworkers', 7520), ('immigrationnewhampshire2012', 7521), ('visited', 7522), ('pronounced', 7523), ('socialsecurityveterans', 7524), ('candidatesbiographyethics', 7525), ('catherine', 7526), ('cortez', 7527), ('masto', 7528), ('sweetheart', 7529), ('firm', 7530), ('economystimulustransportation', 7531), ('tout', 7532), ('candidatesbiographycongressdrugshealthcarehistorymedicarevotingrecord', 7533), ('poliquin', 7534), ('disabilitygovernmentefficiencylaborworkers', 7535), ('outage', 7536), ('criminaljusticepublicsafety', 7537), ('sharpton', 7538), ('declaring', 7539), ('posed', 7540), ('correctionsandupdatesdebtfederalbudgetmessagemachine2012', 7541), ('educationlegalissuespopulationstates', 7542), ('candidatesbiographygovernmentefficiencytrade', 7543), ('suites', 7544), ('racks', 7545), ('minibar', 7546), ('messagemachinestimulustrade', 7547), ('citybudgeteducationflorida', 7548), ('performance', 7549), ('lowperforming', 7550), ('palm', 7551), ('economyjobslaborpovertystatesworkers', 7552), ('candidatesbiographychildrencrimecriminaljusticefamiliespublicsafetywomen', 7553), ('candidatesbiographycrimecriminaljusticegunsjobaccomplishments', 7554), ('economyfederalbudgetgovernmentefficiencyhealthcaremedicaidpovertypublichealthmarketregulationstatebudgetstatefinancesstatestaxes', 7555), ('enrollee', 7556), ('healthcarepublichealth', 7557), ('hani', 7558), ('baragwanath', 7559), ('thirdbiggest', 7560), ('educationgovernmentregulationhumanrightslegalissuesprivacytechnologyunionsworkers', 7561), ('surveillance', 7562), ('cubicle', 7563), ('educationtourism', 7564), ('shave', 7565), ('jobsstatebudget', 7566), ('incentive', 7567), ('laboratories', 7568), ('bankruptcycandidatesbiographyhistoryincomecampaignadvertisingtaxeswealth', 7569), ('candidatesbiographycongresselections', 7570), ('staffer', 7571), ('thencongressman', 7572), ('diversityelectionshistoryimmigrationlegalissues', 7573), ('intrinsically', 7574), ('congresscrimegovernmentregulationgunshomelandsecuritypublicsafetyterrorism', 7575), ('244', 7576), ('223', 7577), ('abortionfederalbudget', 7578), ('cruzs', 7579), ('economylaborstatesunionsworkers', 7580), ('constantly', 7581), ('deficits', 7582), ('climates', 7583), ('economyelections', 7584), ('congressforeignpolicy', 7585), ('deficitfederalbudgethistoryjobaccomplishmentstaxes', 7586), ('economypoverty', 7587), ('unacceptable', 7588), ('panels', 7589), ('photovoltaic', 7590), ('forget', 7591), ('educationhumanrights', 7592), ('angry', 7593), ('resignation', 7594), ('ranking', 7595), ('childrenforeignpolicygovernmentregulationimmigration', 7596), ('texasmexico', 7597), ('economyforeignpolicyisrael', 7598), ('israels', 7599), ('threefourths', 7600), ('federalbudgetmedicarepublichealth', 7601), ('criminaljusticegunsvotingrecord', 7602), ('loughlin', 7603), ('bushadministrationeconomyjobsstimulus', 7604), ('criminaljusticetransparency', 7605), ('broke', 7606), ('18yearold', 7607), ('suspect', 7608), ('robbery', 7609), ('msnbc', 7610), ('practically', 7611), ('behindcloseddoors', 7612), ('revelation', 7613), ('agriculturetrade', 7614), ('300131', 7615), ('countryoforigin', 7616), ('labeling', 7617), ('chicken', 7618), ('pork', 7619), ('correctionsandupdatestransparency', 7620), ('2054th', 7621), ('9999', 7622), ('crimecriminaljusticeeducationstatebudgetstatefinancesstates', 7623), ('corporationseconomyfederalbudgetfinancialregulationhousingtransportation', 7624), ('homelandsecuritymilitary', 7625), ('rounds', 7626), ('caliber', 7627), ('ammunition', 7628), ('unrest', 7629), ('invasion', 7630), ('foreignpolicysports', 7631), ('cup', 7632), ('final', 7633), ('corporationseconomymarketregulation', 7634), ('chunk', 7635), ('soft', 7636), ('civilrightsgaysandlesbianslegalissuesreligionsupremecourt', 7637), ('bushadministrationcongressfakenewsgovernmentefficiencyhistoryjobaccomplishmentsvotingrecord', 7638), ('floridagaysandlesbianshumanrightsstatesworkers', 7639), ('survived', 7640), ('orlando', 7641), ('fl', 7642), ('tomorrow', 7643), ('smallbusinesstaxes', 7644), ('civilrightsdiversityfamilies', 7645), ('saysbarack', 7646), ('ofindianas', 7647), ('gamblingjobsstatebudgettaxes', 7648), ('kenosha', 7649), ('tribe', 7650), ('economygovernmentefficiencystimulus', 7651), ('energygaspricestransportation', 7652), ('energyenvironmentpublichealthstates', 7653), ('ozone', 7654), ('ninetyfive', 7655), ('healthcarereligion', 7656), ('scientologists', 7657), ('consumersafetygovernmentefficiency', 7658), ('designers', 7659), ('childrendrugseconomyfamiliesfloridagovernmentregulationhungermarijuanapovertystatebudgetstatefinances', 7660), ('baseballrecreationsports', 7661), ('thirty', 7662), ('triple', 7663), ('ballparks', 7664), ('correctionsandupdateshealthcaremedicaidstatebudget', 7665), ('expansive', 7666), ('relatively', 7667), ('federalbudgetinfrastructuretransportation', 7668), ('passenger', 7669), ('trains', 7670), ('congressjobaccomplishments', 7671), ('candidatesbiographyjobaccomplishmentsstateswomen', 7672), ('federalbudgettransparency', 7673), ('requesting', 7674), ('countygovernmentjobsrecreationsports', 7675), ('miamis', 7676), ('207000', 7677), ('cargo', 7678), ('cruise', 7679), ('debteconomymessagemachine2012', 7680), ('economyjobaccomplishmentsstatefinancestaxesworkers', 7681), ('agriculturecandidatesbiography', 7682), ('maddox', 7683), ('correctionsandupdatescrimecriminaljusticeforeignpolicyhomelandsecurityimmigration', 7684), ('ukrainians', 7685), ('ranch', 7686), ('afghanistancongressmilitary', 7687), ('rare', 7688), ('determination', 7689), ('correctionsandupdatesdebatesforeignpolicyterrorism', 7690), ('waited', 7691), ('educationgovernmentefficiencyjobaccomplishmentsjobs', 7692), ('steady', 7693), ('congresscongressionalruleshistorypunditssupremecourt', 7694), ('climatechangecorrectionsandupdatespollsscience', 7695), ('causingclimate', 7696), ('debunked', 7697), ('intergovernmental', 7698), ('thin', 7699), ('candidatesbiographycrimeiraqmilitary', 7700), ('acts', 7701), ('merited', 7702), ('countybudgettaxes', 7703), ('gwinnett', 7704), ('cutbacks', 7705), ('staffing', 7706), ('candidatesbiographydebtdeficiteconomyhistoryjobaccomplishmentsstatebudgetstatefinancesstates', 7707), ('correctionsandupdatesforeignpolicymilitaryterrorism', 7708), ('priests', 7709), ('beheaded', 7710), ('rebels', 7711), ('rebel', 7712), ('eating', 7713), ('soldier', 7714), ('medicaremessagemachine2014retirementsocialsecurity', 7715), ('overhaul', 7716), ('69', 7717), ('economyeducationabcnewsweek', 7718), ('educations', 7719), ('candidatesbiographypopculture', 7720), ('nbc', 7721), ('governmentefficiencyjobsstatebudget', 7722), ('taxation', 7723), ('ethicslegalissues', 7724), ('paula', 7725), ('850000', 7726), ('criminaljusticehumanrights', 7727), ('arent', 7728), ('federalbudgetimmigrationlegalissues', 7729), ('dea', 7730), ('atf', 7731), ('climatechangeenvironmentstatebudgetworkers', 7732), ('dep', 7733), ('2812', 7734), ('mid80s', 7735), ('candidatesbiographycriminaljusticeeducationenvironmentlegalissues', 7736), ('campaignfinanceeducation', 7737), ('laborstatebudgetstatefinances', 7738), ('literally', 7739), ('congressfederalbudgethealthcaretaxes', 7740), ('keeps', 7741), ('abortioncrimecriminaljusticewomen', 7742), ('abortionhealthcarepublichealthstatebudgetstatefinances', 7743), ('selling', 7744), ('aborted', 7745), ('candidatesbiographyclimatechangeenergy', 7746), ('citygovernmentcrimecriminaljusticefamilieshumanrightsurban', 7747), ('feral', 7748), ('candidatesbiographycitygovernmentelections', 7749), ('donovan', 7750), ('laborstatebudgetunions', 7751), ('agricultureenergyenvironmentscience', 7752), ('cornell', 7753), ('concludes', 7754), ('generates', 7755), ('militaryterrorism', 7756), ('maj', 7757), ('nidal', 7758), ('hasan', 7759), ('advisor', 7760), ('healthcareimmigrationlegalissues', 7761), ('economyeducationenergyfederalbudgetincomeinfrastructurejobstechnology', 7762), ('goodpaying', 7763), ('economyfederalbudgethistory', 7764), ('kennedy', 7765), ('consumes', 7766), ('taxestechnology', 7767), ('fccs', 7768), ('whatis', 7769), ('abortionscience', 7770), ('alcoholmarketregulationstates', 7771), ('growlers', 7772), ('customers', 7773), ('punditsstatebudgetabcnewsweekworkers', 7774), ('bid', 7775), ('worry', 7776), ('deficitmessagemachine2012', 7777), ('legalissuesmarketregulationtechnology', 7778), ('washingtons', 7779), ('extends', 7780), ('bulb', 7781), ('debtfederalbudgethistorysocialsecuritytaxeswealth', 7782), ('assessed', 7783), ('captures', 7784), ('criminaljusticestatefinances', 7785), ('caseload', 7786), ('energytaxes', 7787), ('historysciencespace', 7788), ('mirror', 7789), ('habitability', 7790), ('censustransportation', 7791), ('amtraks', 7792), ('corridor', 7793), ('i95', 7794), ('accommodate', 7795), ('ethicsmessagemachine2012taxes', 7796), ('fictional', 7797), ('avoidance', 7798), ('schemes', 7799), ('correctionsandupdatesfederalbudgethistorymilitary', 7800), ('happened', 7801), ('hollowed', 7802), ('acrosstheboard', 7803), ('capandtradeclimatechange', 7804), ('1761', 7805), ('candidatesbiographyelectionspopculture', 7806), ('economygovernmentefficiencyjobspunditsworkers', 7807), ('incomelaborworkers', 7808), ('childrenfloridahealthcare', 7809), ('inexpensive', 7810), ('childonly', 7811), ('censusmilitaryveterans', 7812), ('floridastatebudget', 7813), ('context', 7814), ('candidatesbiographyeducationobamabirthcertificate', 7815), ('candidatesbiographypolls', 7816), ('usborn', 7817), ('diversityeconomyincomepoverty', 7818), ('economymarriagepopculturestatebudgetstatefinances', 7819), ('series', 7820), ('spouses', 7821), ('glorifying', 7822), ('congressgunscampaignadvertisingsciencesexuality', 7823), ('idaho', 7824), ('censuseconomytaxes', 7825), ('190000', 7826), ('economyforeignpolicytrade', 7827), ('ethiopia', 7828), ('exports', 7829), ('abortioncorrectionsandupdatesdebateswomen', 7830), ('forcible', 7831), ('citybudgetcountybudgeteducationpensionsstatebudget', 7832), ('saysproposal', 7833), ('boostteacherpension', 7834), ('fundputs', 7835), ('candidatesbiographyhealthcaremessagemachine2012votingrecord', 7836), ('debthistory', 7837), ('healthcaremessagemachinevotingrecord', 7838), ('anyway', 7839), ('climatechangeenvironmentpundits', 7840), ('postsoviet', 7841), ('industrial', 7842), ('crimesports', 7843), ('censuscrimelegalissuespundits', 7844), ('correctionsandupdatesforeignpolicynuclearstates', 7845), ('ransom', 7846), ('hostages', 7847), ('diversityeducation', 7848), ('citybudgettransportation', 7849), ('analysis', 7850), ('youthpass', 7851), ('agencys', 7852), ('bipartisanshipbushadministrationcongresseconomyforeignpolicyhistoryjobslabortradeworkers', 7853), ('energyforeignpolicynewhampshire2012', 7854), ('obamabirthcertificate', 7855), ('startedthe', 7856), ('birther', 7857), ('controversy', 7858), ('diversityeconomypunditsabcnewsweek', 7859), ('disappeared', 7860), ('abortionlegalissueswomen', 7861), ('druginduced', 7862), ('followup', 7863), ('consumersafetycorrectionsandupdatesgovernmentregulationmarketregulationstatessupremecourtworkers', 7864), ('campaignfinancegambling', 7865), ('economyforeignpolicyjobstrade', 7866), ('surged', 7867), ('agriculturechildrenlabor', 7868), ('tools', 7869), ('screwdriver', 7870), ('milking', 7871), ('machine', 7872), ('wheelbarrow', 7873), ('childrendrugspoverty', 7874), ('drugspublichealthveterans', 7875), ('memo', 7876), ('outlined', 7877), ('foreignpolicyhistoryterrorism', 7878), ('flew', 7879), ('managua', 7880), ('dictator', 7881), ('ortega', 7882), ('engaging', 7883), ('citygovernmentsportstaxes', 7884), ('candidatesbiographyeconomylabor', 7885), ('lobbied', 7886), ('bipartisanshipelectionsjobaccomplishmentsjobslaborstates', 7887), ('bluest', 7888), ('economyjobaccomplishments', 7889), ('128000', 7890), ('federalbudgethealthcaremedicaretaxes', 7891), ('childreneducationfloridamessagemachine', 7892), ('candidatesbiographycriminaljustice', 7893), ('correctionsandupdatesstatebudgetstatefinancestaxes', 7894), ('deficitfederalbudgetstimulus', 7895), ('posted', 7896), ('messagemachine2012stimulus', 7897), ('upgrades', 7898), ('animalslegalissues', 7899), ('braley', 7900), ('neighbor', 7901), ('chickens', 7902), ('childrencrimecriminaljusticetechnology', 7903), ('teenagers', 7904), ('agricultureenergy', 7905), ('poultry', 7906), ('criminaljusticeurban', 7907), ('tore', 7908), ('abandoned', 7909), ('havens', 7910), ('environmentfederalbudget', 7911), ('acquire', 7912), ('educationfederalbudgetfinancialregulation', 7913), ('22000the', 7914), ('pickup', 7915), ('candidatesbiographycrimeeconomymessagemachine', 7916), ('fathers', 7917), ('alexi', 7918), ('mobsters', 7919), ('abortioncandidatesbiographyeducation', 7920), ('unequivocal', 7921), ('moderate', 7922), ('campaignadvertising', 7923), ('saysvirginia', 7924), ('republicanscott', 7925), ('taylor', 7926), ('appear', 7927), ('warrant', 7928), ('chinadeficitmilitary', 7929), ('739', 7930), ('economystatefinancestaxes', 7931), ('jobslegalissuesmessagemachine2012womenworkers', 7932), ('candidatesbiographycriminaljusticeelectionslegalissuescampaignadvertising', 7933), ('jobaccomplishmentstrade', 7934), ('incomejobstaxes', 7935), ('candidatesbiographyeconomyfinancialregulationhousing', 7936), ('bubble', 7937), ('childreneducationpublichealthwomen', 7938), ('schooling', 7939), ('energyenvironmentfederalbudgetoilspill', 7940), ('not', 7941), ('consumersafety', 7942), ('candidatesbiographycivilrightsreligion', 7943), ('winds', 7944), ('ugly', 7945), ('criminaljusticeelectionsethicshistorylegalissuestechnologytransparency', 7946), ('675000', 7947), ('congresshealthcare', 7948), ('islampollsreligionterrorism', 7949), ('jihad', 7950), ('civilrightslegalissuessexualityworkers', 7951), ('foreignpolicyimmigrationterrorism', 7952), ('iraqi', 7953), ('islamreligion', 7954), ('canceled', 7955), ('ceremony', 7956), ('ruse', 7957), ('offend', 7958), ('anyonebut', 7959), ('september', 7960), ('pm', 7961), ('religion', 7962), ('hill', 7963), ('beside', 7964), ('economysmallbusinesstaxes', 7965), ('expire', 7966), ('economypollspopculturetechnology', 7967), ('healthcarehistory', 7968), ('civilrightshumanrightslegalissuespunditsreligionsports', 7969), ('aclu', 7970), ('tebow', 7971), ('praying', 7972), ('sidelines', 7973), ('corporationsdebtdeficiteconomysmallbusinessstatebudgetstatefinancestaxes', 7974), ('taxcredit', 7975), ('cvs', 7976), ('correctionsandupdatestaxes', 7977), ('thirdhighest', 7978), ('correctionsandupdatestransportation', 7979), ('proposes', 7980), ('congested', 7981), ('roadways', 7982), ('corporationsfederalbudgettaxes', 7983), ('jobaccomplishmentsjobsworkers', 7984), ('releasing', 7985), ('dreamed', 7986), ('bipartisanshipcapandtradeclimatechangeenvironmentmarketregulationscience', 7987), ('61', 7988), ('nontea', 7989), ('agreethere', 7990), ('partiers', 7991), ('contrarily', 7992), ('messagemachinesocialsecurity', 7993), ('blames', 7994), ('mcintyre', 7995), ('costofliving', 7996), ('debatesmedicareretirement', 7997), ('corporationseconomyelectionsjobaccomplishmentsjobssmallbusiness', 7998), ('crimedrugshomelandsecurityhumanrightsimmigrationmarijuana', 7999), ('transporting', 8000), ('admission', 8001), ('congressfederalbudgetforeignpolicy', 8002), ('contributed', 8003), ('tragedies', 8004), ('economyforeignpolicypundits', 8005), ('suffering', 8006), ('congressenvironment', 8007), ('faso', 8008), ('fossil', 8009), ('fracked', 8010), ('educationstatefinances', 8011), ('adjusted', 8012), ('floridaamendmentsmarijuana', 8013), ('walgreens', 8014), ('abortionfamilieshealthcarepublichealthsexualitystatebudgetwomen', 8015), ('defunding', 8016), ('stranded', 8017), ('economystatefinances', 8018), ('federalbudgetfinancialregulationnewhampshire2012', 8019), ('advantage', 8020), ('federalbudgetlegalissuestaxes', 8021), ('reconciliation', 8022), ('for', 8023), ('candidatesbiographyhistoryjobaccomplishmentslegalissuessupremecourt', 8024), ('deficitveterans', 8025), ('diversityhistoryincomepoverty', 8026), ('selma', 8027), ('ala', 8028), ('healthcaremessagemachinesmallbusiness', 8029), ('crushes', 8030), ('agriculturejobaccomplishmentsmessagemachine', 8031), ('bradys', 8032), ('priority', 8033), ('masseuthanize', 8034), ('sheltered', 8035), ('environmentfoodsafety', 8036), ('imposing', 8037), ('religionscience', 8038), ('denies', 8039), ('evolution', 8040), ('baseballhealthcare', 8041), ('antitrust', 8042), ('economyelectionsjobaccomplishmentsworkers', 8043), ('november', 8044), ('20072008', 8045), ('afghanistaniraqlegalissuesvotingrecord', 8046), ('specifying', 8047), ('engagement', 8048), ('candidatesbiographyelectionsfederalbudgethealthcareimmigrationmessagemachinepoverty', 8049), ('pinnacle', 8050), ('overlook', 8051), ('bushadministration', 8052), ('corporationseconomyfinancialregulationincomeworkers', 8053), ('431', 8054), ('deficitstatebudgetstatefinancestaxes', 8055), ('443', 8056), ('candidatesbiographyredistricting', 8057), ('antoniorooted', 8058), ('seeks', 8059), ('bipartisanshipstates', 8060), ('accomplishments', 8061), ('climatechangeenvironmenttransportation', 8062), ('tailpipe', 8063), ('onehundredths', 8064), ('fahrenheit', 8065), ('2050', 8066), ('agriculturefederalbudget', 8067), ('farmers', 8068), ('punditstransportation', 8069), ('junkyard', 8070), ('tow', 8071), ('4500', 8072), ('electionsgovernmentefficiencylaborunions', 8073), ('binding', 8074), ('arbitration', 8075), ('zanesville', 8076), ('settlement', 8077), ('corporationseconomyfamiliesincome', 8078), ('wellpaid', 8079), ('civilrightspublicservice', 8080), ('appointments', 8081), ('debatestaxes', 8082), ('healthcarehistorymedicare', 8083), ('realized', 8084), ('statebudgetstatefinancestransportation', 8085), ('childreneducationenergystatebudget', 8086), ('45193289', 8087), ('leases', 8088), ('watt', 8089), ('economyfederalbudgetstatebudget', 8090), ('candidatesbiographyimmigration', 8091), ('spanish', 8092), ('economyhealthcarepunditstaxes', 8093), ('childrenhumanrightsimmigration', 8094), ('noncontiguous', 8095), ('healthcaremarijuanastates', 8096), ('cannabis', 8097), ('campaignfinancecandidatesbiographyelections', 8098), ('profiting', 8099), ('candidatesbiographyinfrastructurejobaccomplishments', 8100), ('refinanced', 8101), ('2326', 8102), ('populationstates', 8103), ('vietnamese', 8104), ('commonly', 8105), ('environmentgamblingjobsstatefinances', 8106), ('ensures', 8107), ('preservation', 8108), ('176000', 8109), ('acres', 8110), ('federalbudgethealthcarevotingrecord', 8111), ('criminaljusticeforeignpolicylegalissues', 8112), ('tear', 8113), ('historylegalissuesprivacy', 8114), ('fourth', 8115), ('revolution', 8116), ('spark', 8117), ('infrastructuremessagemachine2012', 8118), ('economypunditsabcnewsweekworkers', 8119), ('fortyfive', 8120), ('beat', 8121), ('surge', 8122), ('disabilitylaborstatesworkers', 8123), ('candidatesbiographycrimehistory', 8124), ('father', 8125), ('photographedwith', 8126), ('harvey', 8127), ('oswald', 8128), ('criminaljusticewomen', 8129), ('shield', 8130), ('healthcarepundits', 8131), ('civilrightseducationelectionsjobaccomplishmentscampaignadvertising', 8132), ('harder', 8133), ('abortioncongressvotingrecord', 8134), ('correctionsandupdatesgunshomelandsecurity', 8135), ('charleston', 8136), ('sc', 8137), ('shootershould', 8138), ('punditssotomayornominationsupremecourt', 8139), ('firefighter', 8140), ('ricci', 8141), ('preference', 8142), ('skin', 8143), ('totally', 8144), ('disregarded', 8145), ('electionsgunsstates', 8146), ('bloombergstyle', 8147), ('criminaljustice', 8148), ('immigrationsocialsecuritytaxes', 8149), ('crimedrugsmarijuana', 8150), ('potshoot', 8151), ('stab', 8152), ('strangle', 8153), ('agricultureanimalsfoodsafetyhealthcare', 8154), ('spares', 8155), ('civilrightsfamiliesgaysandlesbiansgovernmentregulationhumanrightslegalissuesmarriagesexuality', 8156), ('economyimmigration', 8157), ('proclaimed', 8158), ('federalbudgethealthcaremedicaremessagemachine2012', 8159), ('educationmilitary', 8160), ('healthcarestatebudgetstatefinancesstates', 8161), ('educationgunsstatebudget', 8162), ('ogden', 8163), ('hurting', 8164), ('crafted', 8165), ('robin', 8166), ('skyrocket', 8167), ('freedoms', 8168), ('electionspublichealthtaxes', 8169), ('diverts', 8170), ('crimegovernmentregulationgunspublicsafetymarketregulationterrorism', 8171), ('operatives', 8172), ('campaignfinancecorporationsmedicareoilspillsocialsecurity', 8173), ('ruins', 8174), ('reign', 8175), ('abortiondeficitfederalbudget', 8176), ('energyenvironmentgovernmentefficiency', 8177), ('acre', 8178), ('mac', 8179), ('economyhealthcarepundits', 8180), ('rushing', 8181), ('sat', 8182), ('desk', 8183), ('candidatesbiographygaysandlesbianshistorycampaignadvertisingpollspunditsabcnewsweekwomen', 8184), ('characteristics', 8185), ('socialist', 8186), ('abortioncampaignfinance', 8187), ('nocost', 8188), ('donations', 8189), ('crimegunslegalissuespublicsafety', 8190), ('righttocarry', 8191), ('predators', 8192), ('historypolls', 8193), ('carter', 8194), ('healthcarehistorymessagemachine2012', 8195), ('embraced', 8196), ('component', 8197), ('48700', 8198), ('128300', 8199), ('diversityfamiliesgaysandlesbianslegalissuesmarriagereligion', 8200), ('knights', 8201), ('columbus', 8202), ('ceremonies', 8203), ('childreneducationlegalissuessexuality', 8204), ('decree', 8205), ('boys', 8206), ('correctionsandupdateseducation', 8207), ('animalsmilitary', 8208), ('saves', 8209), ('servicemen', 8210), ('agriculturejobsstateswater', 8211), ('dairy', 8212), ('familiessportswomenworkers', 8213), ('economyincomelabor', 8214), ('economically', 8215), ('labormessagemachine2012', 8216), ('relations', 8217), ('boeing', 8218), ('factory', 8219), ('healthcaremedicaid', 8220), ('incomejobspovertywomenworkers', 8221), ('diversityreligion', 8222), ('japan', 8223), ('propagation', 8224), ('import', 8225), ('koran', 8226), ('arabic', 8227), ('homelandsecurityreligion', 8228), ('healthcaresmallbusinesstaxes', 8229), ('republicancontrolled', 8230), ('140', 8231), ('federalbudgetstimulustransportation', 8232), ('advertise', 8233), ('economyjobslaborabcnewsweek', 8234), ('recessions', 8235), ('federalbudgethealthcaremedicaidpublichealthstatebudget', 8236), ('accepting', 8237), ('citybudgetcitygovernmentunionsworkers', 8238), ('advocacy', 8239), ('contracting', 8240), ('altered', 8241), ('brooks', 8242), ('contracted', 8243), ('verifies', 8244), ('authorized', 8245), ('inaccurate', 8246), ('excess', 8247), ('housingincomemessagemachine2014povertytaxes', 8248), ('140000', 8249), ('bipartisanshipcampaignfinancecandidatesbiographyelections', 8250), ('bankruptcyeconomygamblingstatebudgetstatefinancesstates', 8251), ('casinos', 8252), ('drugshealthcaremarijuanastates', 8253), ('lenient', 8254), ('medicalmarijuana', 8255), ('limitless', 8256), ('specified', 8257), ('gunsworkers', 8258), ('workplaces', 8259), ('prohibit', 8260), ('childrenconsumersafetygovernmentregulationlegalissuespublichealth', 8261), ('vapor', 8262), ('populationpunditsstates', 8263), ('440000', 8264), ('messagemachine2012statebudget', 8265), ('ronda', 8266), ('storms', 8267), ('ethicsflorida', 8268), ('bahamas', 8269), ('economyeducationjobsstimulus', 8270), ('messagemachinereligionwomen', 8271), ('federalbudgetmilitarynewhampshire2012', 8272), ('economylaborworkers', 8273), ('discovered', 8274), ('23rd', 8275), ('ebolahealthcaremilitarypublichealthpublicsafety', 8276), ('economyjobaccomplishmentsjobs', 8277), ('prerecession', 8278), ('peak', 8279), ('trailed', 8280), ('correctionsandupdatesvotingrecord', 8281), ('nolan', 8282), ('partyline', 8283), ('harmony', 8284), ('liberties', 8285), ('homelandsecuritytransportation', 8286), ('patdown', 8287), ('initiated', 8288), ('homelandsecurityterrorismtransportation', 8289), ('airline', 8290), ('gaysandlesbianslegalissues', 8291), ('alabamians', 8292), ('legalissuessupremecourtcolbertreport', 8293), ('foreignpolicyislamreligionterrorism', 8294), ('organizations', 8295), ('emirates', 8296), ('abortiontaxes', 8297), ('20112012', 8298), ('4191', 8299), ('via', 8300), ('childrendiversityeducation', 8301), ('crimegunspundits', 8302), ('gunrelated', 8303), ('belgiums', 8304), ('bipartisanship', 8305), ('rush', 8306), ('limbaugh', 8307), ('fail', 8308), ('succeed', 8309), ('criminaljusticediversity', 8310), ('photographed', 8311), ('fear', 8312), ('sons', 8313), ('robs', 8314), ('educationimmigration', 8315), ('miamidade', 8316), ('educating', 8317), ('animalspublicsafetytechnology', 8318), ('peta', 8319), ('fancy', 8320), ('crimeelections', 8321), ('disenfranchised', 8322), ('onequarter', 8323), ('floridians', 8324), ('debteconomy', 8325), ('candidatesbiographyhistoryhomelandsecurityimmigrationvotingrecord', 8326), ('educationfederalbudget', 8327), ('abolish', 8328), ('energyenvironmentjobs', 8329), ('obamasown', 8330), ('assessments', 8331), ('correctionsandupdateseconomypoverty', 8332), ('itd', 8333), ('15th', 8334), ('statebudgetstatefinancesworkers', 8335), ('fulltime', 8336), ('candidatesbiographyeducationstates', 8337), ('establishing', 8338), ('economygambling', 8339), ('correctionsandupdateseconomyjobsstates', 8340), ('bipartisanshipcorrectionsandupdatesforeignpolicy', 8341), ('governmental', 8342), ('abortiongovernmentregulationhealthcaresupremecourtwomen', 8343), ('campaignfinancecorporations', 8344), ('454260', 8345), ('legalissuesstatebudget', 8346), ('plumbers', 8347), ('engineers', 8348), ('healthcareveterans', 8349), ('environmentmessagemachine2012', 8350), ('septic', 8351), ('tank', 8352), ('inspection', 8353), ('crimegovernmentregulationgunspublicsafety', 8354), ('australia', 8355), ('environmenttaxes', 8356), ('5cent', 8357), ('paper', 8358), ('bags', 8359), ('electionsgaysandlesbiansstates', 8360), ('animalssciencetourism', 8361), ('whales', 8362), ('seaworld', 8363), ('crimecriminaljusticefoodsafety', 8364), ('oyster', 8365), ('federalbudgethealthcarespace', 8366), ('lamar', 8367), ('historypundits', 8368), ('litmus', 8369), ('childrenhealthcarestatebudget', 8370), ('ending', 8371), ('immunizations', 8372), ('113000', 8373), ('congresshealthcarepolls', 8374), ('economyjobs', 8375), ('turnaround', 8376), ('debatesdeficittaxes', 8377), ('gaysandlesbianslegalissuesstatesworkers', 8378), ('campaignfinancecorporationsabcnewsweek', 8379), ('sums', 8380), ('congresselectionsenvironmentgovernmentregulationcampaignadvertising', 8381), ('alaskans', 8382), ('occupywallstreetpublicsafety', 8383), ('woodruff', 8384), ('foreignpolicyhistorynuclear', 8385), ('thanksgiving', 8386), ('july', 8387), ('celebrate', 8388), ('gunshealthcare', 8389), ('misadventures', 8390), ('crimeforeignpolicyhomelandsecuritylegalissues', 8391), ('disclosing', 8392), ('whistleblower', 8393), ('exception', 8394), ('agriculturefoodsafetymarketregulation', 8395), ('backyard', 8396), ('healthcareimmigration', 8397), ('nonus', 8398), ('campaignfinancecorporationselectionscampaignadvertisingtransparencyunions', 8399), ('cycle', 8400), ('superpacs', 8401), ('childreneducationwomen', 8402), ('congressstates', 8403), ('misogynist', 8404), ('bigot', 8405), ('epic', 8406), ('proportions', 8407), ('civilrightshomelandsecurityterrorism', 8408), ('federalbudgetforeignpolicyabcnewsweek', 8409), ('campaignfinancedebtstatebudgettransportation', 8410), ('unloading', 8411), ('foodsafety', 8412), ('restricts', 8413), ('salt', 8414), ('economyinfrastructurejobs', 8415), ('publichealthpublicsafety', 8416), ('lyme', 8417), ('infectious', 8418), ('incidence', 8419), ('electionshealthcare', 8420), ('southerland', 8421), ('economystimulustransparency', 8422), ('bidens', 8423), ('with', 8424), ('transparency', 8425), ('agricultureanimalsfoodsafetypublichealthscience', 8426), ('turkeys', 8427), ('weigh', 8428), ('298', 8429), ('weighed', 8430), ('132', 8431), ('citybudgetcitygovernmenttransportation', 8432), ('fixes', 8433), ('candidatesbiographyeconomyjobs', 8434), ('likes', 8435), ('firing', 8436), ('spared', 8437), ('diet', 8438), ('candidatesbiographycongressvotingrecord', 8439), ('historymilitarystates', 8440), ('retained', 8441), ('joined', 8442), ('agriculturepublicsafetytransportation', 8443), ('tractor', 8444), ('gunspundits', 8445), ('270000', 8446), ('candidatesbiographylegalissuessupremecourt', 8447), ('joanne', 8448), ('kloppenburg', 8449), ('society', 8450), ('congresshistorymilitaryterrorism', 8451), ('commanderinchiefs', 8452), ('commanderinchief', 8453), ('foreignpolicyislamreligion', 8454), ('ally', 8455), ('wahhabism', 8456), ('devil', 8457), ('jobswomen', 8458), ('immigrationstatebudget', 8459), ('ranger', 8460), ('recon', 8461), ('jobaccomplishmentsmessagemachine2012statefinances', 8462), ('legalissuestechnology', 8463), ('doctrine', 8464), ('candidatesbiographyeducationelections', 8465), ('unsealed', 8466), ('immigrationlegalissues', 8467), ('appears', 8468), ('reasonably', 8469), ('suspicious', 8470), ('iraqmilitarypundits', 8471), ('afghanistanfederalbudgetiraq', 8472), ('crimegunsimmigrationmarijuana', 8473), ('seized', 8474), ('currency', 8475), ('outbound', 8476), ('civilrightscrimecriminaljusticelegalissues', 8477), ('congresscongressionalrulesforeignpolicyhistorytrade', 8478), ('delano', 8479), ('immigrationpublichealth', 8480), ('examined', 8481), ('quarantined', 8482), ('measles', 8483), ('drugsmarijuanawelfare', 8484), ('healthcarelabor', 8485), ('finally', 8486), ('congresseducationfederalbudget', 8487), ('correctionsandupdateselections', 8488), ('scale', 8489), ('crimecriminaljusticedeathpenalty', 8490), ('electionsfloridafloridaamendmentslegalissuesmessagemachineredistricting', 8491), ('draws', 8492), ('drugsmarijuanapublichealth', 8493), ('thc', 8494), ('surpass', 8495), ('corporationseducationsmallbusinessworkers', 8496), ('foreignpolicylegalissuesmilitary', 8497), ('engage', 8498), ('federalbudgetmedicaidstatebudgettaxes', 8499), ('sends', 8500), ('energyjobaccomplishments', 8501), ('307', 8502), ('230', 8503), ('animalselections', 8504), ('shark', 8505), ('bushadministrationforeignpolicyterrorism', 8506), ('upset', 8507), ('economyelectionsfloridaamendments', 8508), ('ethicstaxestransparencytransportation', 8509), ('footed', 8510), ('bachelor', 8511), ('citygovernmentstatebudget', 8512), ('pensionsretirementstatebudgetstatefinancesunionsworkers', 8513), ('immigrationsports', 8514), ('players', 8515), ('candidatesbiographyvotingrecord', 8516), ('countybudgetcountygovernmentcrimecriminaljustice', 8517), ('overtime', 8518), ('passively', 8519), ('chairs', 8520), ('watching', 8521), ('economyhistorylegalissuesstateswomen', 8522), ('handful', 8523), ('financialregulationvotingrecord', 8524), ('cracking', 8525), ('congressfloridaforeignpolicytrade', 8526), ('foreignpolicynuclearterrorism', 8527), ('lined', 8528), ('energyinfrastructurepublichealthpublicsafety', 8529), ('exposure', 8530), ('electromagnetic', 8531), ('fields', 8532), ('leukemia', 8533), ('candidatesbiographydebateseconomyhistoryjobaccomplishmentsmessagemachine2014statebudgetstatefinances', 8534), ('populationstatebudgetstatefinances', 8535), ('crimedrugshomelandsecurity', 8536), ('capandtradeclimatechangeenergyenvironment', 8537), ('dispute', 8538), ('statebudgettourism', 8539), ('immigrationjobs', 8540), ('lindsey', 8541), ('shortage', 8542), ('congressionalrulesvotingrecord', 8543), ('barr', 8544), ('bipartisanshipclimatechangecongresscongressionalrulesvotingrecord', 8545), ('familiesincomelaborworkers', 8546), ('povertywelfare', 8547), ('saysthere', 8548), ('jobaccomplishments', 8549), ('statebudgetstatesstimulus', 8550), ('abortioneducationhealthcare', 8551), ('obriens', 8552), ('compulsory', 8553), ('congressionalrulesforeignpolicytradetransparency', 8554), ('candidatesbiographycrimesexualitywomen', 8555), ('civilrightscriminaljusticeterrorism', 8556), ('definitions', 8557), ('alqaeda', 8558), ('loosely', 8559), ('jobspovertypunditsworkers', 8560), ('historyiraqmilitary', 8561), ('our', 8562), ('debtdeficiteconomy', 8563), ('consequence', 8564), ('laborstatefinancesunions', 8565), ('healthcarepolls', 8566), ('candidatesbiographydebtdeficiteconomygovernmentregulationhistoryjobaccomplishmentslaborpensionsretirementstatebudgetstatefinancestaxesunionswealthworkers', 8567), ('speaking', 8568), ('federalbudgetmedicaid', 8569), ('sacrificing', 8570), ('refusing', 8571), ('messagemachine2014retirementsocialsecurityworkers', 8572), ('iowa', 8573), ('joni', 8574), ('ernst', 8575), ('immigrationworkers', 8576), ('explicitly', 8577), ('afghanistanbushadministrationcorrectionsandupdatesiraqtaxes', 8578), ('wartime', 8579), ('governmentefficiencystatebudget', 8580), ('lobby', 8581), ('congresssupremecourt', 8582), ('yearlong', 8583), ('citygovernmenteconomysmallbusinesstechnology', 8584), ('startups', 8585), ('patents', 8586), ('candidatesbiographyforeignpolicynuclear', 8587), ('rand', 8588), ('environmenttrade', 8589), ('katie', 8590), ('mcgintyactually', 8591), ('wing', 8592), ('jobsmessagemachinestimulus', 8593), ('kagen', 8594), ('77000', 8595), ('educationgovernmentefficiencystatebudgettransportation', 8596), ('sinks', 8597), ('hungerimmigrationpoverty', 8598), ('contact', 8599), ('recreationtourism', 8600), ('airconditioned', 8601), ('lens', 8602), ('energyjobs', 8603), ('advisers', 8604), ('childreneconomyeducationfamilieshealthcarewomen', 8605), ('fiftythree', 8606), ('teens', 8607), ('foreignpolicyhomelandsecurity', 8608), ('interpol', 8609), ('disapproval', 8610), ('childrendiversityfamiliesmarriagepunditssexuality', 8611), ('wedlock', 8612), ('mathematically', 8613), ('sox', 8614), ('playoffs', 8615), ('countygovernmentstatebudgetunions', 8616), ('170000', 8617), ('activities', 8618), ('energygaspricestaxestransportation', 8619), ('foreignpolicygunslegalissues', 8620), ('all', 8621), ('citybudgetcitygovernmentenergy', 8622), ('considering', 8623), ('deficiteconomyfederalbudget', 8624), ('congressmedicarenewhampshire2012', 8625), ('federalbudgethomelandsecuritypundits', 8626), ('calligraphers', 8627), ('86000', 8628), ('97000', 8629), ('55000', 8630), ('foreignpolicymilitaryabcnewsweek', 8631), ('ratification', 8632), ('treaties', 8633), ('crimecriminaljusticestatebudget', 8634), ('reductions', 8635), ('lowered', 8636), ('abortionlegalissues', 8637), ('vs', 8638), ('define', 8639), ('conception', 8640), ('ethicstransparency', 8641), ('countybudgetrecreation', 8642), ('painting', 8643), ('crimeethicsmessagemachine', 8644), ('grand', 8645), ('fernandez', 8646), ('guts', 8647), ('prosecute', 8648), ('foreignpolicyhistoryhomelandsecurityterrorism', 8649), ('deficitfederalbudgetmilitarypensionsretirement', 8650), ('governmentregulationjobsvotingrecordworkers', 8651), ('citybudgetdebteconomyhistoryjobaccomplishmentslaborpensionsretirementtaxesworkers', 8652), ('abortionpublichealth', 8653), ('curb', 8654), ('citybudgetcitygovernmentdisabilitygovernmentefficiencylaborpublicsafetyretirementunions', 8655), ('thirtyseven', 8656), ('municipalities', 8657), ('diversityislam', 8658), ('manhattan', 8659), ('rally', 8660), ('chanting', 8661), ('federalbudgetgovernmentefficiency', 8662), ('moroccans', 8663), ('pottery', 8664), ('diversityguns', 8665), ('outstripping', 8666), ('bipartisanshipelections', 8667), ('nunn', 8668), ('handpicked', 8669), ('energyenvironmentscience', 8670), ('renewable', 8671), ('governmentregulationhealthcarepolls', 8672), ('economystimulustaxes', 8673), ('debateswomen', 8674), ('militarypatriotism', 8675), ('franken', 8676), ('economyfederalbudgetstimulus', 8677), ('manner', 8678), ('discussed', 8679), ('messagemachinevotingrecord', 8680), ('nye', 8681), ('lock', 8682), ('percentof', 8683), ('energygaspricesgovernmentregulationtransportationvotingrecord', 8684), ('170', 8685), ('corporationseconomyhealthcare', 8686), ('cross', 8687), ('headquarters', 8688), ('economygovernmentefficiencystatebudgettaxes', 8689), ('7step', 8690), ('700000', 8691), ('jobsnewhampshire2012workers', 8692), ('135000', 8693), ('incomejobstaxesworkers', 8694), ('reelection', 8695), ('consumersafetyfinancialregulation', 8696), ('358', 8697), ('filings', 8698), ('electionspovertywelfare', 8699), ('featuring', 8700), ('author', 8701), ('registering', 8702), ('unamerican', 8703), ('candidatesbiographyeconomyforeignpolicyhistorytrade', 8704), ('sudden', 8705), ('campaignfinancelegalissues', 8706), ('changing', 8707), ('governing', 8708), ('educationfederalbudgetnewhampshire2012', 8709), ('consumersafetyenergymarketregulation', 8710), ('friendly', 8711), ('immigrationsmallbusiness', 8712), ('startup', 8713), ('climatechangeenergyvotingrecord', 8714), ('candidatesbiographyeconomymessagemachine2012', 8715), ('educationlaborunions', 8716), ('economyjobaccomplishmentstaxes', 8717), ('jumpstart', 8718), ('bushadministrationtaxes', 8719), ('targeted', 8720), ('gunsrecreation', 8721), ('dare', 8722), ('rapes', 8723), ('murders', 8724), ('healthcaremessagemachine2014', 8725), ('kansans', 8726), ('childrenfederalbudgethealthcarepublichealth', 8727), ('hits', 8728), ('preventive', 8729), ('flu', 8730), ('congressjobaccomplishmentsmessagemachine2012votingrecord', 8731), ('healthcarelaborstatebudgetstatefinances', 8732), ('unreasonable', 8733), ('screamed', 8734), ('legalissuesmilitarycampaignadvertisingpatriotismveterans', 8735), ('flagburning', 8736), ('ridiculous', 8737), ('civilrights', 8738), ('wasilla', 8739), ('respected', 8740), ('consider', 8741), ('library', 8742), ('deficitmessagemachine2012pensionsstatefinances', 8743), ('kyrillos', 8744), ('tune', 8745), ('repay', 8746), ('medicaresexuality', 8747), ('172', 8748), ('penis', 8749), ('pumps', 8750), ('campaignfinancemilitary', 8751), ('donating', 8752), ('punishment', 8753), ('uniform', 8754), ('crimejobaccomplishments', 8755), ('homelandsecurityimmigrationiraqterrorism', 8756), ('truethat', 8757), ('present', 8758), ('ciudad', 8759), ('candidatesbiographylegalissuesmessagemachinetaxes', 8760), ('cheated', 8761), ('deadbeat', 8762), ('cattle', 8763), ('candidatesbiographyeconomyfinancialregulation', 8764), ('federalbudgetforeignpolicymilitary', 8765), ('gamblingmarketregulation', 8766), ('rubberstamped', 8767), ('phony', 8768), ('campaignfinancecorporationsenergyethicsjobsstimulus', 8769), ('gunsmilitaryterrorism', 8770), ('chattanooga', 8771), ('discharging', 8772), ('fulton', 8773), ('19yearolds', 8774), ('civilrightsgovernmentregulationincomewomenworkers', 8775), ('repealed', 8776), ('treated', 8777), ('fairly', 8778), ('childrenfamilieshealthcare', 8779), ('preexisting', 8780), ('condition', 8781), ('uninsurable', 8782), ('campaignfinancecorporationselectionssmallbusinessunions', 8783), ('unlimited', 8784), ('crimecriminaljusticegunspublicsafetypublicservice', 8785), ('eightythree', 8786), ('twentyfour', 8787), ('economyfederalbudgethistorytaxes', 8788), ('redistribution', 8789), ('characteristic', 8790), ('healthcarepopculture', 8791), ('atlantaarea', 8792), ('selfdestruct', 8793), ('abortionhealthcarepublichealth', 8794), ('economyincomejobaccomplishmentsjobsworkers', 8795), ('childrenguns', 8796), ('academy', 8797), ('pediatrics', 8798), ('federalbudgetvotingrecord', 8799), ('economyfinancialregulationmarketregulation', 8800), ('statebudgettransportation', 8801), ('pennies', 8802), ('turning', 8803), ('200mile', 8804), ('stretch', 8805), ('interstate', 8806), ('toll', 8807), ('crimecriminaljusticeeconomygovernmentregulationjobslegalissuesprivacypublicsafety', 8808), ('prisons', 8809), ('reincarcerated', 8810), ('electionsimmigration', 8811), ('testimony', 8812), ('witnessed', 8813), ('electionspoverty', 8814), ('motor', 8815), ('messagemachinesciencestimulus', 8816), ('wyden', 8817), ('exotic', 8818), ('energypundits', 8819), ('pipes', 8820), ('wood', 8821), ('joking', 8822), ('messagemachinestatebudgettaxes', 8823), ('diversitywomen', 8824), ('image', 8825), ('blondwomen', 8826), ('anchors', 8827), ('foreignpolicymilitarypundits', 8828), ('messagemachinesciencestates', 8829), ('passes', 8830), ('microchips', 8831), ('talks', 8832), ('seceding', 8833), ('floridahealthcarelegalissues', 8834), ('statutes', 8835), ('consult', 8836), ('filing', 8837), ('campaignfinancegunspundits', 8838), ('corporationsmessagemachinetaxes', 8839), ('shouldnt', 8840), ('incomeworkers', 8841), ('federalbudgetpollspunditstaxesabcnewsweek', 8842), ('governmentregulationtechnology', 8843), ('proposalputs', 8844), ('determining', 8845), ('types', 8846), ('governmentregulationprivacymarketregulation', 8847), ('climatechangefloridapublichealthscience', 8848), ('quicker', 8849), ('mosquitoes', 8850), ('mature', 8851), ('bite', 8852), ('metabolism', 8853), ('incubate', 8854), ('energygasprices', 8855), ('alcoholchildrencrimepublichealthpublicsafety', 8856), ('binge', 8857), ('popculturerecreation', 8858), ('veganfriendly', 8859), ('electionshistoryhumanrightslegalissues', 8860), ('correctionsandupdateseconomystates', 8861), ('10in', 8862), ('jobaccomplishmentsstimulus', 8863), ('allocation', 8864), ('citygovernmentcivilrightsgaysandlesbiansjobs', 8865), ('campaignfinanceelectionsgovernmentregulationhistorytaxestransparency', 8866), ('trail', 8867), ('childreneducationpovertyreligionstatebudget', 8868), ('consumersafetyhousing', 8869), ('hair', 8870), ('stylists', 8871), ('scrutiny', 8872), ('consultants', 8873), ('candidatesbiographyconsumersafetymessagemachine', 8874), ('justify', 8875), ('congressionalrules', 8876), ('economyfederalbudgetmessagemachinestimulustaxesvotingrecord', 8877), ('debteducationfederalbudget', 8878), ('childreneducationenergy', 8879), ('peaked', 8880), ('citygovernmentgovernmentefficiencypublicsafety', 8881), ('busiest', 8882), ('foreignpolicyiraq', 8883), ('childrencrimecriminaljusticeimmigration', 8884), ('9yearold', 8885), ('campaignfinancecrimecriminaljustice', 8886), ('hamilton', 8887), ('42000', 8888), ('stripclub', 8889), ('distributors', 8890), ('energyforeignpolicy', 8891), ('miserably', 8892), ('economyjobslabor', 8893), ('sectors', 8894), ('leisure', 8895), ('hospitality', 8896), ('fall', 8897), ('highpaying', 8898), ('subminimum', 8899), ('candidatesbiographymessagemachinetaxes', 8900), ('claiming', 8901), ('climatechangecorrectionsandupdatesenvironmentweather', 8902), ('childrencitygovernment', 8903), ('levy', 8904), ('corporationscorrectionsandupdateseducationincomeworkers', 8905), ('daughters', 8906), ('abortionmessagemachine2012statessupremecourtwomen', 8907), ('overturn', 8908), ('v', 8909), ('disabilityfederalbudgetsocialsecurity', 8910), ('ssdi', 8911), ('ssi', 8912), ('abortionvotingrecord', 8913), ('straus', 8914), ('reproductive', 8915), ('homelandsecurityimmigrationmessagemachine', 8916), ('kyl', 8917), ('hell', 8918), ('524', 8919), ('candidatesbiographyeconomyforeignpolicyjobsmessagemachine2012', 8920), ('owe', 8921), ('citygovernmentcorrectionsandupdatessports', 8922), ('appraises', 8923), ('appraisal', 8924), ('consumersafetyenvironment', 8925), ('sherwinwilliams', 8926), ('paint', 8927), ('poisoning', 8928), ('candidatesbiographyeducationelectionsethics', 8929), ('cloud', 8930), ('federalbudgetjobstransportationworkers', 8931), ('middleclass', 8932), ('federalbudgetincomepublicserviceretirementsocialsecurity', 8933), ('gaspricesincome', 8934), ('energypopulation', 8935), ('crimefederalbudgetgovernmentefficiencymedicaidmedicaretaxeswelfare', 8936), ('debatesjobs', 8937), ('incometaxeswealth', 8938), ('deductions', 8939), ('laborpublichealth', 8940), ('sick', 8941), ('alcohollegalissues', 8942), ('fakenews', 8943), ('dreams', 8944), ('passions', 8945), ('healthcarehistorystates', 8946), ('federalism', 8947), ('afghanistanfamiliesgovernmentefficiencyhomelandsecurityiraqmilitaryveterans', 8948), ('missing', 8949), ('repatriate', 8950), ('environmentincome', 8951), ('pleasant', 8952), ('operated', 8953), ('jenkinsons', 8954), ('jobaccomplishmentsmarketregulationstatebudget', 8955), ('federalbudgetincometaxeswealth', 8956), ('extension', 8957), ('climatechangeeconomyenergygasprices', 8958), ('healthcaremedicarevotingrecord', 8959), ('foreignpolicyhomelandsecuritynuclear', 8960), ('warheads', 8961), ('economyinfrastructurerecreationstatebudgetstatefinancestourism', 8962), ('pell', 8963), ('commuter', 8964), ('corporationsjobaccomplishmentsjobssmallbusiness', 8965), ('statebudgetstatefinancesstates', 8966), ('reserve', 8967), ('candidatesbiographyethicsnaturaldisasters', 8968), ('empty', 8969), ('apartment', 8970), ('buildings', 8971), ('rents', 8972), ('orleans', 8973), ('drugslegalissuesmarijuanataxes', 8974), ('raked', 8975), ('licensing', 8976), ('educationforeignpolicy', 8977), ('primaryage', 8978), ('healthcaremedicaidpublichealth', 8979), ('immigrationpundits', 8980), ('vast', 8981), ('arriving', 8982), ('environmentforeignpolicylegalissues', 8983), ('copenhagen', 8984), ('will', 8985), ('again', 8986), ('candidatesbiographycongress', 8987), ('manufacturer', 8988), ('thatd', 8989), ('candidatesbiographyfederalbudgettrade', 8990), ('correctionsandupdatesforeignpolicy', 8991), ('agent', 8992), ('client', 8993), ('duvalier', 8994), ('despot', 8995), ('correctionsandupdatesenergyfloridaamendments', 8996), ('fool', 8997), ('amending', 8998), ('metering', 8999), ('gaysandlesbianspollssupremecourt', 9000), ('radically', 9001), ('legalize', 9002), ('marketregulationsmallbusiness', 9003), ('childrencrimemessagemachine2012votingrecord', 9004), ('militarynewhampshire2012', 9005), ('iraqsocialsecuritytaxes', 9006), ('saypresident', 9007), ('137', 9008), ('foreignpolicyhomelandsecurityimmigrationterrorism', 9009), ('vet', 9010), ('economywealth', 9011), ('walton', 9012), ('abortioncorrectionsandupdatesdebates', 9013), ('citygovernmentgovernmentregulationhousinginfrastructuremessagemachine2012', 9014), ('hales', 9015), ('sweeping', 9016), ('infill', 9017), ('basics', 9018), ('crimemessagemachinestatebudget', 9019), ('atwaters', 9020), ('wasting', 9021), ('lavish', 9022), ('dubbed', 9023), ('golf', 9024), ('musuem', 9025), ('candidatesbiographymilitarysexuality', 9026), ('vitter', 9027), ('answered', 9028), ('honoring', 9029), ('healthcarepublichealthstatessupremecourt', 9030), ('educationpublichealthtaxes', 9031), ('prop', 9032), ('cheats', 9033), ('economyincomepovertywealthwomenworkers', 9034), ('educationincomestatebudgetstatefinancesstatestaxes', 9035), ('civilrightslegalissuesterrorism', 9036), ('lawabiding', 9037), ('childreneconomyfamiliespoverty', 9038), ('struggling', 9039), ('qualifies', 9040), ('subsidized', 9041), ('electionslegalissuesredistrictingstates', 9042), ('nc', 9043), ('unopposed', 9044), ('gerrymandering', 9045), ('energyoilspillabcnewsweek', 9046), ('has', 9047), ('constrained', 9048), ('crimecriminaljusticehealthcare', 9049), ('navigators', 9050), ('identity', 9051), ('immigrationlabor', 9052), ('brings', 9053), ('replace', 9054), ('economyjobsstatestaxes', 9055), ('candidatesbiographyimmigrationtaxes', 9056), ('onehalf', 9057), ('jobslaborwomen', 9058), ('agriculturediversitypunditsabcnewsweek', 9059), ('shirley', 9060), ('gaysandlesbiansmilitaryabcnewsweek', 9061), ('unit', 9062), ('compromising', 9063), ('readiness', 9064), ('candidatesbiographymarriagemessagemachine', 9065), ('incometaxestransportation', 9066), ('commute', 9067), ('educationelectionshealthcaremessagemachine', 9068), ('boyd', 9069), ('economyfederalbudgetgovernmentefficiency', 9070), ('citybudgetcitygovernmenttaxes', 9071), ('burdened', 9072), ('deficitfederalbudgethealthcare', 9073), ('singlebiggest', 9074), ('factor', 9075), ('drugseconomyfederalbudgethealthcarepovertypublichealth', 9076), ('diversityeducationimmigration', 9077), ('languages', 9078), ('statebudgettaxesvotingrecord', 9079), ('billiondollar', 9080), ('propertytax', 9081), ('childrenmessagemachine', 9082), ('injecting', 9083), ('controversial', 9084), ('economyfederalbudgethealthcarejobs', 9085), ('analysts', 9086), ('correctionsandupdateslaborpublicserviceretirementsocialsecuritystatefinancesworkers', 9087), ('sixtypercent', 9088), ('retireesdont', 9089), ('ebolapublichealthtransportation', 9090), ('foreignpolicylegalissuestechnologytrade', 9091), ('stole', 9092), ('inintellectual', 9093), ('citygovernmentelections', 9094), ('14th', 9095), ('economymessagemachinestatebudget', 9096), ('economyjobspunditsabcnewsweek', 9097), ('seems', 9098), ('reflect', 9099), ('childrencrimefamilieslegalissues', 9100), ('defendants', 9101), ('fatherabsent', 9102), ('debtdeficiteconomyfederalbudget', 9103), ('economyimmigrationjobs', 9104), ('foreignpolicyreligionterrorism', 9105), ('feisal', 9106), ('abdul', 9107), ('raufs', 9108), ('economyhistory', 9109), ('leadup', 9110), ('blown', 9111), ('candidatesbiographyelections', 9112), ('talked', 9113), ('switching', 9114), ('afghanistanfederalbudgetiraqterrorism', 9115), ('logistically', 9116), ('environmentstatebudgetstatefinances', 9117), ('intervenors', 9118), ('economystatestourismtransportation', 9119), ('uhaul', 9120), ('flee', 9121), ('golden', 9122), ('historymessagemachine2012', 9123), ('endowed', 9124), ('creator', 9125), ('declaration', 9126), ('correctionsandupdatesobamabirthcertificate', 9127), ('barry', 9128), ('soetoro', 9129), ('taxesvotingrecord', 9130), ('punditsterrorism', 9131), ('petition', 9132), ('blew', 9133), ('debtstatebudgetstatefinancestaxestransportation', 9134), ('budgeting', 9135), ('federalbudgetpunditsabcnewsweek', 9136), ('electionstechnology', 9137), ('burying', 9138), ('dishonest', 9139), ('abortionmessagemachine', 9140), ('forms', 9141), ('pill', 9142), ('candidatesbiographycorrectionsandupdatesethicsmessagemachine2012', 9143), ('fined', 9144), ('ethics', 9145), ('citygovernmentcorporationsjobssmallbusiness', 9146), ('candidatesbiographymessagemachine2012taxes', 9147), ('carried', 9148), ('trick', 9149), ('foreignpolicyisrael', 9150), ('chinacrime', 9151), ('crimedebtfederalbudgetsocialsecurity', 9152), ('citybudgetcitygovernmentelectionstaxes', 9153), ('subsidize', 9154), ('animalspublichealth', 9155), ('meat', 9156), ('yulin', 9157), ('statebudgetstatefinancesstatestaxes', 9158), ('assemblies', 9159), ('consumersafetymarketregulation', 9160), ('slice', 9161), ('slices', 9162), ('crimetaxes', 9163), ('refund', 9164), ('gaysandlesbianskagannominationmilitary', 9165), ('barred', 9166), ('up', 9167), ('economylegalissuesmarketregulation', 9168), ('businessfriendly', 9169), ('24th', 9170), ('energyvotingrecord', 9171), ('abortionsupremecourt', 9172), ('trumphas', 9173), ('sister', 9174), ('appeals', 9175), ('hardcore', 9176), ('populationwater', 9177), ('crimeelectionstransportation', 9178), ('survivors', 9179), ('ridesharing', 9180), ('economylaborstatebudgetstatefinancesworkers', 9181), ('belief', 9182), ('employeeswe', 9183), ('healthcarenewhampshire2012', 9184), ('treat', 9185), ('criminaljusticedrugsmarijuana', 9186), ('possession', 9187), ('sciencetechnology', 9188), ('gps', 9189), ('enjoying', 9190), ('miracle', 9191), ('foreignpolicyhomelandsecurityterrorism', 9192), ('largescale', 9193), ('foreignpolicytaxes', 9194), ('estonia', 9195), ('laborunions', 9196), ('immigrationmessagemachinetaxes', 9197), ('economyjobslaborlegalissues', 9198), ('childreneducationpublichealthrecreation', 9199), ('respiratory', 9200), ('militaryreligion', 9201), ('deathpenaltyelections', 9202), ('populationin', 9203), ('hemisphere', 9204), ('congressfinancialregulation', 9205), ('civilrightselectionshistoryhumanrightslegalissues', 9206), ('comparable', 9207), ('abortionhealthcarepublichealthwomen', 9208), ('provider', 9209), ('campaignfinanceelectionsjobaccomplishments', 9210), ('correctionsandupdatesgaysandlesbianshumanrightsislamreligionwomen', 9211), ('subjugation', 9212), ('capandtradeclimatechangeeconomyhealthcaremessagemachine', 9213), ('energygaspricestourism', 9214), ('childrenfamiliesmarriagepoverty', 9215), ('probability', 9216), ('citybudgetcitygovernmentelectionsethics', 9217), ('alison', 9218), ('alter', 9219), ('64000', 9220), ('criminaljusticelegalissuessupremecourt', 9221), ('justices', 9222), ('energystimulus', 9223), ('corporationseducationsmallbusinessstatebudgettaxes', 9224), ('superintendent', 9225), ('pridemore', 9226), ('bankruptcycandidatesbiographygambling', 9227), ('incomepopulationwealth', 9228), ('retirementsocialsecurityvotingrecord', 9229), ('push', 9230), ('legalissuespopculturetechnology', 9231), ('crimemarijuanatransportation', 9232), ('illicit', 9233), ('licit', 9234), ('electionsguns', 9235), ('stockman', 9236), ('gunfilled', 9237), ('zones', 9238), ('economyeducationstimulus', 9239), ('dorms', 9240), ('repaired', 9241), ('candidatesbiographyelectionshistory', 9242), ('bald', 9243), ('deficiteducationfederalbudget', 9244), ('agriculturetaxes', 9245), ('trees', 9246), ('bankruptcydebteconomyhistoryjobaccomplishmentsmessagemachine2014campaignadvertisingstatebudgetstatefinancestaxes', 9247), ('investors', 9248), ('bust', 9249), ('healthcareimmigrationpoverty', 9250), ('275', 9251), ('crimeelectionsguns', 9252), ('restored', 9253), ('childrencrimeeducationwelfare', 9254), ('preschool', 9255), ('dole', 9256), ('childrencivilrightseducationhumanrightsimmigrationlegalissuespovertystatebudgetwelfareworkers', 9257), ('medicaremessagemachine2012', 9258), ('congressdebteconomy', 9259), ('merkley', 9260), ('ethicsmessagemachine', 9261), ('ebay', 9262), ('whitman', 9263), ('porn', 9264), ('debatessmallbusinesstaxes', 9265), ('definition', 9266), ('foreignpolicymilitaryterrorism', 9267), ('criticism', 9268), ('abortioncrimevotingrecordwomen', 9269), ('runyan', 9270), ('candidatesbiographyincomevotingrecord', 9271), ('80s', 9272), ('candidatesbiographycitybudgetcitygovernmentjobaccomplishmentsmessagemachine2014', 9273), ('7million', 9274), ('110million', 9275)])




```python
embedding_matrix=np.zeros((2560,20))
```


```python
embedding_matrix[0]
```




    array([0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0., 0.,
           0., 0., 0.])




```python
embedding_index.get('without')
```




    array([ 7.1347e-02,  1.3955e-02,  4.2260e-01, -2.7192e-01,  2.4444e-02,
            5.4742e-01, -3.6910e-01,  1.0744e-01,  4.4933e-01, -1.0871e-01,
            3.2773e-01,  2.5514e-02,  1.8760e-01,  7.1466e-02,  4.2016e-01,
           -9.3691e-01, -1.8025e-01,  1.9189e-01, -1.6659e-01, -2.1146e-02,
           -2.6291e-01,  3.9201e-01, -3.0405e-02, -2.3321e-01,  3.1795e-01,
            2.0729e-01, -5.0425e-01, -9.8723e-01,  2.2798e-01, -1.2164e-01,
            3.9037e-01, -3.7607e-02, -6.4537e-01, -5.3506e-01,  1.7855e-01,
           -3.3326e-01, -1.8963e-01, -2.7955e-02, -7.1821e-01, -9.1901e-02,
           -2.1918e-01,  1.5901e-01,  5.5907e-01, -1.8219e-01, -4.6256e-02,
           -4.2019e-01,  2.2868e-01, -4.9949e-01, -2.8316e-01, -9.7350e-01,
            5.9732e-01,  2.6562e-02, -3.1037e-01,  1.3910e+00,  1.2275e-01,
           -2.4381e+00,  2.9824e-02, -3.9304e-01,  1.8912e+00,  3.6262e-01,
           -3.8255e-01,  6.0898e-01, -4.8231e-01,  4.0125e-02,  9.5468e-01,
            1.8180e-03,  6.8199e-01,  8.7480e-02, -1.0827e-01, -2.0613e-01,
           -8.0502e-01, -2.5513e-01,  3.6673e-01, -8.1574e-01,  3.7747e-01,
            4.4177e-01, -2.7978e-01, -1.2400e-01, -1.0991e+00, -1.3992e-01,
            5.4497e-01, -4.9608e-01, -5.5284e-01,  3.8242e-01, -1.4233e+00,
            2.9444e-01,  3.6053e-01,  3.8297e-01, -2.4599e-01, -3.7326e-01,
           -1.8529e-01, -5.3523e-01, -3.8073e-02, -7.3348e-02, -3.2447e-01,
            6.1967e-02,  1.8508e-01, -1.2532e-01,  6.4393e-01,  9.4035e-02],
          dtype=float32)




```python
padded_seq[0]
```




    array([ 171,    1, 3808,  942,  336,  311,  211, 2286,  509, 2323,    0,
              0,    0,    0,    0,    0,    0,    0,    0,    0])



---------------------------------------------------------------------------------------------------------------------------------


```python
def get_key(d,value):
    for key, val in d.items():
        if value==val:
            return key
```


```python
embedding_matrix = np.zeros((2560,20,100))
```

for i in range(len(padded_seq)):
    for j in range(len(padded_seq[i])):
        try: 
            word = get_key(word_index, padded_seq[i][j])
            vector = embedding_index.get(word)
            embedding_matrix[i][j] = vector
        except:
            embedding_matrix[i][j] = 0


```python
embedding_matrix[0]

```




    array([[0., 0., 0., ..., 0., 0., 0.],
           [0., 0., 0., ..., 0., 0., 0.],
           [0., 0., 0., ..., 0., 0., 0.],
           ...,
           [0., 0., 0., ..., 0., 0., 0.],
           [0., 0., 0., ..., 0., 0., 0.],
           [0., 0., 0., ..., 0., 0., 0.]])




```python
len(embedding_matrix)
```




    2560




```python
#Create Embedding_matrix
#embedding_matrix=np.zeros((2560,20))
#for word,i in word_index.items():
    #embedding_vector = embedding_index.get(word)
    #if embedding_vector is not None:
        #embedding_matrix[i]=embedding_vector
```


```python
Sequences[0][0]
```




    171




```python

```


```python
len(embedding_matrix)
```




    2560




```python
padded_seq[0]
```




    array([ 171,    1, 3808,  942,  336,  311,  211, 2286,  509, 2323,    0,
              0,    0,    0,    0,    0,    0,    0,    0,    0])




```python
len(np.unique(y))

```




    6



---------------------------------------------------------------------------------------------------------------------------------


```python
from keras.layers import LSTM, Dropout,Dense,Embedding
from keras import Sequential
```


```python
model = Sequential([
        keras.layers.Embedding(vocab_size,128,
                              ),
        keras.layers.BatchNormalization(),
#         keras.layers.Bidirectional(keras.layers.LSTM(128,return_sequences=True)),
        keras.layers.Dense(128, activation='relu', kernel_regularizer=tf.keras.regularizers.L2(0.002)),
        keras.layers.GlobalMaxPool1D(), # Remove flatten layer
        keras.layers.Dense(64, activation='relu', kernel_regularizer=tf.keras.regularizers.L2(0.002)),
        keras.layers.Dropout(0.3),
        keras.layers.Dense(32, activation='relu', kernel_regularizer=tf.keras.regularizers.L2(0.002)),
        keras.layers.Dropout(0.3),
        keras.layers.Dense(1,activation='softmax')
    ])

```

model=Sequential([Embedding(vocab_size+1,128,weights=[embedding_matrix],trainable=False),
                            Dropout(0.2),
                            LSTM(128,return_sequences=True),
                            LSTM(128),
                 Dropout(0.3),
                 Dense(128),
                 Dropout(0.3),
                 Dense(64,),
                 Dense(6,activation='relu')]) 


```python
model.compile(loss='binary_crossentropy',optimizer='adam',metrics='accuracy')
model.summary()
```

    Model: "sequential"
    _________________________________________________________________
     Layer (type)                Output Shape              Param #   
    =================================================================
     embedding (Embedding)       (None, None, 128)         1187200   
                                                                     
     batch_normalization (BatchN  (None, None, 128)        512       
     ormalization)                                                   
                                                                     
     dense (Dense)               (None, None, 128)         16512     
                                                                     
     global_max_pooling1d (Globa  (None, 128)              0         
     lMaxPooling1D)                                                  
                                                                     
     dense_1 (Dense)             (None, 64)                8256      
                                                                     
     dropout (Dropout)           (None, 64)                0         
                                                                     
     dense_2 (Dense)             (None, 32)                2080      
                                                                     
     dropout_1 (Dropout)         (None, 32)                0         
                                                                     
     dense_3 (Dense)             (None, 1)                 33        
                                                                     
    =================================================================
    Total params: 1,214,593
    Trainable params: 1,214,337
    Non-trainable params: 256
    _________________________________________________________________
    


```python
history =model.fit(embedding_matrix, 
                        y_train,
                        batch_size=512,
                        epochs=20,
                        verbose=1,
                        validation_split=0.2)
```

    Epoch 1/20
    WARNING:tensorflow:Model was constructed with shape (None, None) for input KerasTensor(type_spec=TensorSpec(shape=(None, None), dtype=tf.float32, name='embedding_input'), name='embedding_input', description="created by layer 'embedding_input'"), but it was called on an input with incompatible shape (512, 20, 100).
    


    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    Input In [80], in <cell line: 1>()
    ----> 1 history =model.fit(embedding_matrix, 
          2                         y_train,
          3                         batch_size=512,
          4                         epochs=20,
          5                         verbose=1,
          6                         validation_split=0.2)
    

    File ~\anaconda3\lib\site-packages\keras\utils\traceback_utils.py:67, in filter_traceback.<locals>.error_handler(*args, **kwargs)
         65 except Exception as e:  # pylint: disable=broad-except
         66   filtered_tb = _process_traceback_frames(e.__traceback__)
    ---> 67   raise e.with_traceback(filtered_tb) from None
         68 finally:
         69   del filtered_tb
    

    File ~\AppData\Local\Temp\__autograph_generated_filev4piy4lw.py:15, in outer_factory.<locals>.inner_factory.<locals>.tf__train_function(iterator)
         13 try:
         14     do_return = True
    ---> 15     retval_ = ag__.converted_call(ag__.ld(step_function), (ag__.ld(self), ag__.ld(iterator)), None, fscope)
         16 except:
         17     do_return = False
    

    ValueError: in user code:
    
        File "C:\Users\javee\anaconda3\lib\site-packages\keras\engine\training.py", line 1051, in train_function  *
            return step_function(self, iterator)
        File "C:\Users\javee\anaconda3\lib\site-packages\keras\engine\training.py", line 1040, in step_function  **
            outputs = model.distribute_strategy.run(run_step, args=(data,))
        File "C:\Users\javee\anaconda3\lib\site-packages\keras\engine\training.py", line 1030, in run_step  **
            outputs = model.train_step(data)
        File "C:\Users\javee\anaconda3\lib\site-packages\keras\engine\training.py", line 889, in train_step
            y_pred = self(x, training=True)
        File "C:\Users\javee\anaconda3\lib\site-packages\keras\utils\traceback_utils.py", line 67, in error_handler
            raise e.with_traceback(filtered_tb) from None
        File "C:\Users\javee\anaconda3\lib\site-packages\keras\engine\input_spec.py", line 214, in assert_input_compatibility
            raise ValueError(f'Input {input_index} of layer "{layer_name}" '
    
        ValueError: Exception encountered when calling layer "sequential" (type Sequential).
        
        Input 0 of layer "batch_normalization" is incompatible with the layer: expected ndim=3, found ndim=4. Full shape received: (512, 20, 100, 128)
        
        Call arguments received by layer "sequential" (type Sequential):
          ??? inputs=tf.Tensor(shape=(512, 20, 100), dtype=float32)
          ??? training=True
          ??? mask=None
    



```python
y_preds =model.predict(x_test, batch_size=256)
```


```python

```
