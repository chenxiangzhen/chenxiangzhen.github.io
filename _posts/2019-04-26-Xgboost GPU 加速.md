```python
import xgboost as xgb
import numpy as np
from sklearn.datasets import fetch_covtype
from sklearn.model_selection import train_test_split
import time
```


```python
# Fetch dataset using sklearn
cov = fetch_covtype()
X = cov.data
y = cov.target
```


```python
# Create 0.75/0.25 train/test split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.25, train_size=0.75, random_state=42)
```


```python
# Specify sufficient boosting iterations to reach a minimum
num_round = 25 #3000

# Leave most parameters as default
param = {'objective': 'multi:softmax', # Specify multiclass classification
         'num_class': 8, # Number of possible output classes
         'tree_method': 'gpu_hist' # Use GPU accelerated algorithm
         }
```


```python
# Convert input data from numpy to XGBoost format
dtrain = xgb.DMatrix(X_train, label=y_train)
dtest = xgb.DMatrix(X_test, label=y_test)
```


```python
gpu_res = {} # Store accuracy result
tmp = time.time()
# Train model
param['tree_method'] = 'gpu_hist'
xgb.train(param, dtrain, num_round, evals=[(dtest, 'test')], evals_result=gpu_res)
print("GPU Training Time: %s seconds" % (str(time.time() - tmp)))
```

    [0]	test-merror:0.254804
    [1]	test-merror:0.247885
    [2]	test-merror:0.24427
    [3]	test-merror:0.240677
    [4]	test-merror:0.238474
    [5]	test-merror:0.234763
    [6]	test-merror:0.232147
    [7]	test-merror:0.229716
    [8]	test-merror:0.227162
    [9]	test-merror:0.224622
    [10]	test-merror:0.222632
    [11]	test-merror:0.220773
    [12]	test-merror:0.218453
    [13]	test-merror:0.215582
    [14]	test-merror:0.214605
    [15]	test-merror:0.212223
    [16]	test-merror:0.211176
    [17]	test-merror:0.209868
    [18]	test-merror:0.208622
    [19]	test-merror:0.205917
    [20]	test-merror:0.20434
    [21]	test-merror:0.203727
    [22]	test-merror:0.202591
    [23]	test-merror:0.201621
    [24]	test-merror:0.199817
    GPU Training Time: 4.505811929702759 seconds
    


```python
# Repeat for CPU algorithm
tmp = time.time()
param['tree_method'] = 'hist'
cpu_res = {}
xgb.train(param, dtrain, num_round, evals=[(dtest, 'test')], evals_result=cpu_res)
print("CPU Training Time: %s seconds" % (str(time.time() - tmp)))
```

    [0]	test-merror:0.254831
    [1]	test-merror:0.247912
    [2]	test-merror:0.244298
    [3]	test-merror:0.24069
    [4]	test-merror:0.238536
    [5]	test-merror:0.234804
    [6]	test-merror:0.232229
    [7]	test-merror:0.229703
    [8]	test-merror:0.227162
    [9]	test-merror:0.224519
    [10]	test-merror:0.222784
    [11]	test-merror:0.220705
    [12]	test-merror:0.21844
    [13]	test-merror:0.21676
    [14]	test-merror:0.214736
    [15]	test-merror:0.212257
    [16]	test-merror:0.210206
    [17]	test-merror:0.209345
    [18]	test-merror:0.207617
    [19]	test-merror:0.206102
    [20]	test-merror:0.205194
    [21]	test-merror:0.202798
    [22]	test-merror:0.202309
    [23]	test-merror:0.200554
    [24]	test-merror:0.199328
    CPU Training Time: 49.719186305999756 seconds