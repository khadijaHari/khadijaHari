{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Tree-Based Intelligent Intrusion Detection System in Internet of Vehicles \n",
   
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Import libraries"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 1,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "import warnings\n",
    "warnings.filterwarnings(\"ignore\")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "metadata": {},
   "outputs": [],
   "source": [
    "import numpy as np\n",
    "import pandas as pd\n",
    "import seaborn as sns\n",
    "import matplotlib.pyplot as plt\n",
    "from sklearn.preprocessing import LabelEncoder \n",
    "from sklearn.model_selection import train_test_split\n",
    "from sklearn.metrics import classification_report,confusion_matrix,accuracy_score,precision_recall_fscore_support\n",
    "from sklearn.metrics import f1_score\n",
    "from sklearn.ensemble import RandomForestClassifier,ExtraTreesClassifier\n",
    "from sklearn.tree import DecisionTreeClassifier\n",
    "import xgboost as xgb\n",
    "from xgboost import plot_importance"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Read the sampled CICIDS2017 dataset\n",
    "The CICIDS2017 dataset is publicly available at: https://www.unb.ca/cic/datasets/ids-2017.html  \n",
    "Due to the large size of this dataset, the sampled subsets of CICIDS2017 is used. The subsets are in the \"data\" folder.  \n",
    "If you want to use this code on other datasets (e.g., CAN-intrusion dataset), just change the dataset name and follow the same steps. The models in this code are generic models that can be used in any intrusion detection/network traffic datasets."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "metadata": {},
   "outputs": [],
   "source": [
    "#Read dataset\n",
    "df = pd.read_csv('./data/CICIDS2017.csv')\n",
    "# The results in this code is based on the original CICIDS2017 dataset. Please go to cell [10] if you work on the sampled dataset. "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Flow Duration</th>\n",
       "      <th>Total Fwd Packets</th>\n",
       "      <th>Total Backward Packets</th>\n",
       "      <th>Total Length of Fwd Packets</th>\n",
       "      <th>Total Length of Bwd Packets</th>\n",
       "      <th>Fwd Packet Length Max</th>\n",
       "      <th>Fwd Packet Length Min</th>\n",
       "      <th>Fwd Packet Length Mean</th>\n",
       "      <th>Fwd Packet Length Std</th>\n",
       "      <th>Bwd Packet Length Max</th>\n",
       "      <th>...</th>\n",
       "      <th>min_seg_size_forward</th>\n",
       "      <th>Active Mean</th>\n",
       "      <th>Active Std</th>\n",
       "      <th>Active Max</th>\n",
       "      <th>Active Min</th>\n",
       "      <th>Idle Mean</th>\n",
       "      <th>Idle Std</th>\n",
       "      <th>Idle Max</th>\n",
       "      <th>Idle Min</th>\n",
       "      <th>Label</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>3</td>\n",
       "      <td>2</td>\n",
       "      <td>0</td>\n",
       "      <td>12</td>\n",
       "      <td>0</td>\n",
       "      <td>6</td>\n",
       "      <td>6</td>\n",
       "      <td>6.0</td>\n",
       "      <td>0.00000</td>\n",
       "      <td>0</td>\n",
       "      <td>...</td>\n",
       "      <td>20</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>BENIGN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>109</td>\n",
       "      <td>1</td>\n",
       "      <td>1</td>\n",
       "      <td>6</td>\n",
       "      <td>6</td>\n",
       "      <td>6</td>\n",
       "      <td>6</td>\n",
       "      <td>6.0</td>\n",
       "      <td>0.00000</td>\n",
       "      <td>6</td>\n",
       "      <td>...</td>\n",
       "      <td>20</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>BENIGN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>52</td>\n",
       "      <td>1</td>\n",
       "      <td>1</td>\n",
       "      <td>6</td>\n",
       "      <td>6</td>\n",
       "      <td>6</td>\n",
       "      <td>6</td>\n",
       "      <td>6.0</td>\n",
       "      <td>0.00000</td>\n",
       "      <td>6</td>\n",
       "      <td>...</td>\n",
       "      <td>20</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>BENIGN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>34</td>\n",
       "      <td>1</td>\n",
       "      <td>1</td>\n",
       "      <td>6</td>\n",
       "      <td>6</td>\n",
       "      <td>6</td>\n",
       "      <td>6</td>\n",
       "      <td>6.0</td>\n",
       "      <td>0.00000</td>\n",
       "      <td>6</td>\n",
       "      <td>...</td>\n",
       "      <td>20</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>BENIGN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>3</td>\n",
       "      <td>2</td>\n",
       "      <td>0</td>\n",
       "      <td>12</td>\n",
       "      <td>0</td>\n",
       "      <td>6</td>\n",
       "      <td>6</td>\n",
       "      <td>6.0</td>\n",
       "      <td>0.00000</td>\n",
       "      <td>0</td>\n",
       "      <td>...</td>\n",
       "      <td>20</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>BENIGN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>...</th>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "      <td>...</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2830738</th>\n",
       "      <td>32215</td>\n",
       "      <td>4</td>\n",
       "      <td>2</td>\n",
       "      <td>112</td>\n",
       "      <td>152</td>\n",
       "      <td>28</td>\n",
       "      <td>28</td>\n",
       "      <td>28.0</td>\n",
       "      <td>0.00000</td>\n",
       "      <td>76</td>\n",
       "      <td>...</td>\n",
       "      <td>20</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>BENIGN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2830739</th>\n",
       "      <td>324</td>\n",
       "      <td>2</td>\n",
       "      <td>2</td>\n",
       "      <td>84</td>\n",
       "      <td>362</td>\n",
       "      <td>42</td>\n",
       "      <td>42</td>\n",
       "      <td>42.0</td>\n",
       "      <td>0.00000</td>\n",
       "      <td>181</td>\n",
       "      <td>...</td>\n",
       "      <td>20</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>BENIGN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2830740</th>\n",
       "      <td>82</td>\n",
       "      <td>2</td>\n",
       "      <td>1</td>\n",
       "      <td>31</td>\n",
       "      <td>6</td>\n",
       "      <td>31</td>\n",
       "      <td>0</td>\n",
       "      <td>15.5</td>\n",
       "      <td>21.92031</td>\n",
       "      <td>6</td>\n",
       "      <td>...</td>\n",
       "      <td>32</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>BENIGN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2830741</th>\n",
       "      <td>1048635</td>\n",
       "      <td>6</td>\n",
       "      <td>2</td>\n",
       "      <td>192</td>\n",
       "      <td>256</td>\n",
       "      <td>32</td>\n",
       "      <td>32</td>\n",
       "      <td>32.0</td>\n",
       "      <td>0.00000</td>\n",
       "      <td>128</td>\n",
       "      <td>...</td>\n",
       "      <td>20</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>BENIGN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2830742</th>\n",
       "      <td>94939</td>\n",
       "      <td>4</td>\n",
       "      <td>2</td>\n",
       "      <td>188</td>\n",
       "      <td>226</td>\n",
       "      <td>47</td>\n",
       "      <td>47</td>\n",
       "      <td>47.0</td>\n",
       "      <td>0.00000</td>\n",
       "      <td>113</td>\n",
       "      <td>...</td>\n",
       "      <td>20</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0.0</td>\n",
       "      <td>0</td>\n",
       "      <td>0</td>\n",
       "      <td>BENIGN</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "<p>2830743 rows × 78 columns</p>\n",
       "</div>"
      ],
      "text/plain": [
       "         Flow Duration  Total Fwd Packets  Total Backward Packets  \\\n",
       "0                    3                  2                       0   \n",
       "1                  109                  1                       1   \n",
       "2                   52                  1                       1   \n",
       "3                   34                  1                       1   \n",
       "4                    3                  2                       0   \n",
       "...                ...                ...                     ...   \n",
       "2830738          32215                  4                       2   \n",
       "2830739            324                  2                       2   \n",
       "2830740             82                  2                       1   \n",
       "2830741        1048635                  6                       2   \n",
       "2830742          94939                  4                       2   \n",
       "\n",
       "         Total Length of Fwd Packets  Total Length of Bwd Packets  \\\n",
       "0                                 12                            0   \n",
       "1                                  6                            6   \n",
       "2                                  6                            6   \n",
       "3                                  6                            6   \n",
       "4                                 12                            0   \n",
       "...                              ...                          ...   \n",
       "2830738                          112                          152   \n",
       "2830739                           84                          362   \n",
       "2830740                           31                            6   \n",
       "2830741                          192                          256   \n",
       "2830742                          188                          226   \n",
       "\n",
       "         Fwd Packet Length Max  Fwd Packet Length Min  Fwd Packet Length Mean  \\\n",
       "0                            6                      6                     6.0   \n",
       "1                            6                      6                     6.0   \n",
       "2                            6                      6                     6.0   \n",
       "3                            6                      6                     6.0   \n",
       "4                            6                      6                     6.0   \n",
       "...                        ...                    ...                     ...   \n",
       "2830738                     28                     28                    28.0   \n",
       "2830739                     42                     42                    42.0   \n",
       "2830740                     31                      0                    15.5   \n",
       "2830741                     32                     32                    32.0   \n",
       "2830742                     47                     47                    47.0   \n",
       "\n",
       "         Fwd Packet Length Std  Bwd Packet Length Max  ...  \\\n",
       "0                      0.00000                      0  ...   \n",
       "1                      0.00000                      6  ...   \n",
       "2                      0.00000                      6  ...   \n",
       "3                      0.00000                      6  ...   \n",
       "4                      0.00000                      0  ...   \n",
       "...                        ...                    ...  ...   \n",
       "2830738                0.00000                     76  ...   \n",
       "2830739                0.00000                    181  ...   \n",
       "2830740               21.92031                      6  ...   \n",
       "2830741                0.00000                    128  ...   \n",
       "2830742                0.00000                    113  ...   \n",
       "\n",
       "         min_seg_size_forward  Active Mean  Active Std  Active Max  \\\n",
       "0                          20          0.0         0.0           0   \n",
       "1                          20          0.0         0.0           0   \n",
       "2                          20          0.0         0.0           0   \n",
       "3                          20          0.0         0.0           0   \n",
       "4                          20          0.0         0.0           0   \n",
       "...                       ...          ...         ...         ...   \n",
       "2830738                    20          0.0         0.0           0   \n",
       "2830739                    20          0.0         0.0           0   \n",
       "2830740                    32          0.0         0.0           0   \n",
       "2830741                    20          0.0         0.0           0   \n",
       "2830742                    20          0.0         0.0           0   \n",
       "\n",
       "         Active Min  Idle Mean  Idle Std  Idle Max  Idle Min   Label  \n",
       "0                 0        0.0       0.0         0         0  BENIGN  \n",
       "1                 0        0.0       0.0         0         0  BENIGN  \n",
       "2                 0        0.0       0.0         0         0  BENIGN  \n",
       "3                 0        0.0       0.0         0         0  BENIGN  \n",
       "4                 0        0.0       0.0         0         0  BENIGN  \n",
       "...             ...        ...       ...       ...       ...     ...  \n",
       "2830738           0        0.0       0.0         0         0  BENIGN  \n",
       "2830739           0        0.0       0.0         0         0  BENIGN  \n",
       "2830740           0        0.0       0.0         0         0  BENIGN  \n",
       "2830741           0        0.0       0.0         0         0  BENIGN  \n",
       "2830742           0        0.0       0.0         0         0  BENIGN  \n",
       "\n",
       "[2830743 rows x 78 columns]"
      ]
     },
     "execution_count": 4,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "BENIGN          2273097\n",
       "DoS              380699\n",
       "PortScan         158930\n",
       "BruteForce        13835\n",
       "WebAttack          2180\n",
       "Bot                1966\n",
       "Infiltration         36\n",
       "Name: Label, dtype: int64"
      ]
     },
     "execution_count": 5,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "df.Label.value_counts()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Data sampling\n",
    "Due to the space limit of GitHub files, we sample a small-sized subset for model learning using random sampling"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Randomly sample instances from majority classes\n",
    "df_minor = df[(df['Label']=='WebAttack')|(df['Label']=='Bot')|(df['Label']=='Infiltration')]\n",
    "df_BENIGN = df[(df['Label']=='BENIGN')]\n",
    "df_BENIGN = df_BENIGN.sample(n=None, frac=0.01, replace=False, weights=None, random_state=None, axis=0)\n",
    "df_DoS = df[(df['Label']=='DoS')]\n",
    "df_DoS = df_DoS.sample(n=None, frac=0.05, replace=False, weights=None, random_state=None, axis=0)\n",
    "df_PortScan = df[(df['Label']=='PortScan')]\n",
    "df_PortScan = df_PortScan.sample(n=None, frac=0.05, replace=False, weights=None, random_state=None, axis=0)\n",
    "df_BruteForce = df[(df['Label']=='BruteForce')]\n",
    "df_BruteForce = df_BruteForce.sample(n=None, frac=0.2, replace=False, weights=None, random_state=None, axis=0)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 7,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_s = df_BENIGN.append(df_DoS).append(df_PortScan).append(df_BruteForce).append(df_minor)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "metadata": {},
   "outputs": [],
   "source": [
    "df_s = df_s.sort_index()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 9,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "# Save the sampled dataset\n",
    "df_s.to_csv('./data/CICIDS2017_sample.csv',index=0)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Preprocessing (normalization and padding values)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "df = pd.read_csv('./data/CICIDS2017_sample.csv')"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 11,
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "C:\\Users\\41364\\AppData\\Roaming\\Python\\Python35\\site-packages\\pandas\\compat\\_optional.py:106: UserWarning: Pandas requires version '2.6.2' or newer of 'numexpr' (version '2.6.1' currently installed).\n",
      "  warnings.warn(msg, UserWarning)\n"
     ]
    }
   ],
   "source": [
    "# Min-max normalization\n",
    "numeric_features = df.dtypes[df.dtypes != 'object'].index\n",
    "df[numeric_features] = df[numeric_features].apply(\n",
    "    lambda x: (x - x.min()) / (x.max()-x.min()))\n",
    "# Fill empty values by 0\n",
    "df = df.fillna(0)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### split train set and test set"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 18,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "labelencoder = LabelEncoder()\n",
    "df.iloc[:, -1] = labelencoder.fit_transform(df.iloc[:, -1])\n",
    "X = df.drop(['Label'],axis=1).values \n",
    "y = df.iloc[:, -1].values.reshape(-1,1)\n",
    "y=np.ravel(y)\n",
    "X_train, X_test, y_train, y_test = train_test_split(X,y, train_size = 0.8, test_size = 0.2, random_state = 0,stratify = y)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 19,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "(45328, 77)"
      ]
     },
     "execution_count": 19,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "X_train.shape"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 20,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "0    18184\n",
       "3    15228\n",
       "5     6357\n",
       "2     2213\n",
       "6     1744\n",
       "1     1573\n",
       "4       29\n",
       "dtype: int64"
      ]
     },
     "execution_count": 20,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "pd.Series(y_train).value_counts()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Oversampling by SMOTE"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 21,
   "metadata": {},
   "outputs": [],
   "source": [
    "from imblearn.over_sampling import SMOTE\n",
    "smote=SMOTE(n_jobs=-1,sampling_strategy={4:1500}) # Create 1500 samples for the minority class \"4\""
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 22,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "X_train, y_train = smote.fit_resample(X_train, y_train)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 23,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/plain": [
       "0    18184\n",
       "3    15228\n",
       "5     6357\n",
       "2     2213\n",
       "6     1744\n",
       "1     1573\n",
       "4     1500\n",
       "dtype: int64"
      ]
     },
     "execution_count": 23,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "pd.Series(y_train).value_counts()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {
    "collapsed": true
   },
   "source": [
    "## Machine learning model training"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Training four base learners: decision tree, random forest, extra trees, XGBoost"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 11,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy of DT: 0.9960292949792641\n",
      "Precision of DT: 0.9960126519428796\n",
      "Recall of DT: 0.9960292949792641\n",
      "F1-score of DT: 0.9960148981765187\n",
      "              precision    recall  f1-score   support\n",
      "\n",
      "           0       1.00      0.99      1.00      4547\n",
      "           1       0.99      0.98      0.98       393\n",
      "           2       0.99      1.00      1.00       554\n",
      "           3       1.00      1.00      1.00      3807\n",
      "           4       0.83      0.71      0.77         7\n",
      "           5       1.00      1.00      1.00      1589\n",
      "           6       0.99      0.99      0.99       436\n",
      "\n",
      "    accuracy                           1.00     11333\n",
      "   macro avg       0.97      0.95      0.96     11333\n",
      "weighted avg       1.00      1.00      1.00     11333\n",
      "\n"
     ]
    },
    {
     "data": {
      "image/png": "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\nAyutpPYrFrEuAAAAAElFTkSuQmCC\n",
      "text/plain": [
       "<Figure size 360x360 with 2 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "# Decision tree training and prediction\n",
    "dt = DecisionTreeClassifier(random_state = 0)\n",
    "dt.fit(X_train,y_train) \n",
    "dt_score=dt.score(X_test,y_test)\n",
    "y_predict=dt.predict(X_test)\n",
    "y_true=y_test\n",
    "print('Accuracy of DT: '+ str(dt_score))\n",
    "precision,recall,fscore,none= precision_recall_fscore_support(y_true, y_predict, average='weighted') \n",
    "print('Precision of DT: '+(str(precision)))\n",
    "print('Recall of DT: '+(str(recall)))\n",
    "print('F1-score of DT: '+(str(fscore)))\n",
    "print(classification_report(y_true,y_predict))\n",
    "cm=confusion_matrix(y_true,y_predict)\n",
    "f,ax=plt.subplots(figsize=(5,5))\n",
    "sns.heatmap(cm,annot=True,linewidth=0.5,linecolor=\"red\",fmt=\".0f\",ax=ax)\n",
    "plt.xlabel(\"y_pred\")\n",
    "plt.ylabel(\"y_true\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 12,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "dt_train=dt.predict(X_train)\n",
    "dt_test=dt.predict(X_test)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 13,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy of RF: 0.9924115415159269\n",
      "Precision of RF: 0.9924641723210192\n",
      "Recall of RF: 0.9924115415159269\n",
      "F1-score of RF: 0.9923720836357383\n",
      "              precision    recall  f1-score   support\n",
      "\n",
      "           0       0.99      0.99      0.99      4547\n",
      "           1       0.96      0.97      0.96       393\n",
      "           2       0.97      1.00      0.98       554\n",
      "           3       1.00      1.00      1.00      3807\n",
      "           4       0.83      0.71      0.77         7\n",
      "           5       1.00      1.00      1.00      1589\n",
      "           6       1.00      0.93      0.96       436\n",
      "\n",
      "    accuracy                           0.99     11333\n",
      "   macro avg       0.96      0.94      0.95     11333\n",
      "weighted avg       0.99      0.99      0.99     11333\n",
      "\n"
     ]
    },
    {
     "data": {
      "image/png": "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\nAAAASUVORK5CYII=\n",
      "text/plain": [
       "<Figure size 360x360 with 2 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "# Random Forest training and prediction\n",
    "rf = RandomForestClassifier(random_state = 0)\n",
    "rf.fit(X_train,y_train) \n",
    "rf_score=rf.score(X_test,y_test)\n",
    "y_predict=rf.predict(X_test)\n",
    "y_true=y_test\n",
    "print('Accuracy of RF: '+ str(rf_score))\n",
    "precision,recall,fscore,none= precision_recall_fscore_support(y_true, y_predict, average='weighted') \n",
    "print('Precision of RF: '+(str(precision)))\n",
    "print('Recall of RF: '+(str(recall)))\n",
    "print('F1-score of RF: '+(str(fscore)))\n",
    "print(classification_report(y_true,y_predict))\n",
    "cm=confusion_matrix(y_true,y_predict)\n",
    "f,ax=plt.subplots(figsize=(5,5))\n",
    "sns.heatmap(cm,annot=True,linewidth=0.5,linecolor=\"red\",fmt=\".0f\",ax=ax)\n",
    "plt.xlabel(\"y_pred\")\n",
    "plt.ylabel(\"y_true\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 14,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "rf_train=rf.predict(X_train)\n",
    "rf_test=rf.predict(X_test)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 15,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy of ET: 0.9920585899585282\n",
      "Precision of ET: 0.9920197520868573\n",
      "Recall of ET: 0.9920585899585282\n",
      "F1-score of ET: 0.9920217765183937\n",
      "              precision    recall  f1-score   support\n",
      "\n",
      "           0       0.99      0.99      0.99      4547\n",
      "           1       0.96      0.97      0.97       393\n",
      "           2       0.99      1.00      0.99       554\n",
      "           3       0.99      1.00      1.00      3807\n",
      "           4       0.80      0.57      0.67         7\n",
      "           5       1.00      1.00      1.00      1589\n",
      "           6       0.98      0.97      0.98       436\n",
      "\n",
      "    accuracy                           0.99     11333\n",
      "   macro avg       0.96      0.93      0.94     11333\n",
      "weighted avg       0.99      0.99      0.99     11333\n",
      "\n"
     ]
    },
    {
     "data": {
      "image/png": "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\nYII=\n",
      "text/plain": [
       "<Figure size 360x360 with 2 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "# Extra trees training and prediction\n",
    "et = ExtraTreesClassifier(random_state = 0)\n",
    "et.fit(X_train,y_train) \n",
    "et_score=et.score(X_test,y_test)\n",
    "y_predict=et.predict(X_test)\n",
    "y_true=y_test\n",
    "print('Accuracy of ET: '+ str(et_score))\n",
    "precision,recall,fscore,none= precision_recall_fscore_support(y_true, y_predict, average='weighted') \n",
    "print('Precision of ET: '+(str(precision)))\n",
    "print('Recall of ET: '+(str(recall)))\n",
    "print('F1-score of ET: '+(str(fscore)))\n",
    "print(classification_report(y_true,y_predict))\n",
    "cm=confusion_matrix(y_true,y_predict)\n",
    "f,ax=plt.subplots(figsize=(5,5))\n",
    "sns.heatmap(cm,annot=True,linewidth=0.5,linecolor=\"red\",fmt=\".0f\",ax=ax)\n",
    "plt.xlabel(\"y_pred\")\n",
    "plt.ylabel(\"y_true\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 16,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "et_train=et.predict(X_train)\n",
    "et_test=et.predict(X_test)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 17,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy of XGBoost: 0.9947939645283684\n",
      "Precision of XGBoost: 0.994785642359916\n",
      "Recall of XGBoost: 0.9947939645283684\n",
      "F1-score of XGBoost: 0.9947770558136226\n",
      "              precision    recall  f1-score   support\n",
      "\n",
      "           0       0.99      0.99      0.99      4547\n",
      "           1       0.99      0.97      0.98       393\n",
      "           2       1.00      1.00      1.00       554\n",
      "           3       0.99      1.00      1.00      3807\n",
      "           4       0.83      0.71      0.77         7\n",
      "           5       1.00      1.00      1.00      1589\n",
      "           6       1.00      0.98      0.99       436\n",
      "\n",
      "    accuracy                           0.99     11333\n",
      "   macro avg       0.97      0.95      0.96     11333\n",
      "weighted avg       0.99      0.99      0.99     11333\n",
      "\n"
     ]
    },
    {
     "data": {
      "image/png": "iVBORw0KGgoAAAANSUhEUgAAAUgAAAFBCAYAAAAVGzb7AAAABHNCSVQICAgIfAhkiAAAAAlwSFlz\nAAALEgAACxIB0t1+/AAAADl0RVh0U29mdHdhcmUAbWF0cGxvdGxpYiB2ZXJzaW9uIDMuMC4zLCBo\ndHRwOi8vbWF0cGxvdGxpYi5vcmcvnQurowAAIABJREFUeJzt3Xd8FHX+x/HXZ5OA0lV6ggKCioqI\nAopwAhZAKXKngp4oeCh3igpyghX94clZUUE9FRRpCgQb0gSkiHhSonSIEMpBIPQiRSXl8/tjJyFA\nNsnGndll83nymAe7s7P7/k6WfPhO+46oKsYYY07lC3cDjDEmUlmBNMaYAKxAGmNMAFYgjTEmACuQ\nxhgTgBVIY4wJwAqkMcYEYAXSGGMCsAJpjDEBxIa7AfkSsct8jAkHVSnK29L3bAz6dzauYu0iZXkh\nogtk+u4NYcuOq3Q+sXHVw5afkb497PlxYcpPT98OELb1z7D8sORGoogukMaY00xWZrhbEFJWII0x\noaNZ4W5BSFmBNMaETpYVSGOMyZNaD9IYYwKwHqQxxgRgPUhjjAnAjmIbY0wA1oM0xpgAbB+kMcbk\nzY5iG2NMINaDNMaYAKwHaYwxAUTZUezTbjzIzMxMbuveiwf7PQfA0y8Mps1t3bm1Wy9u7daL5HX+\nEYCmzJjDn+95gD/f8wB3/b0vyes35nzGgoVJtL/jPm7q/Dc+GJMY0vYlJFTnm5kTWbliHsuXzeHh\nh3qE9PMLo03rlqxeNZ/kNQvo36+X63kJCdWZNXMiK1bMY1mudT7rrApMnzaONasXMH3aOCpUKO96\nW4YPG8z21OUsWzrb9axAfD4fSxbPYNIXozzP9vq7P4VmBT9FsNOuQI6dOInaNc89Yd4/e/Xgs1Hv\n8Nmod7jogvMBiK9elZFvv8IXo9/lH93vZOArQwF/gX1h8Du8O/hffPXx+0z7Zh4bNv0vZO3LyMig\nX/+B1L+sJc2ad+CBB7pTr17dkH1+QXw+H0OHDKJ9h67Ub9CKLl06uZ6fkZFB//4DueyyljRv3oF/\nOOvcv38v5sxdwMWXNGfO3AX07+/+L+zo0Ym0a3+X6zn5eeTh+0hOXu95bji++2jnWoEUkYtE5HER\nGSoiQ5zH9f7IZ+7YtZv5/13MrR3aFLhsw/oXU75cWQAuu+Qidu7aA8DKtes4N6E6NeKrERcXx03X\nt2DOdwv/SLNObOOOXSxdtgqAw4ePkJy8nvjqVUP2+QVp0rghGzZsZtOmLaSnp5OYOImOhfh5/RF5\nrXP16lXp0KENY8ZMBGDMmIl07NjW1XYAfLdgEfv2H3A9J5D4+GrcfNP1jBgxzvPscHz3p8jKCn6K\nYK4USBF5HBgPCLAYWOI8HiciTxT1c18e8j59H+yByInNHvr+KP58zwO8POR9jh07dsr7Pp8yg+ZX\nNwJg1+49VK1cKee1KpUrsmv33qI2KV/nnZfA5Q0uZdHipa58fl6qx1dla+rxAU9Tt6VR3cMCnb3O\nixcvpUrliuzYsQvwF9HKlc7xrB3h8vrggTzx5AtkheEXP9zfPWCb2IXUA2isqi+p6lhneglo4rwW\ntHnfL+LssypwyUUnbjL0+ce9TB43nAkfDOHgL4f4cOzEE15f/ONyPp8yk74P/g0AzWNAeHFhwPfS\npUuROGE4fR97jkOHDoc+IADJY2U0r5V2QfY6/9PjdY4U7W6+gV279vDT0pVhyQ/nd5/DepCFkgXk\nNV58Nee1gESkp4gkiUjSB6OPb6YsXbGGeQsW0vrWbvR77iUW/7icxwe+QqWKZyMilChRgk7tWrNy\n7bqc9/ycsolnX3qTt156lgrlywH+HuOOXbtzltm5aw+VKoa2ZxMbG8vECcMZN+4Lvvxyekg/uyDb\nUtOokXD8R58QX420tJ2u58bGxpJ40jrv3LWHqlUrA1C1amXXeuqR4pprGtGhfWtS1i3k47H/oVWr\nZowaOdSz/HB997mpZgY9RTK3CmQfYLaITBeRYc70NTAb6J3fG1V1mKo2UtVG991zZ878Rx+4l9lf\njmXmZ6N4deATNLmyAS8/15/de/Zlv4858/9L3drnAZC2Yxd9nvoXLz7bj5rnJuR8zqUXXcCW1O2k\nbt9Beno602d/S6vmV4d05YcPG8za5BTeHDIspJ9bGEuSllGnTi1q1qxBXFwcnTvfwuQpM13PHT5s\nMMknrfOUyTO5++7bAbj77tuZPHmG6+0Ip6efeYmatRtR54Kruavrg8yd+z3duj/iWX64vvsTRNkm\ntivnQarq1yJyAf5N6nj8+x9TgSUa4v8yHh/4CvsPHERVubBubZ7r9zAA7370CQd/OcQLr70DQExM\nDIkjhhIbG8NTjz7A3/s+Q2ZmJn9u35o6TlENhWbXNOburrexYuUakpb4/3EOGPAS07+eE7KM/GRm\nZtK7zzNMm/oJMT4fI0dNYM2adQW/8Q9odk1juna9jZW51vmZAS/xyqvvMO6T97i3+51s3bqNO+78\nu6vtABg75h1aXNuUihXPZvPGJAY+/xofjRzvem4kCMd3f4oI32QOlni+jyIIRbmFZKjYXQ3trobF\nOr+It3397ccvg/6dPePKTnbbV2NMMRBlV9JYgTTGhE6E71MMlhVIY0zoRNk+SCuQxpjQibIe5Gl3\nLbYxJoK5eKK4iMSIyFIRmeI8ryUii0RkvYhMEJESzvySzvMU5/WauT7jSWf+zyJS4HWYViCNMaHj\n7pU0vYG1uZ6/DLyhqnWB/Ry/Sq8HsF9V6wBvOMshIhcDdwCXAG2B/4hITH6BViCNMSHj1pU0IpIA\ntAM+cJ4LcB3wqbPIKKCT8/gW5znO69c7y98CjFfV31V1E5CC/1ztgKxAGmNCpwg9yNyXFztTzzw+\n+U2gP8cvVT4HOKCqGc7zVPwXpeD8vRXAef2gs3zO/Dzekyc7SGOMCZ0iHKRR1WFAwOtyRaQ9sEtV\nfxSRltmz8/qoAl7L7z15sgJpjIl0zYCOInIzcAZQDn+PsoKIxDq9xAQge6y3VKAGkCoisUB5YF+u\n+dlyvydPtoltjAkdFw7SqOqTqpqgqjXxH2SZo6p3AXOB25zFugGTnMdfOc9xXp+j/muqvwLucI5y\n1wLq4h+vNiDrQRpjQsfb8yAfB8aLyAvAUuBDZ/6HwBgRScHfc7wDQFVXi0gisAbIAHoVNHiOFUhj\nTOi4fCWNqs4D5jmPN5LHUWhV/Q24PcD7BwGDCptnBdIYEzpRdiVNRBfIuErnhzU/e9ip4pqfXszX\nv7jnF4ldi22MMQFYgfROuAeMrXX2ZWHL37RvRdjXv1gPGGv5RWOb2MYYE4D1II0xJgDrQRpjTADW\ngzTGmACsB2mMMQFYD9IYYwKwAmmMMQFo2G5l7workMaY0LEepDHGBGAF0hhjAoiyo9g2YK4xxgRg\nPUhjTOjYJrYxxgQQZUexo2ITe/iwwWxPXc6ypbNz5t16a3uWL5vDsd+2cuUVoR+Vp0TJEnw562Om\nfZvIjO8/p8/jDwBwzbVNmDxnPFPnTSBx6kjOq+W/R1CPB+5m5n8/Z/r8iYz9YhjxCdVC3qZsbVq3\nZPWq+SSvWUD/fr1cy7H8yMqOhHw37kkTTlFRIEePTqRd+7tOmLd6dTK3d76f775b6Ermsd+P8ddO\n93Fzi860a9GZFtc34/JG9Xnh1Wfo848nadeyC199No2H/nm/vz0rk+l4/V+56drbmf7VLJ74v0dd\naZfP52PokEG079CV+g1a0aVLJ+rVq+tKluVHTnYk5ANWICPRdwsWsW//gRPmJSensG7dBldzjx75\nFYDYuFhiY2NBQVHKli0DQNlyZdi5YzcACxcs4bdffwNgadJKqlav7EqbmjRuyIYNm9m0aQvp6ekk\nJk6iY4c2rmRZfuRkR0I+4D+KHewUwcJSIEXk3nDkhprP52PqvAkkJc9lwbcLWfbjSp7o/X+MGP82\n/105kz93bs97Q0ac8r4uXf/Mt7O/d6VN1eOrsjX1+ICnqdvSqF69qitZlh852ZGQD6BZGvQUycLV\ngxwY6AUR6SkiSSKSlJV1xMs2BS0rK4t2LbvQtH5rGjS8lAsuqsPfHribv93xENfUb82nn0zimX89\ndsJ7Ot3ejvqXX8ywt0a60iYROWWeerjjvDjnF+d1zxFlm9iuHcUWkRWBXgKqBHqfqg4DhgHEloiP\n7P9eHId+OcTC75fQ8oZm1LvkApb9uBKAKV/MYOTE/+Qs16zFVfTqex93dOjBsWPprrRlW2oaNRKO\nD9WfEF+NtLSdrmRZfuRkR0I+EPGbzMFyswdZBbgH6JDHtNfFXE+cfc5ZlC1XFoCSZ5SkeYurSVm3\nibLlylDr/PMAaN6yKSnrNgFwcf2LGDR4APff1Zu9e/a51q4lScuoU6cWNWvWIC4ujs6db2HylJmu\n5Vl+ZGRHQj4AWRr8FMHcPA9yClBGVZed/IKIzAtl0Ngx79Di2qZUrHg2mzcmMfD519i3/wBD3niB\nSpXO5qtJo1m+fDU3n3Sk+4+oXKUir73zAjExPsTnY+qXM5kzcz5PPvo8/xk5GM3K4uCBX+j/yHMA\nPDnwUUqXLsU7I14FYHvqDu7v2jtk7cmWmZlJ7z7PMG3qJ8T4fIwcNYE1a9aFPMfyIys7EvKBiN9k\nDpZ4vo8iCOHcxLa7GtpdDYt1vuqpOzQL4eiQfwT9O1uq93tFyvKCXUljjAmdCO5wFYUVSGNM6ETZ\nJrYVSGNM6ET4QZdgWYE0xoROlJ3mYwXSGBM6UdaDjIprsY0xxg3WgzTGhIzaQRpjjAkgyjaxrUAa\nY0LHDtIYY0wA1oM0xpgAbB+kMcYEYD1IY4wJwPZBGmNMANaD9E72sE/hsmlfoEHRvRHu9bf84p1f\nFHYepIeK63iI2fkNqjQNW/7ynT8U7/EQLb9orAdpjDEBWIE0xpgA7CCNMcYEYD1IY4zJm1qBNMaY\nAKxAGmNMAFF2mo8NmGuMMQFYD9IYEzq2iW2MMQFEWYG0TWxjTMioatBTQUTkDBFZLCLLRWS1iAx0\n5tcSkUUisl5EJohICWd+Sed5ivN6zVyf9aQz/2cRaVNQthVIY0zoZGnwU8F+B65T1QbA5UBbEbka\neBl4Q1XrAvuBHs7yPYD9qloHeMNZDhG5GLgDuARoC/xHRGLyC7YCaYwJHRcKpPoddp7GOZMC1wGf\nOvNHAZ2cx7c4z3Fev15ExJk/XlV/V9VNQArQJL9sK5DGmJDRLA16EpGeIpKUa+p58ueKSIyILAN2\nAbOADcABVc1wFkkF4p3H8cBWAOf1g8A5uefn8Z482UEaY0zoFOEgjaoOA4YVsEwmcLmIVAC+AOrl\ntZjztwR4LdD8gKKuBzl82GC2py5n2dLZYWtDm9YtWb1qPslrFtC/Xy/XcqYt+YxP545hwjcj+WTG\nhwD847EezFo6iQnfjGTCNyNpfr1/yLSrr23MuBkj+HTuGMbNGEGTZle61i6v1j8S88P97y/cP3uy\nijAFQVUPAPOAq4EKIpLdyUsAssdpSwVqADivlwf25Z6fx3vyFHUFcvToRNq1vyts+T6fj6FDBtG+\nQ1fqN2hFly6dqFevrmt59936EF1u6M5f2/TImTdm2Hi63NCdLjd0Z8HsHwA4sO8gj9zTn9ta3c2A\n3i8w6O1nXWmP1+sfafnh/PcX7nWHom1iF0REKjk9R0TkTOAGYC0wF7jNWawbMMl5/JXzHOf1Oeo/\nXP4VcIdzlLsWUBdYnF+2awVSRC4SketFpMxJ89u6lQnw3YJF7Nt/wM2IfDVp3JANGzazadMW0tPT\nSUycRMcOBZ5N4LrkVevYvXMPACnJGylRsgRxJeJCnhPu9Q93fjj//YV73QG3jmJXA+aKyApgCTBL\nVacAjwN9RSQF/z7GD53lPwTOceb3BZ4AUNXVQCKwBvga6OVsugfkSoEUkUfwV/OHgVUickuul//t\nRmakqB5fla2px3vtqdvSqF69qjthqrw3/k3GzRjBrV2P/4jv+NttTJwzmoFvPEXZ8mVPedsN7VuR\nvGod6cfSQ94kT9c/AvPDKSLW3YVNbFVdoaoNVfUyVb1UVZ935m9U1SaqWkdVb1fV3535vznP6ziv\nb8z1WYNU9XxVvVBVpxeU7dZBmvuBK1X1sHOS5qciUlNVh5D3jtIczhGsngASUx6fr7RLTXSH/2yC\nExXmZNii6NbhH+zeuYezK57FexPeZFPK/0gc+TnDXv8IVaXX4z157P8e5rlHj/+fdP6FtejzzIP8\no0sfV9rk5fpHYn44RcK6R9twZ25tYsdkn7ekqpuBlsBNIvI6BRRIVR2mqo1UtdHpVhwBtqWmUSPh\n+L1EEuKrkZa205Ws7E3mfXv2M2f6fC5tWI99e/aTlZWFqvL5x5O4tOHFOctXrlaJN0a8yDMPP0/q\n/7a50iYv1z8S88MpItbd5YM0XnOrQO4QkcuznzjFsj1QEajvUmZEWJK0jDp1alGzZg3i4uLo3PkW\nJk+ZGfKcM0udQanSpXIeN23RhJTkjVSsfE7OMtfd1IKUZP/WRdlyZXh77GsM+fd7LFuyMuTtyebV\n+kdqfjhFwrq7cZAmnNzaxL4HyMg9wzlh8x4Red+lTADGjnmHFtc2pWLFs9m8MYmBz7/GRyPHuxl5\ngszMTHr3eYZpUz8hxudj5KgJrFmzLuQ5Z1c8mzc+ehGA2NgYpn0+i//OXcSgt57lwkvroqps35rG\nv/q9Avj3S55bK4Gej3an56PdAXjgjkfZt2d/SNvl1fpHan44//2Fe92BiO8RBksief9MbIn4sDXO\nbvtqt30t1vmq+e4KC2RvhxZB/86eM/nbImV5IerOgzTGmFCxSw2NMaETZZvYViCNMSETZbfFtgJp\njAkhK5DGGJM360EaY0wAViCNMSYAK5DGGBNI0U6fjFhWII0xIWM9SGOMCUCzrAdpjDF5sh6kMcYE\nUMRLuCOWFUhjTMhYD9IYYwKItn2QET3cGSIR3DhjolgRt5W3NLo+6N/Zc5NmR2xVjegeZLjHYyzu\n+b9+/mJYss/8y5NAMR+PMQLyiyLaepARXSCNMaeXaCuQNmCuMcYEYD1IY0zIRPIhjaKwAmmMCZlo\n28S2AmmMCZloO1G8wH2QIlJFRD4UkenO84tFpIf7TTPGnG40K/gpkhXmIM1IYAaQfc7BOqCPWw0y\nxpy+slSCniJZYQpkRVVNxLnbhKpmAJmutsoYc1pSlaCnSFaYfZBHROQcQAFE5GrgoKutMsaclorj\nQZq+wFfA+SLyPVAJuM3VVhljTkvF7jQfVf1JRFoAFwIC/Kyq6a63zBhz2il2PUgRueekWVeICKo6\n2qU2GWNOU5F+0CVYhdnEbpzr8RnA9cBPgBVIY8wJIv2gS7AKs4n9cO7nIlIeGONai4wxp61o2wdZ\nlMEqjgJ1Q92QUCpfvhwTxg9j1cpvWbliHldfdaWn+W1at2T1qvkkr1lA/369PM12K//39Azuensy\nnd/8kr+8/gX/mbUUgEUp27lj6CQ6D5lE93ensmXPLwAcy8ik/ydz6fDqp3R9ZzLb9h0C4MCR37hv\n2HSaPjuGFyf9EJK2nSycP/9o/O6DEW3nQRZmH+RknFN88BfUi4FENxv1R73x+vPMmDGXLnf0JC4u\njlKlzvQs2+fzMXTIINrefCepqWks/GEak6fMZO3a9ad1fonYGIbf35ZSJeNIz8zi3vem0vzCeAZ9\n+QNv3nM9tStXYMIPaxk+Zzn/6vwnvliyjnJnlmRyv9v4evlGhnydxCt/bUXJuBh6tb6ClB37Sdm5\nP0RrfVw4f/7R+t0HI9o2sQvTg3wNGOxMLwLXquoTBb1JRJqISGPn8cUi0ldEbv5DrS2EsmXL8Kfm\nVzHio3EApKenc/DgL27H5mjSuCEbNmxm06YtpKenk5g4iY4d2pz2+SJCqZJxAGRkZpGRmYUgCHDk\nN/9JDYd/S6dSuVIAzFuzhQ5X1AHghktrsjglDVXlzBJxNKxZhRKxMX+4TXkJ588/Wr/7YKgGP0Wy\nfHuQIhIDDFDVG4L5UBF5DrgJiBWRWcBVwDzgCRFpqKqDitjeAtWufR579uzlww/e4LLLLuann1bw\naN9nOXr0V7ciT1A9vipbU4+PyJy6LY0mjRt6ku12fmZWFne+NZmte3+hS9OLqH9uJZ67tRkPjZxF\nydgYypwRx+gH2wOw65ejVK1QGoDYGB9lzijBgaO/c1bpM0LSlkDC+fOP5u++sCJ9kzlY+fYgVTUT\nOOocmAnGbUAz4FqgF9BJVZ8H2gBd8nujiPQUkSQRScrKOhJkLMTGxNCwYX3ef380jZu04ciRozze\n/6GgP6eoRE79B+LlfX/czI/x+UjsfQsznuzMqq17SNmxn7ELVvN29xuZ+VQXOl5Zl8FTFjuZebQt\nJK3IXzh//tH83RdWtF1qWJhN7N+Alc6IPkOzpwLek6Gqmap6FNigqr8AqOqvONd0B6Kqw1S1kao2\n8vlKF2olckvdlkZqahqLl/gPInz++VQaXl4/6M8pqm2padRIOH4vkYT4aqSl7Yyq/HJnlqRR7aos\n+DmVdWn7qX9uJQDaNKjF8i27AKhSvhQ7Dvj/g8vIzOLwb8coX6pkSNuRl3D+/IvDd1/cFKZATgUG\nAPOBH50pqYD3HBORUs7jnEPITk/U1QGOdu7cTWrqdi644HwArruuOWvXrnMz8gRLkpZRp04tatas\nQVxcHJ0738LkKTNP+/x9h3/jl19/B+C39AwWpaRRu3IFDv92jP/t9l+av3D9dmpVqgBAi4vPZfJP\nKQB8s2ozjc+vlmcPJ9TC+fOP1u8+GMXuKDZQQVWH5J4hIr0LeM+1qvo7gOoJI77FAd2Ca2Lwej86\ngNGj3qJEiTg2bdpCj/v6uh2ZIzMzk959nmHa1E+I8fkYOWoCa9Z4V6Ddyt9z6CgDEr8jS5UsVVrX\nr8W19Wrw7F+a8c+xc/CJUPbMkgy8rTkAf25Ul6cTv6PDq59S7sySvHxny5zPuumliRz5/RjpmVnM\nXb2Fd3u04fwqFf5wGyG8P/9o/e6DEeHHXIJW4H2xReQnVb3ipHlLVdX1vb+xJeLD9vOOhNuuhjvf\nbvtajPOLuHPwv9VuDfp39pq0zyK2GxmwBykidwJ/BWqJyFe5XioL7HW7YcaY00+kH3QJVn6b2P8F\n0oCK+M+BzHYIWOFmo4wxp6cIv4NC0AIWSFX9H/A/oGl+HyAiP6hqvssYY4oH9eRkLu+E4q6G7p75\na4w5bWRF2VGaUBTIKPuRGGOKKst6kMYYk7do28QuzH2xHxKRs/JbJITtMcacxrKKMEWywlxJUxVY\nIiKJItJWTr0c4m4X2mWMOQ0pEvRUEBGpISJzRWStiKzOvlBFRM4WkVkist75+yxnvjiXRKeIyAoR\nuSLXZ3Vzll8vIgVetFJggVTVZ/APkPsh0B1YLyL/FpHznddXFbiGxphiwaUeZAbwT1WtB1wN9BKR\ni4EngNmqWheY7TwH/0hidZ2pJ/Au+Asq8Bz+0cWaAM8VsHVcuBHF1X+5zQ5nygDOAj4VkVcKt37G\nmOLAjQKpqmmq+pPz+BCwFogHbgFGOYuNAjo5j28BRqvfQqCCiFTDP5rYLFXdp6r7gVlA2/yyCzOi\n+CP4r5/eA3wA9FPVdBHxAeuB/oVYR2NMMeD2QRoRqQk0BBYBVVQ1DfxFVEQqO4vFA1tzvS3VmRdo\nfkCFOYpdEfiLc+J4DlXNEpH2hXi/MaaYKMptsUWkJ/5N4WzDVHVYHsuVAT4D+qjqL/mMDpXXC5rP\n/IAKc1fDZ/N5bW1B7zfGFB9FOQ/SKYanFMTcRCQOf3H8WFU/d2bvFJFqTu+xGrDLmZ8K1Mj19gRg\nuzO/5Unz5+WXW5S7GhpjjGecM2c+BNaq6uu5XvqK48MndgMm5Zp/j3M0+2rgoLMpPgNoLSJnOQdn\nWjvzAmd7PSR7UEQiuHHGRLEiDsvzZdW/Bv0722nHJ/lmiUhz4DtgJceP6zyFfz9kInAusAW4XVX3\nOQX1bfwHYI4C96pqkvNZf3PeCzBIVT/KNzuSC6SNB1k88yNiPMTinl/EAvl5EQrkXwookOFklxoa\nY0Imy4PbanjJCqQxJmQid3u0aKxAGmNCJtKvrQ6WFUhjTMgU5TzISGYF0hgTMjYepDHGBGD7II0x\nJgDbxDbGmADsII0xxgRgm9jGGBOAbWIbY0wAtoltjDEBWIE0xpgAijbEReSKuvEgExKq883Miaxc\nMY/ly+bw8EM9PG9Dm9YtWb1qPslrFtC/Xy/L91jKuoUs/ekbkpbMZOEP0zzNDve6hzs/2m77GnU9\nyIyMDPr1H8jSZasoU6Y0ixd9zTez57N27XpP8n0+H0OHDKLtzXeSmprGwh+mMXnKTMv3KD/bDTfe\nzt69+z3NDPe6hzs/GkVdD3LHjl0sXea/E+3hw0dITl5PfPWqnuU3adyQDRs2s2nTFtLT00lMnETH\nDm0svxgI97qHOx+irwfpWYEUkdFeZWU777wELm9wKYsWL/Uss3p8Vbambs95nrotjeoeFujing+g\nqkyfNo5FC6dzX4+7PMsN97qHOx/850EGO0UyVzaxReSrk2cBrUSkAoCqdnQjN7fSpUuROGE4fR97\njkOHDrsdlyOvO615OWp7cc8HuLZlJ9LSdlKp0jl8PX08P/+cwncLFrmeG+51D3c+2HmQhZUArMF/\nH+3s2y02AgYX9Mbct4CUmPL4fKWDDo+NjWXihOGMG/cFX345Pej3/xHbUtOokXB8qPyE+Gqkpe20\nfA9l5+3evZdJk6bTuPHlnhTIcK97uPMh8jeZg+XWJnYj4Efgafx3FJsH/Kqq36rqt/m9UVWHqWoj\nVW1UlOIIMHzYYNYmp/DmkHzvJOmKJUnLqFOnFjVr1iAuLo7OnW9h8pSZlu+RUqXOpEyZ0jmPb7yh\nBatX/+xJdrjXPdz5EH37IF3pQapqFvCGiEx0/t7pVtbJml3TmLu73saKlWtIWuL/xzFgwEtM/3qO\nF/FkZmbSu88zTJv6CTE+HyNHTWDNmnWeZFs+VKlSiU8nfghAbGwM48d/yYyZ8zzJDve6hzsfIn+f\nYrA8uauhiLQDmqnqUwUunIvd1bB45kfEXf2Ke34R72r4ynldg/6d7f+/sRG759KTXp2qTgWmepFl\njAmfSN9kDlbUnShujAmfaNsrKseFAAASIUlEQVTEtgJpjAmZrCgrkVYgjTEhY5vYxhgTQHT1H61A\nGmNCyHqQxhgTgF1qaIwxAdhBGmOMCSC6ymMUjgdpjDGhYj1IY0zI2EEaY4wJwPZBGmNMANFVHq1A\nGmNCyDaxPZQ97JPlW77lnx5sE9sYYwKIrvIY4QWyuA4YW9zzI2LAWOC1Gt7dETG3x7Z+DIR//YvC\nNrGNMSYAjbI+pBVIY0zIWA/SGGMCsIM0xhgTQHSVRyuQxpgQsh6kMcYEYPsgjTEmADuKbYwxAVgP\n0hhjAoi2HqQNmGuMMQFYD9IYEzK2iW2MMQFkqW1iG2NMnrQIU0FEZISI7BKRVbnmnS0is0RkvfP3\nWc58EZGhIpIiIitE5Ipc7+nmLL9eRLoVZn2irkAOHzaY7anLWbZ0dtja0KZ1S1avmk/ymgX079er\nWOUnJFTnm5kTWbliHsuXzeHhh3p4mg/urH+bV+/nwZ/eofusF3PmXfPoX/j74qHcM30Q90wfRK1W\nDQDwxcZw0+t/p9vMF7l39ss06dUh5z0ly5Wi43uPcO+cV7h39stUu6JOSNqX084w/9vLQoOeCmEk\n0PakeU8As1W1LjDbeQ5wE1DXmXoC74K/oALPAVcBTYDnsotqfqKuQI4enUi79uEZpgrA5/MxdMgg\n2nfoSv0GrejSpRP16tUtNvkZGRn06z+Q+pe1pFnzDjzwQPeoWP/VE+fz6T2vnjL/xw++ZvRNTzP6\npqfZNHc5ABe0a0JMiVhGtX6SMe0G0OCv11EuoSIA1/3f3Wyat4KPruvPqLZPsS8ldIPihvu7B/9R\n7GD/FPiZqvOBfSfNvgUY5TweBXTKNX+0+i0EKohINaANMEtV96nqfmAWpxbdU0RdgfxuwSL27T8Q\ntvwmjRuyYcNmNm3aQnp6OomJk+jYoU2xyd+xYxdLl/m3hA4fPkJy8nriq1f1LN+t9U9d/DO/HThc\nuIUV4kqVRGJ8xJ5Rgsz0DI4d+pUSZc4kocmFrBw/D4Cs9Ex+/+XoH25btnB/9+A/SBPsVERVVDUN\nwPm7sjM/Htiaa7lUZ16g+fnypECKSHMR6Ssirb3IC6fq8VXZmnq8V5C6LY3qHhaIcOfndt55CVze\n4FIWLV7qWabX69+w2410m/Fv2rx6PyXLlwJg3bTFpB/9nQeS3ubvC98kadg0fjt4hPLnVuLovkO0\nHdyTu6e9QOuX7yPuzJIha0skfPdF2cQWkZ4ikpRr6vkHmiB5zNN85ufLlQIpIotzPb4feBsoi3+7\n/4mAb4wCIqd+D+rhkb1w52crXboUiROG0/ex5zh0qJA9rxDwcv2XjfmGD/7Ul1Ftn+bIrgO0fMa/\na6fq5bXJyszivcYPM7xZXxrdfzPlz62ELzaGKpfWZNmY2Yy5+RnSf/2dJg92KCCl8CLhuy/KJraq\nDlPVRrmmYYWI2ulsOuP8vcuZnwrUyLVcArA9n/n5cqsHGZfrcU/gRlUdCLQG8t1BmPt/k6ysIy41\nzz3bUtOokXB8qPyE+Gqkpe0sNvkAsbGxTJwwnHHjvuDLL6d7mu3l+h/d8wuapaDKinFzqXZ5bQDq\n3XINm79dQVZGJkf3/sK2pHVUvaw2h9L2cShtHzuWbQD8Pc0ql9YMWXsi4bv3cBP7KyD7SHQ3YFKu\n+fc4R7OvBg46m+AzgNYicpZzcKa1My9fbhVIn9OQcwBR1d0AqnoEyMjvjbn/N/H5SrvUPPcsSVpG\nnTq1qFmzBnFxcXTufAuTp8wsNvngP5NgbXIKbw4pTEcgtLxc/9KVK+Q8rtumEXt+TgXg0Pa9nHvN\nJQDEnVmS6lfUYW/Kdo7uPsihtH2cVbsaAOc1u4S967eFrD2R8N2ratBTQURkHPADcKGIpIpID+Al\n4EYRWQ/c6DwHmAZsBFKA4cCDTrv2Af8CljjT8868fLl1onh54Ef82/0qIlVVdYeIlCHvfQEhM3bM\nO7S4tikVK57N5o1JDHz+NT4aOd7NyBNkZmbSu88zTJv6CTE+HyNHTWDNmnXFJr/ZNY25u+ttrFi5\nhqQl/l/OAQNeYvrXczzJd2v9273VixpN63HmWWX4+6KhfP/6Z9RoWo/KF58HqhxM3cOsJ0cAsHTU\nLNoO7kn3b15CRFiVOJ89yf7jA7OfHUW7oQ8QExfLgS27+Pqx0P0nEu7vHtwZD1JV7wzw0vV5LKtA\nnuc3qeoIYEQw2eLx/rFS+I8+bSrM8rEl4sN2Wn5xvqtguPPtroYRcFdD1SJ1ZDqc2z7o39nJW6a4\n2mn6Izy91FBVjwKFKo7GmNNPtI3mY9diG2NCxm65YIwxAYTjlDI3WYE0xoSMDXdmjDEBRNs+yKi7\nFtsYY0LFepDGmJCxgzTGGBOAHaQxxpgArAdpjDEBRNtBGiuQxpiQibabdlmBNMaETHSVRyuQxpgQ\nsn2QxhgTQLQVSE+HOwuaSAQ3zpgoVsThzq6u3jLo39mF2+fZcGfGmOgXbT3IiC6QxXXA2OKeHykD\n5oY7v27FK8KSv37PT0V+r53mY4wxAUT0LrsisAJpjAkZ28Q2xpgArAdpjDEBWA/SGGMCiLaDNDZg\nrjHGBGA9SGNMyNhgFcYYE0C0bWJbgTTGhIz1II0xJgDrQRpjTADWgzTGmACsB2mMMQFEWw8yKs+D\nLF++HBPGD2PVym9ZuWIeV191paf5bVq3ZPWq+SSvWUD/fr08zQ53fkJCdb6ZOZGVK+axfNkcHn6o\nh6f5EN719zLb5/Mxac7HDPv4TQAGv/sCM374jKnzJ/DikGeJjfX3f8qULcP7Y9/gq7njmPZdIrfe\n2cG1NmkR/kSyqCyQb7z+PDNmzOXS+i244sobWZu83rNsn8/H0CGDaN+hK/UbtKJLl07Uq1e32ORn\nZGTQr/9A6l/WkmbNO/DAA92Lzfp7nd2t551sWLc55/lXn02nTdNbaXdtF844oySdu3YCoGuP20n5\neSMdW91J1049eWLgo8TFubPxqJoV9BTJXCmQInKViJRzHp8pIgNFZLKIvCwi5d3IzFa2bBn+1Pwq\nRnw0DoD09HQOHvzFzcgTNGnckA0bNrNp0xbS09NJTJxExw5tik3+jh27WLpsFQCHDx8hOXk98dWr\nepYfzvX3Mrtqtcq0vLE5iWO/zJn37Tff5zxe/tNqqlSvDIAqlC5TGoBSpUtx8MAvZGRkutKuLDTo\nKZK51YMcARx1Hg8BygMvO/M+cikTgNq1z2PPnr18+MEbLFk8g/ffe5VSpc50M/IE1eOrsjV1e87z\n1G1pVPewQIQ7P7fzzkvg8gaXsmjxUs8yw7n+XmY/PeifvDJwCFlZp/bAYmNj6dS5Hd/N+S8AYz+Y\nwPkX1OL7VTOYMn8CLzz9mmuj7qhq0FMkc6tA+lQ1w3ncSFX7qOoCVR0I1M7vjSLSU0SSRCQpK+tI\n0MGxMTE0bFif998fTeMmbThy5CiP93+oCKtQNCKn3l7Dy38E4c7PVrp0KRInDKfvY89x6NBhz3LD\nuf5eZbe68U/s3b2f1SuS83z9/155giU//ETSwmUA/Om6pqxd9TPNLm1Dx1Z38uyL/Snj9ChDzXqQ\nhbNKRO51Hi8XkUYAInIBkJ7fG1V1mKo2UtVGPl/wX2LqtjRSU9NYvMTfa/n886k0vLx+0J9TVNtS\n06iRcHyo/oT4aqSl7Sw2+eDvwUycMJxx477gyy+ne5odzvX3KvuKqxpwfdtrmfvjZN4c/m+ubt6Y\n1/7zLwAeeux+zj7nLP494PWc5W+9syMzp84BYMumVFK3bKd23ZohbxdYD7Kw7gNaiMgG4GLgBxHZ\nCAx3XnPNzp27SU3dzgUXnA/Addc1Z+3adW5GnmBJ0jLq1KlFzZo1iIuLo3PnW5g8ZWaxyQcYPmww\na5NTeHPIME9zIbzr71X24Bfe5k8NbqbVlR3oc/9TLFywhMceHMDtXTvxp1ZNefTvT51QeLan7qDp\nn5oAcE6ls6lV5zy2/m9byNsF/tN8gp0imSuHslT1INBdRMri36SOBVJV1ZP/yns/OoDRo96iRIk4\nNm3aQo/7+noRC0BmZia9+zzDtKmfEOPzMXLUBNas8a5Ahzu/2TWNubvrbaxYuYakJf7iMGDAS0z/\neo4n+eFc/3D/7J9/9Um2b93BxOn+3fwzp8zl7cHDeWfwcF5+ayBTvp2ACLz6/FD27zvgShsi/bSd\nYEX0fbFjS8SHrXHF+a6C4c6PlLsKhjs/rHc1LOJ9sauUvyjo39mdB5Mj9r7YUXkepDHGhIJdamiM\nCZlIPyodLCuQxpiQieRddkVhBdIYEzKRflQ6WFYgjTEhYz1IY4wJwPZBGmNMANaDNMaYAGwfpDHG\nBBBtV9JYgTTGhIz1II0xJoBo2wdplxoaY0LGrXvSiEhbEflZRFJE5AmXVyOH9SCNMSHjRg9SRGKA\nd4AbgVRgiYh8paprQh52EutBGmNCxqUBc5sAKaq6UVWPAeOBW1xdEUdE9yCzh32yfMsvjvnr9/wU\n1vyicGkPZDywNdfzVOAqd6JOFNEFsqhj0mUTkZ6q6v2w1mHOtnzLD1d+xrFtQf/OikhPoGeuWcNO\nanten+nJ0aBo38TuWfAiUZlt+ZYf7vxCy30fKmc6ubCnAjVyPU8APOneR3uBNMac/pYAdUWkloiU\nAO4AvvIiOLI3sY0xxZ6qZojIQ8AMIAYYoaqrvciO9gIZtn1AYc62fMsPd35Iqeo0YJrXuRF90y5j\njAkn2wdpjDEBWIE0xpgAorJAhuu6TSd7hIjsEpFVXubmyq8hInNFZK2IrBaR3h7nnyEii0VkuZM/\n0Mt8pw0xIrJURKZ4ne3kbxaRlSKyTESSPM6uICKfikiy82+gqZf50Sbq9kE6122uI9d1m8CdXly3\n6eRfCxwGRqvqpV5knpRfDaimqj+JSFngR6CTh+svQGlVPSwiccACoLeqLvQi32lDX6ARUE5V23uV\nmyt/M9BIVfeEIXsU8J2qfuCcElNKVQ943Y5oEY09yLBdtwmgqvOBfV7l5ZGfpqo/OY8PAWvxX6rl\nVb6q6mHnaZwzefa/sIgkAO2AD7zKjBQiUg64FvgQQFWPWXH8Y6KxQOZ13aZnBSKSiEhNoCGwyOPc\nGBFZBuwCZqmql/lvAv2BLA8zT6bATBH50bmMziu1gd3AR84uhg9EpLSH+VEnGgtk2K7bjCQiUgb4\nDOijqr94ma2qmap6Of5LwpqIiCe7GkSkPbBLVX/0Ii8fzVT1CuAmoJez28ULscAVwLuq2hA4Ani6\nDz7aRGOBDNt1m5HC2ff3GfCxqn4ernY4m3fzgLYeRTYDOjr7AMcD14nIWI+yc6jqdufvXcAX+Hf7\neCEVSM3VY/8Uf8E0RRSNBTJs121GAucgyYfAWlV9PQz5lUSkgvP4TOAGINmLbFV9UlUTVLUm/u99\njqp29SI7m4iUdg6O4WzetgY8OaNBVXcAW0XkQmfW9YAnB+eiVdRdahjO6zYBRGQc0BKoKCKpwHOq\n+qFX+fh7UXcDK539gABPOZdqeaEaMMo5m8AHJKpqWE63CZMqwBf+/6eIBT5R1a89zH8Y+NjpHGwE\n7vUwO+pE3Wk+xhgTKtG4iW2MMSFhBdIYYwKwAmmMMQFYgTTGmACsQBpjTABWIE1EEZGa4RoJyZiT\nWYE0nnDOizTmtGIF0uRJRP6VeyxJERkkIo/ksVxLEZkvIl+IyBoReU9EfM5rh0XkeRFZBDQVkStF\n5FtnEIcZztBsOPOXi8gPQC+v1tGYgliBNIF8CHQDcAreHcDHAZZtAvwTqA+cD/zFmV8aWKWqV+Ef\nUegt4DZVvRIYAQxylvsIeERVbXBXE1Gi7lJDExqqullE9opIQ/yXzy1V1b0BFl+sqhsh51LL5vgH\nSsjEP2gGwIXApcAs5zK8GCBNRMoDFVT1W2e5MfhHwTEm7KxAmvx8AHQHquLv8QVy8vWq2c9/U9VM\n57EAq0/uJToDW9j1riYi2Sa2yc8X+Icqa4x/8I9AmjijJ/mALvhvs3Cyn4FK2fdIEZE4EbnEGRLt\noIg0d5a7K3TNN+aPsR6kCUhVj4nIXOBArp5gXn4AXsK/D3I+/sKa12fdBgx1Nqtj8Y/+vRr/iDMj\nROQo+RdiYzxlo/mYgJwe4U/A7aq6PsAyLYHHwnFzLGPcZpvYJk8icjGQAswOVByNiXbWgzSFIiL1\n8R9hzu135xQeY6KSFUhjjAnANrGNMSYAK5DGGBOAFUhjjAnACqQxxgRgBdIYYwL4f0PeymnbxSRa\nAAAAAElFTkSuQmCC\n",
      "text/plain": [
       "<Figure size 360x360 with 2 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "# XGboost training and prediction\n",
    "xg = xgb.XGBClassifier(n_estimators = 10)\n",
    "xg.fit(X_train,y_train)\n",
    "xg_score=xg.score(X_test,y_test)\n",
    "y_predict=xg.predict(X_test)\n",
    "y_true=y_test\n",
    "print('Accuracy of XGBoost: '+ str(xg_score))\n",
    "precision,recall,fscore,none= precision_recall_fscore_support(y_true, y_predict, average='weighted') \n",
    "print('Precision of XGBoost: '+(str(precision)))\n",
    "print('Recall of XGBoost: '+(str(recall)))\n",
    "print('F1-score of XGBoost: '+(str(fscore)))\n",
    "print(classification_report(y_true,y_predict))\n",
    "cm=confusion_matrix(y_true,y_predict)\n",
    "f,ax=plt.subplots(figsize=(5,5))\n",
    "sns.heatmap(cm,annot=True,linewidth=0.5,linecolor=\"red\",fmt=\".0f\",ax=ax)\n",
    "plt.xlabel(\"y_pred\")\n",
    "plt.ylabel(\"y_true\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 18,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "xg_train=xg.predict(X_train)\n",
    "xg_test=xg.predict(X_test)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {
    "collapsed": true
   },
   "source": [
    "### Stacking model construction (ensemble for 4 base learners)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 19,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>DecisionTree</th>\n",
       "      <th>ExtraTrees</th>\n",
       "      <th>RandomForest</th>\n",
       "      <th>XgBoost</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>2</td>\n",
       "      <td>2</td>\n",
       "      <td>2</td>\n",
       "      <td>2</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   DecisionTree  ExtraTrees  RandomForest  XgBoost\n",
       "0             5           5             5        5\n",
       "1             3           3             3        3\n",
       "2             5           5             5        5\n",
       "3             3           3             3        3\n",
       "4             2           2             2        2"
      ]
     },
     "execution_count": 19,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# Use the outputs of 4 base models to construct a new ensemble model\n",
    "base_predictions_train = pd.DataFrame( {\n",
    "    'DecisionTree': dt_train.ravel(),\n",
    "        'RandomForest': rf_train.ravel(),\n",
    "     'ExtraTrees': et_train.ravel(),\n",
    "     'XgBoost': xg_train.ravel(),\n",
    "    })\n",
    "base_predictions_train.head(5)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 20,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "dt_train=dt_train.reshape(-1, 1)\n",
    "et_train=et_train.reshape(-1, 1)\n",
    "rf_train=rf_train.reshape(-1, 1)\n",
    "xg_train=xg_train.reshape(-1, 1)\n",
    "dt_test=dt_test.reshape(-1, 1)\n",
    "et_test=et_test.reshape(-1, 1)\n",
    "rf_test=rf_test.reshape(-1, 1)\n",
    "xg_test=xg_test.reshape(-1, 1)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 21,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "x_train = np.concatenate(( dt_train, et_train, rf_train, xg_train), axis=1)\n",
    "x_test = np.concatenate(( dt_test, et_test, rf_test, xg_test), axis=1)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 22,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "stk = xgb.XGBClassifier().fit(x_train, y_train)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 23,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy of Stacking: 0.9960292949792641\n",
      "Precision of Stacking: 0.9960126519428796\n",
      "Recall of Stacking: 0.9960292949792641\n",
      "F1-score of Stacking: 0.9960148981765187\n",
      "              precision    recall  f1-score   support\n",
      "\n",
      "           0       1.00      0.99      1.00      4547\n",
      "           1       0.99      0.98      0.98       393\n",
      "           2       0.99      1.00      1.00       554\n",
      "           3       1.00      1.00      1.00      3807\n",
      "           4       0.83      0.71      0.77         7\n",
      "           5       1.00      1.00      1.00      1589\n",
      "           6       0.99      0.99      0.99       436\n",
      "\n",
      "    accuracy                           1.00     11333\n",
      "   macro avg       0.97      0.95      0.96     11333\n",
      "weighted avg       1.00      1.00      1.00     11333\n",
      "\n"
     ]
    },
    {
     "data": {
      "image/png": "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\nAyutpPYrFrEuAAAAAElFTkSuQmCC\n",
      "text/plain": [
       "<Figure size 360x360 with 2 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "y_predict=stk.predict(x_test)\n",
    "y_true=y_test\n",
    "stk_score=accuracy_score(y_true,y_predict)\n",
    "print('Accuracy of Stacking: '+ str(stk_score))\n",
    "precision,recall,fscore,none= precision_recall_fscore_support(y_true, y_predict, average='weighted') \n",
    "print('Precision of Stacking: '+(str(precision)))\n",
    "print('Recall of Stacking: '+(str(recall)))\n",
    "print('F1-score of Stacking: '+(str(fscore)))\n",
    "print(classification_report(y_true,y_predict))\n",
    "cm=confusion_matrix(y_true,y_predict)\n",
    "f,ax=plt.subplots(figsize=(5,5))\n",
    "sns.heatmap(cm,annot=True,linewidth=0.5,linecolor=\"red\",fmt=\".0f\",ax=ax)\n",
    "plt.xlabel(\"y_pred\")\n",
    "plt.ylabel(\"y_true\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Feature Selection"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Feature importance"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 24,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Save the feature importance lists generated by four tree-based algorithms\n",
    "dt_feature = dt.feature_importances_\n",
    "rf_feature = rf.feature_importances_\n",
    "et_feature = et.feature_importances_\n",
    "xgb_feature = xg.feature_importances_"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 25,
   "metadata": {},
   "outputs": [],
   "source": [
    "# calculate the average importance value of each feature\n",
    "avg_feature = (dt_feature + rf_feature + et_feature + xgb_feature)/4"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 26,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Features sorted by their score:\n",
      "[(0.1235, 'Bwd Packet Length Std'), (0.0853, 'Bwd Packet Length Min'), (0.0574, 'Average Packet Size'), (0.05, 'Init_Win_bytes_backward'), (0.0492, 'Bwd Packet Length Mean'), (0.0401, 'Init_Win_bytes_forward'), (0.0397, 'PSH Flag Count'), (0.038, 'Bwd Packets/s'), (0.034, 'Bwd Header Length'), (0.028, 'Avg Bwd Segment Size'), (0.0264, 'Packet Length Mean'), (0.0235, 'Packet Length Variance'), (0.0228, 'Fwd Header Length'), (0.0209, 'Bwd Packet Length Max'), (0.0193, 'min_seg_size_forward'), (0.0182, 'ACK Flag Count'), (0.017, 'act_data_pkt_fwd'), (0.0157, 'Fwd Header Length.1'), (0.0128, 'Packet Length Std'), (0.0122, 'Total Length of Fwd Packets'), (0.0117, 'Fwd PSH Flags'), (0.0108, 'Fwd Packet Length Max'), (0.0108, 'Fwd IAT Mean'), (0.0106, 'Total Fwd Packets'), (0.0106, 'Flow IAT Max'), (0.01, 'Subflow Fwd Bytes'), (0.0095, 'Fwd IAT Max'), (0.0094, 'Subflow Bwd Bytes'), (0.0094, 'Max Packet Length'), (0.0092, 'Subflow Bwd Packets'), (0.009, 'Min Packet Length'), (0.0086, 'Total Backward Packets'), (0.0086, 'Bwd IAT Total'), (0.0085, 'Idle Max'), (0.0083, 'Fwd IAT Min'), (0.008, 'Fwd Packet Length Mean'), (0.0075, 'URG Flag Count'), (0.0075, 'Subflow Fwd Packets'), (0.0071, 'Avg Fwd Segment Size'), (0.0063, 'Fwd IAT Std'), (0.0063, 'Down/Up Ratio'), (0.0061, 'Fwd IAT Total'), (0.0059, 'Flow Duration'), (0.0058, 'Fwd Packet Length Min'), (0.0057, 'Flow IAT Std'), (0.0055, 'Flow IAT Min'), (0.005, 'Total Length of Bwd Packets'), (0.0049, 'Bwd IAT Mean'), (0.0047, 'Bwd IAT Min'), (0.0045, 'Fwd Packets/s'), (0.0044, 'Fwd Packet Length Std'), (0.0043, 'Bwd IAT Max'), (0.0042, 'FIN Flag Count'), (0.004, 'Flow IAT Mean'), (0.0034, 'Idle Min'), (0.0031, 'SYN Flag Count'), (0.0027, 'Bwd IAT Std'), (0.0023, 'Idle Mean'), (0.0006, 'Active Mean'), (0.0004, 'Active Min'), (0.0003, 'Active Max'), (0.0002, 'Idle Std'), (0.0001, 'Active Std'), (0.0, 'RST Flag Count'), (0.0, 'Fwd URG Flags'), (0.0, 'Fwd Avg Packets/Bulk'), (0.0, 'Fwd Avg Bytes/Bulk'), (0.0, 'Fwd Avg Bulk Rate'), (0.0, 'Flow Packets/s'), (0.0, 'Flow Bytes/s'), (0.0, 'ECE Flag Count'), (0.0, 'CWE Flag Count'), (0.0, 'Bwd URG Flags'), (0.0, 'Bwd PSH Flags'), (0.0, 'Bwd Avg Packets/Bulk'), (0.0, 'Bwd Avg Bytes/Bulk'), (0.0, 'Bwd Avg Bulk Rate')]\n"
     ]
    }
   ],
   "source": [
    "feature=(df.drop(['Label'],axis=1)).columns.values\n",
    "print (\"Features sorted by their score:\")\n",
    "print (sorted(zip(map(lambda x: round(x, 4), avg_feature), feature), reverse=True))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 27,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "f_list = sorted(zip(map(lambda x: round(x, 4), avg_feature), feature), reverse=True)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 28,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/plain": [
       "77"
      ]
     },
     "execution_count": 28,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "len(f_list)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 29,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Select the important features from top-importance to bottom-importance until the accumulated importance reaches 0.9 (out of 1)\n",
    "Sum = 0\n",
    "fs = []\n",
    "for i in range(0, len(f_list)):\n",
    "    Sum = Sum + f_list[i][0]\n",
    "    fs.append(f_list[i][1])\n",
    "    if Sum>=0.9:\n",
    "        break        "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 30,
   "metadata": {},
   "outputs": [],
   "source": [
    "X_fs = df[fs].values"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 31,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "X_train, X_test, y_train, y_test = train_test_split(X_fs,y, train_size = 0.8, test_size = 0.2, random_state = 0,stratify = y)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 32,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "(45328, 38)"
      ]
     },
     "execution_count": 32,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "X_train.shape"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 33,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "0    18184\n",
       "3    15228\n",
       "5     6357\n",
       "2     2213\n",
       "6     1744\n",
       "1     1573\n",
       "4       29\n",
       "dtype: int64"
      ]
     },
     "execution_count": 33,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "pd.Series(y_train).value_counts()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Oversampling by SMOTE"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 34,
   "metadata": {},
   "outputs": [],
   "source": [
    "from imblearn.over_sampling import SMOTE\n",
    "smote=SMOTE(n_jobs=-1,sampling_strategy={4:1500})"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 35,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "X_train, y_train = smote.fit_resample(X_train, y_train)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 36,
   "metadata": {
    "scrolled": true
   },
   "outputs": [
    {
     "data": {
      "text/plain": [
       "0    18184\n",
       "3    15228\n",
       "5     6357\n",
       "2     2213\n",
       "6     1744\n",
       "1     1573\n",
       "4     1500\n",
       "dtype: int64"
      ]
     },
     "execution_count": 36,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "pd.Series(y_train).value_counts()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {
    "collapsed": true
   },
   "source": [
    "## Machine learning model training after feature selection"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 37,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy of DT: 0.9959410570899144\n",
      "Precision of DT: 0.9959421882780577\n",
      "Recall of DT: 0.9959410570899144\n",
      "F1-score of DT: 0.995922941273118\n",
      "              precision    recall  f1-score   support\n",
      "\n",
      "           0       1.00      0.99      1.00      4547\n",
      "           1       0.98      0.98      0.98       393\n",
      "           2       0.99      1.00      1.00       554\n",
      "           3       1.00      1.00      1.00      3807\n",
      "           4       1.00      0.71      0.83         7\n",
      "           5       1.00      1.00      1.00      1589\n",
      "           6       1.00      0.98      0.99       436\n",
      "\n",
      "    accuracy                           1.00     11333\n",
      "   macro avg       0.99      0.95      0.97     11333\n",
      "weighted avg       1.00      1.00      1.00     11333\n",
      "\n"
     ]
    },
    {
     "data": {
      "image/png": "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\n8zCdAAAAAElFTkSuQmCC\n",
      "text/plain": [
       "<Figure size 360x360 with 2 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "dt = DecisionTreeClassifier(random_state = 0)\n",
    "dt.fit(X_train,y_train) \n",
    "dt_score=dt.score(X_test,y_test)\n",
    "y_predict=dt.predict(X_test)\n",
    "y_true=y_test\n",
    "print('Accuracy of DT: '+ str(dt_score))\n",
    "precision,recall,fscore,none= precision_recall_fscore_support(y_true, y_predict, average='weighted') \n",
    "print('Precision of DT: '+(str(precision)))\n",
    "print('Recall of DT: '+(str(recall)))\n",
    "print('F1-score of DT: '+(str(fscore)))\n",
    "print(classification_report(y_true,y_predict))\n",
    "cm=confusion_matrix(y_true,y_predict)\n",
    "f,ax=plt.subplots(figsize=(5,5))\n",
    "sns.heatmap(cm,annot=True,linewidth=0.5,linecolor=\"red\",fmt=\".0f\",ax=ax)\n",
    "plt.xlabel(\"y_pred\")\n",
    "plt.ylabel(\"y_true\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 38,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "dt_train=dt.predict(X_train)\n",
    "dt_test=dt.predict(X_test)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 39,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy of RF: 0.9963822465366629\n",
      "Precision of RF: 0.9963638997836398\n",
      "Recall of RF: 0.9963822465366629\n",
      "F1-score of RF: 0.9963668543569548\n",
      "              precision    recall  f1-score   support\n",
      "\n",
      "           0       1.00      1.00      1.00      4547\n",
      "           1       0.99      0.98      0.98       393\n",
      "           2       1.00      1.00      1.00       554\n",
      "           3       1.00      1.00      1.00      3807\n",
      "           4       0.83      0.71      0.77         7\n",
      "           5       1.00      1.00      1.00      1589\n",
      "           6       1.00      0.98      0.99       436\n",
      "\n",
      "    accuracy                           1.00     11333\n",
      "   macro avg       0.97      0.95      0.96     11333\n",
      "weighted avg       1.00      1.00      1.00     11333\n",
      "\n"
     ]
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 360x360 with 2 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "rf = RandomForestClassifier(random_state = 0)\n",
    "rf.fit(X_train,y_train) # modelin veri üzerinde öğrenmesi fit fonksiyonuyla yapılıyor\n",
    "rf_score=rf.score(X_test,y_test)\n",
    "y_predict=rf.predict(X_test)\n",
    "y_true=y_test\n",
    "print('Accuracy of RF: '+ str(rf_score))\n",
    "precision,recall,fscore,none= precision_recall_fscore_support(y_true, y_predict, average='weighted') \n",
    "print('Precision of RF: '+(str(precision)))\n",
    "print('Recall of RF: '+(str(recall)))\n",
    "print('F1-score of RF: '+(str(fscore)))\n",
    "print(classification_report(y_true,y_predict))\n",
    "cm=confusion_matrix(y_true,y_predict)\n",
    "f,ax=plt.subplots(figsize=(5,5))\n",
    "sns.heatmap(cm,annot=True,linewidth=0.5,linecolor=\"red\",fmt=\".0f\",ax=ax)\n",
    "plt.xlabel(\"y_pred\")\n",
    "plt.ylabel(\"y_true\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 40,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "rf_train=rf.predict(X_train)\n",
    "rf_test=rf.predict(X_test)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 41,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy of ET: 0.995764581311215\n",
      "Precision of ET: 0.9957533838731291\n",
      "Recall of ET: 0.995764581311215\n",
      "F1-score of ET: 0.9957536562646003\n",
      "              precision    recall  f1-score   support\n",
      "\n",
      "           0       1.00      0.99      0.99      4547\n",
      "           1       0.97      0.98      0.98       393\n",
      "           2       1.00      1.00      1.00       554\n",
      "           3       1.00      1.00      1.00      3807\n",
      "           4       0.83      0.71      0.77         7\n",
      "           5       1.00      1.00      1.00      1589\n",
      "           6       1.00      0.99      0.99       436\n",
      "\n",
      "    accuracy                           1.00     11333\n",
      "   macro avg       0.97      0.95      0.96     11333\n",
      "weighted avg       1.00      1.00      1.00     11333\n",
      "\n"
     ]
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 360x360 with 2 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "et = ExtraTreesClassifier(random_state = 0)\n",
    "et.fit(X_train,y_train) \n",
    "et_score=et.score(X_test,y_test)\n",
    "y_predict=et.predict(X_test)\n",
    "y_true=y_test\n",
    "print('Accuracy of ET: '+ str(et_score))\n",
    "precision,recall,fscore,none= precision_recall_fscore_support(y_true, y_predict, average='weighted') \n",
    "print('Precision of ET: '+(str(precision)))\n",
    "print('Recall of ET: '+(str(recall)))\n",
    "print('F1-score of ET: '+(str(fscore)))\n",
    "print(classification_report(y_true,y_predict))\n",
    "cm=confusion_matrix(y_true,y_predict)\n",
    "f,ax=plt.subplots(figsize=(5,5))\n",
    "sns.heatmap(cm,annot=True,linewidth=0.5,linecolor=\"red\",fmt=\".0f\",ax=ax)\n",
    "plt.xlabel(\"y_pred\")\n",
    "plt.ylabel(\"y_true\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 42,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "et_train=et.predict(X_train)\n",
    "et_test=et.predict(X_test)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 43,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy of XGBoost: 0.9945292508603194\n",
      "Precision of XGBoost: 0.9945034026564313\n",
      "Recall of XGBoost: 0.9945292508603194\n",
      "F1-score of XGBoost: 0.9944940662484604\n",
      "              precision    recall  f1-score   support\n",
      "\n",
      "           0       0.99      0.99      0.99      4547\n",
      "           1       1.00      0.98      0.99       393\n",
      "           2       1.00      1.00      1.00       554\n",
      "           3       0.99      1.00      1.00      3807\n",
      "           4       0.80      0.57      0.67         7\n",
      "           5       1.00      1.00      1.00      1589\n",
      "           6       1.00      0.98      0.99       436\n",
      "\n",
      "    accuracy                           0.99     11333\n",
      "   macro avg       0.97      0.93      0.95     11333\n",
      "weighted avg       0.99      0.99      0.99     11333\n",
      "\n"
     ]
    },
    {
     "data": {
      "image/png": "iVBORw0KGgoAAAANSUhEUgAAAUgAAAFBCAYAAAAVGzb7AAAABHNCSVQICAgIfAhkiAAAAAlwSFlz\nAAALEgAACxIB0t1+/AAAADl0RVh0U29mdHdhcmUAbWF0cGxvdGxpYiB2ZXJzaW9uIDMuMC4zLCBo\ndHRwOi8vbWF0cGxvdGxpYi5vcmcvnQurowAAIABJREFUeJzt3Xl4FFXWwOHf6STsuyBLggKCioII\nAoq4oCiggPCpAy4oOCqjoogbbjgMjsy4DCqMzjigCG5AFAFZFJBFXIEIYTWyC4EAIjsIZjnfH10J\nAVNJuu2ubjrn5akn3ber+tzqJif31q26JaqKMcaY3/NFugLGGBOtLEEaY4wLS5DGGOPCEqQxxriw\nBGmMMS4sQRpjjAtLkMYY48ISpDHGuLAEaYwxLuIjXYFCidhlPsZEgqoEs1nmrg0B/84mVG8QVCwv\nRHWCzPx5fcRiJ9Q4g/iEOhGLn5W5LeLxEyIUPzNzG0DE9j/L4kckbjSK6gRpjDnJ5GRHugYhZQnS\nGBM6mhPpGoSUJUhjTOjkWII0xpgCqbUgjTHGhbUgjTHGhbUgjTHGhY1iG2OMC2tBGmOMCzsGaYwx\nBbNRbGOMcWMtSGOMcWEtSGOMcRFjo9gn3XyQ2dnZ3NinH/c9NhiAp58bRscb+3BD737c0LsfaWv8\nMwBt+GkLt/Z9iObtuvL2Bx8d9x7vJk+me6976HbrX3h3wqSQ1m/UyGFsS19G6tI5IX3faI6flFSH\n2bM+ZPny+aSmzuWB++8E4IYbupCaOpejR7ZwQYvzPKlLpD9/AJ/Px+JFM5kyaaznsTt2aMeqlQtI\nW/0VAx/r53l8NCfwJYqddAnyvQ+n0KDeaceVPdLvTiaOfZ2JY1/n7DPPAKBypYo88dA99Ln5huPW\nXbthExM/+Yxxb77KxLH/4YtvFvHTlq0hq9877yTTucutIXu/kyF+VlYWAwcO4bzz2nHJJV25594+\nNG7ciFWr0ujR426+/PI7z+oS6c8foP8Dd5GWttbzuD6fjxHDh9Klay+aNruCnj2707hxI8/rEUvC\nliBF5GwReVxERojIcOdx4z/yntt3/syCbxZxQ9eORa57StUqNG18FvHxxx9F2LBpC+edezZly5Qh\nPj6Oluc3Zc6Cb/5ItY7z5VcL2b1nb8je72SIv337TpamrgTg4MFDpKWtpU6dWqSlrWPNGm/n9Iz0\n55+YWJtrr2nP6NHjPI/dulVz1q/fxMaNm8nMzCQ5eQrXFeN3JaRycgJfolhYEqSIPA6MBwRYBCx2\nHo8TkSeCfd8Xhv+Ph++7E5Hjqz3if2P5v9vv5YXh/+O3334r9D0aNjid75etZO++/fx65AhffruY\n7Tt+DrZK5gSnn57E+c2asGjR0khXJSJeHjaEJ558jpwI/OLXSazFlvRjk92mb82gTp1a3lYixrrY\n4RqkuRM4V1Uz8xeKyMvAKuD5QN9w/tcLqVa1Cuee3YhFS5bnlQ+45w6qn1KVzMxM/vbCCN5670Pu\n/bN7F+uMeqfx51v/xN0DnqJc2bKc2bABcXFxgVbHFKB8+XIkTxjFI48O5sCBg5Gujuc6X3sVO3fu\nYsnSFVx+WRvP44v8/s4Fqh7ftSTKW4SBClcXOwcoaL742s5rrkSkr4ikiEjKm+8c66YsXb6a+V99\nR4cbevPY4OdZ9P0yHh/yIjWqV0NEKFWqFN07d2DFD2uKrNwNXTvy4duvMfY/L1G5UkVOr5sY4O6Z\nE8XHx5M8YRTjxk1i8uRPI12diLj44pZ07dKBdWu+4/33/sMVV7Rl7JgRnsXfmp5B3aRjv3ZJibXJ\nyNjhWXwA1eyAl2gWrhbkAGCOiKwFtjhlpwENgfsL21BVRwIj4fgbAD107x08dO8dACxaspwx4yby\nwuCB/LxrNzWqV0NVmbvgGxo1OL3Iyv2yZy+nVK1CxvadzPnia97738vB7KPJZ9TIYaSlrePV4SMj\nXZWIeXrQ8zw9yN85uvyyNjz80D307tPfs/iLU1Jp2LA+9erVZevW7fTo0Y3bbvd4JDvKu8yBCkuC\nVNXPRORMoDWQiP/4YzqwWEP8J+PxIS+yZ+8+VJWzGjVg8GMPALDrl930vLM/Bw8dxufz8V7yZKa8\n/z8qlC/PQ089x979+4mPj+fpR+6jcqWKIavPe+++zuWXtaF69Wps2pDCkGf/xdtjxofs/aMxftuL\nW9Gr142sWLGalMWzABj0zPOULl2KV195jho1qjFlyjssW7Yq7CPMkf78Iyk7O5sHBwxixvQPiPP5\nGDN2AqtXF92jCqkY62KL58coAhDMLSRDxe5qaHc1LNHxg7zt65HvJwf8O1vmgu5221djTAkQY1fS\nWII0xoSOHYM0xhgXMXYM0hKkMSZ0YqwFedJdi22MiWJhvNRQROJEZKmITHOe1xeRhSKyVkQmiEgp\np7y083yd83q9fO/xpFP+o4gUeR2mJUhjTOiE91rsB4Ef8j1/AXhFVRsBe/BfwYfzc4+qNgRecdZD\nRM4BbgLOBToB/xGRQi+jswRpjAmZcF1JIyJJQGfgTee5AFcCuXMZjgW6O4+7Oc9xXm/vrN8NGK+q\nR1V1I7AO/7narixBGmNCJ4gWZP7Li52lbwHv/CowkGOXKp8C7FXVLOd5Ov6LUnB+bgFwXt/nrJ9X\nXsA2BbJBGmNM6AQxSJP/8uKCiEgXYKeqfi8i7XKLC3qrIl4rbJsCWYI0xkS7tsB1InItUAaohL9F\nWUVE4p1WYhKQO9dbOlAXSBeReKAysDtfea782xTIutjGmNAJwyCNqj6pqkmqWg//IMtcVb0VmAfc\n6KzWG5jiPP7EeY7z+lz1X1P9CXCTM8pdH2iEf75aV9aCNMaEjrfnQT4OjBeR54ClwFtO+VvAuyKy\nDn/L8SYAVV0lIsnAaiAL6FfU5DmWII0xoRPmK2lUdT4w33m8gQJGoVX1CPAnl+2HAkOLG88SpDEm\ndGLsSpqoTpAJNc6IaPzcaadKavzMEr7/JT1+UOxabGOMcWEJ0juRnjC2fjVvbnZfkI27l0d8/0v0\nhLEWPzjWxTbGGBfWgjTGGBfWgjTGGBfWgjTGGBfWgjTGGBfWgjTGGBeWII0xxoVG7Fb2YWEJ0hgT\nOtaCNMYYF5YgjTHGRYyNYtuEucYY48JakMaY0LEutjHGuIixUeyY7GI/cP+dpC6dw7LUufR/4K6w\nxChVuhSTZ7/PjC+Smfn1xwx4/F4ALr6sNVPnjmf6/AkkTx/D6fXrHrfdNV2vYuMvy2h6/jlhqRdA\nxw7tWLVyAWmrv2LgY/3CFsfiR1fsaIgfjnvSRFLMJchzzz2LO++8hTYXd6bFBVfT+dqraNiwfsjj\n/Hb0N27pfhfXXt6Dzpf34PL2bTm/ZVOee2kQA+55ks7tevLJxBnc/8jdeduUr1COPn1vYWnK8pDX\nJ5fP52PE8KF06dqLps2uoGfP7jRu3Chs8Sx+dMSOhviAJchod/bZjVi4cAm//nqE7OxsFnz5Hd27\ndQpLrMOHfgUgPiGe+Ph4UFCUihUrAFCxUgV2bP85b/2Hn+zH//49hqNHjoalPgCtWzVn/fpNbNy4\nmczMTJKTp3Bd145hi2fxoyN2NMQH/KPYgS5RLCIJUkTuCNd7r1qVxqWXXkS1alUpW7YM13S6kqSk\n8Ew86vP5mD5/Ailp8/jqi+9I/X4FTzz4N0aPf41vVszi/3p04Y3howE4p+nZ1E6sxdxZC8JSl1x1\nEmuxJf3YhKfpWzOoU6dWWGNa/MjHjob4AJqjAS/RLFItyCFuL4hIXxFJEZGUnJxDAb9xWto6Xnrp\ndT77dBwzpr3PsuWryc4q9M6OQcvJyaFzu560adqBZs2bcObZDfnzvbfx55vu5+KmHfjogykM+vuj\niAjPPPcoQ58ZFpZ65CcivytTDw+cl+T4JXnf81gXu3hEZLnLsgKo6badqo5U1Zaq2tLnKx9U7LfH\njKf1hZ24ov0N7Nmzl7XrNga7G8VyYP8Bvvt6Me2uakvjc88k9fsVAEybNJMWrZtRoUJ5zmzckPGf\nvMmXS2fQvOV5jHp/eFgGaramZ1A3X4s5KbE2GRk7Qh7H4kdX7GiID1gXOwA1gduBrgUsv4QxLjVq\nnAJA3bp16N79GsZPmBzyGNVOqUrFShUBKF2mNJdcfhHr1mykYqUK1D/jdAAuadeGdWs2cuDAQS44\nsx2XNr+WS5tfy9KU5dx964OsSF0d8notTkmlYcP61KtXl4SEBHr06MbUabNCHsfiR1fsaIgPQI4G\nvkSxcJ4HOQ2ooKqpJ74gIvPDGJcPJ4yi2ilVyczMon//p9m7d1/IY5xaszr/ev054uJ8iM/H9Mmz\nmDtrAU8+9Cz/GTMMzclh3979DOw/OOSxC5Odnc2DAwYxY/oHxPl8jBk7gdWr11j8GI8dDfGBqO8y\nB0o8P0YRgPhSiRGrnN3V0O5qWKLjq/7+gGYxHB5+T8C/s+UefCOoWF6wK2mMMaETxQ2uYFiCNMaE\nTox1sS1BGmNCJ8oHXQJlCdIYEzpRftpOoCxBGmNCJ8ZakDF3LbYxxoSKtSCNMSGjNkhjjDEuYqyL\nbQnSGBM6NkhjjDEurAVpjDEu7BikMca4sBakMca4sGOQxhjjwlqQ3smd9ilSNu4O390HiyPS+2/x\nS3b8YNh5kB4qqfMh5sZvVrNNxOIv2/FtyZ4P0eIHx1qQxhjjwhKkMca4sEEaY4xxYS1IY4wpmFqC\nNMYYF5YgjTHGRYyd5mMT5hpjjAtrQRpjQse62MYY4yLGEqR1sY0xIaOqAS9FEZEyIrJIRJaJyCoR\nGeKU1xeRhSKyVkQmiEgpp7y083yd83q9fO/1pFP+o4h0LCq2JUhjTOjkaOBL0Y4CV6pqM+B8oJOI\nXAS8ALyiqo2APcCdzvp3AntUtSHwirMeInIOcBNwLtAJ+I+IxBUW2BKkMSZ0wpAg1e+g8zTBWRS4\nEvjIKR8LdHced3Oe47zeXkTEKR+vqkdVdSOwDmhdWGxLkMaYkNEcDXgRkb4ikpJv6Xvi+4pInIik\nAjuB2cB6YK+qZjmrpAOJzuNEYAuA8/o+4JT85QVsUyAbpDHGhE4QgzSqOhIYWcQ62cD5IlIFmAQ0\nLmg156e4vOZW7irmWpBJSXX4fNaHrFg+n2Wpc3ng/juL3ijEOnZox6qVC0hb/RUDH+sXtjgzFk/k\no3nvMuHzMXww8y0A7nn0TmYvncKEz8cw4fMxXNLeP2XaRZe1YtzM0Xw0713GzRxN67YXhK1eXu1/\nNMYvyfsOQE4QSwBUdS8wH7gIqCIiuY28JCB3nrZ0oC6A83plYHf+8gK2KVDMtSCzsrJ4bOAQlqau\npEKF8ixa+Bmfz1nADz+s9SS+z+djxPChdLr2ZtLTM/ju2xlMnTYrbPHvuuF+9u7ed1zZuyPH885/\nxx1Xtnf3PvrfPpCfd+yi4dkN+O+4V7i6ebeQ18fr/Y+m+CV533OF41psEakBZKrqXhEpC1yFf+Bl\nHnAjMB7oDUxxNvnEef6t8/pcVVUR+QT4QEReBuoAjYBFhcUOWwtSRM4WkfYiUuGE8k7higmwfftO\nlqauBODgwUOkpa0lsU6tcIY8TutWzVm/fhMbN24mMzOT5OQpXNe1yLMJwi5t5Rp+3rELgHVpGyhV\nuhQJpRJCHifS+x/J+CV53/OEZxS7NjBPRJYDi4HZqjoNeBx4WETW4T/G+Jaz/lvAKU75w8ATAKq6\nCkgGVgOfAf2crrursCRIEemPP5s/AKwUkfxNlX+EI2ZBTj89ifObNWHhoqVehaROYi22pB9rtadv\nzaBOuBK0Km+Mf5VxM0dzQ69jH/FNf76RD+e+w5BXnqJi5Yq/2+yqLleQtnINmb9lhrxKnu5/lMUv\nyfueJwxdbFVdrqrNVfU8VW2iqs865RtUtbWqNlTVP6nqUaf8iPO8ofP6hnzvNVRVz1DVs1T106Ji\nh6uLfTdwgaoedE7S/EhE6qnqcAo+UJrHGcHqCyBxlfH5ygdVgfLly5E8YRQPPzqYAwcOFr1BiPjP\nJjhecU6GDUbvrvfw845dVKtelTcmvMrGdT+RPOZjRr78NqpKv8f78ujfHmDwQ8f+Jp1xVn0GDLqP\ne3oOCEudvNz/aItfkvc9L55dSVMscbnnLanqJqAdcI3T9y80QarqSFVtqaotg02O8fHxfDhhFOPG\nTWLy5CL/SITU1vQM6iYdu5dIUmJtMjJ2hCVWbpd59649zP10AU2aN2b3rj3k5OSgqnz8/hSaND8n\nb/1Ta9fgldH/ZNADz5L+09aw1MnL/Y+2+CV53/OEeZDGa+FKkNtF5PzcJ06y7AJUB5qGKWaeUSOH\n8UPaOl4dXuiZA2GxOCWVhg3rU69eXRISEujRoxtTp80KeZyy5cpQrny5vMdtLm/NurQNVD/1lLx1\nrrzmctal+XsXFStV4LX3/sXwf7xB6uIVIa9PLq/2Pxrjl+R9zxXMeZDRLFxd7NuBrPwFzgmbt4vI\n/8IUE4C2F7fitl43snzFalIW+/9zPPPM83z62dxwhs2TnZ3NgwMGMWP6B8T5fIwZO4HVq9eEPE61\n6tV45e1/AhAfH8eMj2fzzbyFDP33XzmrSSNUlW1bMvj7Yy8C/uOSp9VPou9Dfej7UB8A7r3pIXbv\n2hPSenm1/9EYvyTve54obxEGSrw+RhGI+FKJEauc3fbVbvtaouOrFnoozM0vXS8P+Hf2lKlfBBXL\nCzF3orgxxoRKzJ0oboyJoBjrYluCNMaETIzdFtsSpDEmhCxBGmNMwawFaYwxLixBGmOMC0uQxhjj\nJrjTJ6OWJUhjTMhYC9IYY1xojrUgjTGmQNaCNMYYF0Fewh21LEEaY0LGWpDGGOMi1o5BRvV0Z4hE\nceWMiWFB9pU3t2wf8O/saSlzojarRnULMtLzMZb0+L9+/M+IxC57/ZNACZ+PMQriByPWWpBRnSCN\nMSeXWEuQNmGuMca4sBakMSZkonlIIxiWII0xIRNrXWxLkMaYkIm1E8WLPAYpIjVF5C0R+dR5fo6I\n3Bn+qhljTjaaE/gSzYozSDMGmAnknnOwBhgQrgoZY05eOSoBL9GsOAmyuqom49xtQlWzgOyw1soY\nc1JSlYCXaFacY5CHROQUQAFE5CJgX1hrZYw5KZXEQZqHgU+AM0Tka6AGcGNYa2WMOSmVuNN8VHWJ\niFwOnAUI8KOqZoa9ZsaYk06Ja0GKyO0nFLUQEVT1nTDVyRhzkor2QZdAFaeL3Srf4zJAe2AJYAnS\nGHOcaB90CVRxutgP5H8uIpWBd8NWI2PMSSvWjkEGM1nFYaBRqCsSKmeeeQYpi2flLbt3pdH/gbs8\nrUPHDu1YtXIBaau/YuBj/TyNHa74RzOzuPW1qfR4dTLXvzyJ/8xeCsDCddu4acQUegyfQp//Tmfz\nrv0A/JaVzcAP5tH1pY/o9fpUtu4+kPdeb81bTteXPqLbvybyzZqtIalffpH8/GPxuw9ErJ0HWZxj\nkFNxTvHBn1DPAZLDWak/Ys2a9bRs1QEAn8/H5k3fM3nKp57F9/l8jBg+lE7X3kx6egbffTuDqdNm\n8cMPa0/q+KXi4xh1dyfKlU4gMzuHO96YziVnJTJ08re8ent7GpxahQnf/sCoucv4e49LmbR4DZXK\nlmbqYzfy2bINDP8shRdvuYL1O/Yyc9kGJj70f/y8/zB/eXMmUx69njhfaCaWiuTnH6vffSBirYtd\nnP+V/wKGOcs/gctU9YmiNhKR1iLSynl8jog8LCLX/qHaBqj9lZewYcNPbN4c+laKm9atmrN+/SY2\nbtxMZmYmyclTuK5rx5M+vohQrnQCAFnZOWRl5yAIAhw64j+p4eCRTGpUKgfA/NWb6dqiIQBXNanH\nonUZqCrzV2+mY7MGlIqPI7FaReqeUpGVW3b94frliuTnH6vffSBUA1+iWaEtSBGJA55R1asCeVMR\nGQxcA8SLyGzgQmA+8ISINFfVoUHWNyA9enRj/ITJXoTKUyexFlvSj83InL41g9atmsdE/OycHG7+\n91S2/LKfnm3OpulpNRh8Q1vuHzOb0vFxVCiTwDv3dQFg5/7D1KpSHoD4OB8VypRi7+Gj7Nx/iPNO\nOzXvPWtWLs/O/YdDUj+I7Ocfy999cUV7lzlQhbYgVTUbOOwMzATiRqAtcBnQD+iuqs8CHYGehW0o\nIn1FJEVEUnJyDgUY9piEhAS6dunARxOnBf0ewRD5/X8QL+/7E874cT4fyQ92Y+aTPVi5ZRfrtu/h\nva9W8Vqfq5n1VE+uu6ARw6YtcmIWULdCykMlkp9/LH/3xVUSLzU8AqxwWoJ5GUtV+xeyTVa+5Lpe\nVfc72/wqIoXO36GqI4GRAPGlEoP+djt1uoKlS1ewc2foum/FsTU9g7pJx+4lkpRYm4yMHTEVv1LZ\n0rRsUIuvfkxnTcYemp5WA4COzerTb/QsAGpWLsf2vYeoWbk8Wdk5HDzyG5XLlaZm5fJs33vsD9+O\nfYfyuuWhEMnPvyR89yVNcY5BTgeeARYA3ztLShHb/CYiuf/rL8gtdFqinkxwdFPP7p53rwEWp6TS\nsGF96tWrS0JCAj16dGPqtFknffzdB4+w/9ejABzJzGLhugwanFqFg0d+46ef/Zfmf7d2G/VrVAHg\n8nNOY+qSdQB8vnITrc6ojYhw+Tl1mblsA79lZbN19wE2/7KfJnWr/+H65Yrk5x+r330gStwoNlBF\nVYfnLxCRB4vY5jJVPQqgetyMbwlA78CqGLiyZctwVfvLuPe+x8Md6neys7N5cMAgZkz/gDifjzFj\nJ7B69ZqTPv6uA4d5JvlLclTJUaVD0/pc1rguf72+LY+8NxefCBXLlmbIjZcA8H8tG/F08pd0fekj\nKpUtzQs3twOgYc2qXH1efa5/eRJxPuHJbm1CNoINkf38Y/W7D0SUj7kErMj7YovIElVtcULZUlUN\n+9HfP9LF/qOi4barkY5vt30twfGDPDj4Te0bAv6dvThjYtQ2I11bkCJyM3ALUF9EPsn3UkXgl3BX\nzBhz8on2QZdAFdbF/gbIAKrjPwcy1wFgeTgrZYw5OUX5HRQC5pogVfUn4CegTWFvICLfqmqh6xhj\nSgYN6UlbkReKuxqWCcF7GGNiQE6MjdKEIkHG2EdijAlWjrUgjTGmYLHWxS7OfbHvF5Gqha0SwvoY\nY05iOUEs0aw4Z+jWAhaLSLKIdJLfX/B5WxjqZYw5CSkS8FIUEakrIvNE5AcRWZV7oYqIVBOR2SKy\n1vlZ1SkXERkhIutEZLmItMj3Xr2d9deKSJEXrRSZIFV1EP4Jct8C+gBrReQfInKG8/rKIvfQGFMi\nhKkFmQU8oqqNgYuAfiJyDvAEMEdVGwFznOfgn0mskbP0Bf4L/oQKDMY/u1hrYHARvePizSiu/stt\ntjtLFlAV+EhEXize/hljSoJwJEhVzVDVJc7jA8APQCLQDRjrrDYW6O487ga8o37fAVVEpDb+2cRm\nq+puVd0DzAY6FRa7ODOK98d//fQu4E3gMVXNFBEfsBYYWIx9NMaUAOEepBGRekBzYCFQU1UzwJ9E\nRSR3otFEYEu+zdKdMrdyV8UZxa4OXO+cOJ5HVXNEpEsxtjfGlBDB3BZbRPri7wrnGulMe3jiehWA\nicAAVd1f0PyXuasWUKaFlLsqzl0N/1rIaz8Utb0xpuQI5jzI/HPAuhGRBPzJ8X1V/dgp3iEitZ3W\nY21gp1OeDtTNt3kSsM0pb3dC+fzC4oZuniljjAkD58yZt4AfVPXlfC99wrHpE3sDU/KV3+6MZl8E\n7HO64jOBDiJS1Rmc6eCUucf2ekr2gIhEceWMiWFBTsszudYtAf/Odt/+QaGxROQS4EtgBcfGdZ7C\nfxwyGTgN2Az8SVV3Own1NfwDMIeBO1Q1xXmvPzvbAgxV1bcLjR3NCdLmgyyZ8aNiPsSSHj/IBPlx\nEAny+iISZCTZpYbGmJDJcR84OSlZgjTGhEz09keDYwnSGBMy0X5tdaAsQRpjQiaY8yCjmSVIY0zI\n2HyQxhjjwo5BGmOMC+tiG2OMCxukMcYYF9bFNsYYF9bFNsYYF9bFNsYYF5YgjTHGRXBTXESvmJsP\nsnTp0nz79TS+T5nNstS5DP7rI57XoWOHdqxauYC01V8x8LF+Ft9jPp+PxYtmMmXS2KJXDrFI73uk\n45fE276eVI4ePcpVHXpwQcuruaBlBzp2aMeFrVsUvWGI+Hw+RgwfSpeuvWja7Ap69uxO48aNLL6H\n+j9wF2lpaz2NCZHf90jHj0UxlyABDh06DEBCQjzxCQl4Oedl61bNWb9+Exs3biYzM5Pk5Clc17Wj\nxfdIYmJtrr2mPaNHj/MsZq5I73uk44O1IIMmIu94Fcvn85GyeBYZW5czZ84CFi1e6lVo6iTWYkv6\ntrzn6VszqFOnlsX3yMvDhvDEk8+Rk+P9r16k9z3S8cF/HmSgSzQLyyCNiHxyYhFwhYhUAVDV68IR\nN1dOTg4tW3WgcuVKTPzwLc499yxWrfoxnCHzFHSnNS9bsCU5fudrr2Lnzl0sWbqCyy9r40nM/Ery\nZ5/LzoMsniRgNf77aOfebrElMKyoDfPfAlLiKuPzlQ+6Evv27eeLBd/4D1x7lCC3pmdQN+nYVPlJ\nibXJyNjhSeySHv/ii1vStUsHrul0JWXKlKZSpYqMHTOC3n36exK/JH/2uaK9yxyocHWxWwLfA0/j\nv6PYfOBXVf1CVb8obENVHamqLVW1ZTDJsXr1alSuXAmAMmXK0P7KS/nxx/UBv0+wFqek0rBhferV\nq0tCQgI9enRj6rRZFt8DTw96nnoNWtLwzIu4tdd9zJv3tWfJEUr2Z58r1o5BhqUFqao5wCsi8qHz\nc0e4Yp2odu2ajH7rVeLifPh8Pj76aCrTZ3zuRWgAsrOzeXDAIGZM/4A4n48xYyewevUai18CRHrf\nIx0fov+YYqA8uauhiHQG2qrqU0WunI/d1bBkxo+Ku/qV9PhB3tXwxdN7Bfw7O/Cn96L2yKUnrTpV\nnQ5M9yKWMSZyor3LHCi71NAYEzKx1sW2BGmMCZmcGEuRliCNMSFjXWxjjHERW+1HS5DGmBCyFqQx\nxriwSw2NMcaFDdIYY4yL2EqPMTofpDHGhIK1II0xIWODNMYY48KOQRpjjIvYSo+WII0xIWRdbA/l\nTvtk8S2+xT85WBfbGGNcxFaQ2r3oAAARxElEQVR6jPIEWVInjC3p8aNiwljgX3VvjUj8R7e8D0R+\n/4NhXWxjjHGhMdaGtARpjAkZa0EaY4wLG6QxxhgXsZUeLUEaY0LIWpDGGOPCjkEaY4wLG8U2xhgX\n1oI0xhgXsdaCtAlzjTHGhbUgjTEhY11sY4xxkaPWxTbGmAJpEEtRRGS0iOwUkZX5yqqJyGwRWev8\nrOqUi4iMEJF1IrJcRFrk26a3s/5aEeldnP2JuQSZlFSHz2d9yIrl81mWOpcH7r/T8zp07NCOVSsX\nkLb6KwY+1q9ExR81chjb0peRunSOp3HzC8f+d3zpbu5b8jp9Zv8zr+zih67nL4tGcPunQ7n906HU\nv6IZAL74OK55+S/0nvVP7pjzAq37dc3bpnSlclz3Rn/umPsid8x5gdotGoakfnn1jPD/vRw04KUY\nxgCdTih7Apijqo2AOc5zgGuARs7SF/gv+BMqMBi4EGgNDM5NqoWJuQSZlZXFYwOH0PS8drS9pCv3\n3tuHxo0beRbf5/MxYvhQunTtRdNmV9CzZ/cSFf+dd5Lp3CUy04RB+PZ/1YcL+Oj2l35X/v2bn/HO\nNU/zzjVPs3HeMgDO7NyauFLxjO3wJO92foZmt1xJpaTqAFz5t9vYOH85b185kLGdnmL3utBNihvp\n7x78o9iB/ivyPVUXALtPKO4GjHUejwW65yt/R/2+A6qISG2gIzBbVXer6h5gNr9Pur8Tcwly+/ad\nLE31t8QPHjxEWtpaEuvU8ix+61bNWb9+Exs3biYzM5Pk5Clc17VjiYn/5VcL2b1nr2fxThSu/U9f\n9CNH9h4s3soKCeVKI3E+4suUIjszi98O/EqpCmVJan0WK8bPByAnM5uj+w//4brlivR3D/5BmkCX\nINVU1QwA5+epTnkisCXfeulOmVt5oTxJkCJyiYg8LCIdvIiX6/TTkzi/WRMWLlrqWcw6ibXYkn6s\nVZC+NYM6HiboSMePNK/3v3nvq+k98x90fOluSlcuB8CaGYvIPHyUe1Ne4y/fvUrKyBkc2XeIyqfV\n4PDuA3Qa1pfbZjxHhxfuIqFs6ZDVJRq++2C62CLSV0RS8i19/0AVpIAyLaS8UGFJkCKyKN/ju4HX\ngIr4+/1PuG4YQuXLlyN5wigefnQwBw4U8y9/CIj8/ntQD0f2Ih0/0rzc/9R3P+fNSx9mbKenObRz\nL+0G+Q8t1Dq/ATnZObzR6gFGtX2YlndfS+XTauCLj6Nmk3qkvjuHd68dROavR2l9X9ciohRfNHz3\nwXSxVXWkqrbMt4wsRqgdTtcZ5+dOpzwdqJtvvSRgWyHlhQpXCzIh3+O+wNWqOgToABR6gCr/X5Oc\nnENBBY+Pj+fDCaMYN24Skyd/GtR7BGtregZ1k45NlZ+UWJuMjB0lJn6kebn/h3ftR3MUVFk+bh61\nz28AQONuF7Ppi+XkZGVz+Jf9bE1ZQ63zGnAgYzcHMnazPXU94G9p1mxSL2T1iYbv3sMu9idA7kh0\nb2BKvvLbndHsi4B9Thd8JtBBRKo6gzMdnLJChStB+pyKnAKIqv4MoKqHgKzCNsz/18TnKx9U8FEj\nh/FD2jpeHV6cP0ShtTgllYYN61OvXl0SEhLo0aMbU6fNKjHxI83L/S9/apW8x406tmTXj+kAHNj2\nC6ddfC4ACWVLU6dFQ35Zt43DP+/jQMZuqjaoDcDpbc/ll7VbQ1afaPjuVTXgpSgiMg74FjhLRNJF\n5E7geeBqEVkLXO08B5gBbADWAaOA+5x67Qb+Dix2lmedskKF60TxysD3+Pv9KiK1VHW7iFSg4GMB\nIdP24lbc1utGlq9YTcpi/3+OZ555nk8/mxvOsHmys7N5cMAgZkz/gDifjzFjJ7B69RpPYkdD/Pfe\nfZ3LL2tD9erV2LQhhSHP/ou3x4z3LH649r/zv/tRt01jylatwF8WjuDrlydSt01jTj3ndFBlX/ou\nZj85GoClY2fTaVhf+nz+PCLCyuQF7Erzjw/M+etYOo+4l7iEePZu3slnj4buj3ikv3sIz3yQqnqz\ny0vtC1hXgQLPb1LV0cDoQGKLx8fHyuEffdpYnPXjSyVG7OBZSb6rYKTj210No+CuhqpBNWS6ntYl\n4N/ZqZunhbXR9Ed4eqmhqh4GipUcjTEnn1ibzceuxTbGhIzdcsEYY1zE2illliCNMSFj050ZY4yL\nWDsGGXPXYhtjTKhYC9IYEzI2SGOMMS5skMYYY1xYC9IYY1zE2iCNJUhjTMjE2k27LEEaY0ImttKj\nJUhjTAjZMUhjjHERawnS0+nOAiYSxZUzJoYFOd3ZRXXaBfw7+922+TbdmTEm9sVaCzKqE2RJnTC2\npMePlglzIx2/UfUWEYm/dteSoLe103yMMcZFVB+yC4IlSGNMyFgX2xhjXFgL0hhjXFgL0hhjXMTa\nII1NmGuMMS6sBWmMCRmbrMIYY1zEWhfbEqQxJmSsBWmMMS6sBWmMMS6sBWmMMS6sBWmMMS5irQUZ\nk+dBPtj/bpalziV16Rzee/d1Spcu7Wn8jh3asWrlAtJWf8XAx/p5GjvS8ZOS6vD5rA9ZsXw+y1Ln\n8sD9d3oaHyK7/17G9vl8TJn7PiPffxWAYf99jpnfTmT6ggn8c/hfiY8/1v5pffEFfDLvA2Z8mcz7\nU0aGrU4axL9oFnMJsk6dWtzf789ceNG1nN+8PXFxcfTs0c2z+D6fjxHDh9Klay+aNruCnj2707hx\noxITPysri8cGDqHpee1oe0lX7r23T4nZf69j9+57M+vXbMp7/snET+nY5gY6X9aTMmVK06NXdwAq\nVqrAkBef4C+9HubaS3vwwJ2Ph61OqjkBL9EsLAlSRC4UkUrO47IiMkREporICyJSORwx84uPj6ds\n2TLExcVRrmxZMjK2hztkntatmrN+/SY2btxMZmYmyclTuK5rxxITf/v2nSxNXQnAwYOHSEtbS2Kd\nWp7Fj+T+exm7Vu1TaXf1JSS/Nzmv7IvPv857vGzJKmrWORWArjdcw6zpc8nY6v892L1rT1jqBP5r\nsQNdolm4WpCjgcPO4+FAZeAFp+ztMMUEYNu27bz8yhtsXL+I9M1L2bd/P7M/XxDOkMepk1iLLenb\n8p6nb82gjocJItLx8zv99CTOb9aEhYuWehYzkvvvZeynhz7Ci0OGk5Pz+xZYfHw83Xt05su53wBQ\n/4zTqFSlEu9N/h+TPn+P7j06h6VO4J/NJ9AlmoUrQfpUNct53FJVB6jqV6o6BGhQ2IYi0ldEUkQk\nJSfnUMCBq1SpzHVdO9LwzIuoe3oLypcvxy23XB/ELgRH5Pe31/DyP0Gk4+cqX74cyRNG8fCjgzlw\n4KBncSO5/17FvuLqS/nl5z2sWp5W4Ot/e/EJFn+7hJTvUgGIi4+jyXmNufuWB/lzj/vp98hd1Gtw\nWsjrBdaCLK6VInKH83iZiLQEEJEzgczCNlTVkaraUlVb+nzlAw7cvv2lbNy0mV27dpOVlcWkyZ/S\n5qKWAb9PsLamZ1A36dhU/UmJtcnI2FFi4oO/BfPhhFGMGzeJyZM/9TR2JPffq9gtLmxG+06XMe/7\nqbw66h9cdEkr/vWfvwNw/6N3U+2UqvzjmZfz1t++bScL5n7Dr4ePsGf3XhZ/u4Szm5wZ8nqBtSCL\n6y7gchFZD5wDfCsiG4BRzmths2XzVi68sAVly5YB4MorLiEtbW04Qx5ncUoqDRvWp169uiQkJNCj\nRzemTptVYuIDjBo5jB/S1vHq8PCNlrqJ5P57FXvYc69xabNrueKCrgy4+ym++2oxj973DH/q1Z1L\nr2jDQ3956rjEM+fT+bS8qDlxcXGUKVuGZi2asH7NxpDXC/yn+QS6RLOwnAepqvuAPiJSEX+XOh5I\nV9Ww/ylftHgpH388ncWLZpKVlUVq6ipGvfl+uMPmyc7O5sEBg5gx/QPifD7GjJ3A6tVrSkz8the3\n4rZeN7J8xWpSFvuTwzPPPM+nn831JH4k9z/Sn/2zLz3Jti3b+fBT/2H+WdPm8dqwUaxfu4kv537D\ntC/Gk5OTw4fvT2Zt2vqw1CHaT9sJVFTfFzu+VGLEKleS7yoY6fjRclfBSMeP6F0Ng7wvds3KZwf8\nO7tjX1rU3hc75s6DNMaYULFLDY0xIRPto9KBsgRpjAmZaD5kFwxLkMaYkIn2UelAWYI0xoSMtSCN\nMcaFHYM0xhgX1oI0xhgXdgzSGGNcxNqVNJYgjTEhYy1IY4xxEWvHIO1SQ2NMyITrnjQi0klEfhSR\ndSLyRJh3I4+1II0xIROOFqSIxAGvA1cD6cBiEflEVVeHPNgJrAVpjAmZME2Y2xpYp6obVPU3YDzg\nyZ34oroFmTvtk8W3+CUx/tpdSyIaPxhhOgKZCGzJ9zwduDA8oY4X1Qky2DnpcolIX1X1flrrCMe2\n+BY/UvGzftsa8O+siPQF+uYrGnlC3Qt6T09Gg2K9i9236FViMrbFt/iRjl9s+e9D5SwnJvZ0oG6+\n50mAJ837WE+QxpiT32KgkYjUF5FSwE3AJ14Eju4utjGmxFPVLBG5H5gJxAGjVXWVF7FjPUFG7BhQ\nhGNbfIsf6fghpaozgBlex43qm3YZY0wk2TFIY4xxYQnSGGNcxGSCjNR1m07s0SKyU0RWehk3X/y6\nIjJPRH4QkVUi8qDH8cuIyCIRWebEH+JlfKcOcSKyVESmeR3bib9JRFaISKqIpHgcu4qIfCQiac7/\ngTZexo81MXcM0rlucw35rtsEbvbiuk0n/mXAQeAdVW3iRcwT4tcGaqvqEhGpCHwPdPdw/wUor6oH\nRSQB+Ap4UFW/8yK+U4eHgZZAJVXt4lXcfPE3AS1VdVcEYo8FvlTVN51TYsqp6l6v6xErYrEFGbHr\nNgFUdQGw26t4BcTPUNUlzuMDwA/4L9XyKr6q6kHnaYKzePZXWESSgM7Am17FjBYiUgm4DHgLQFV/\ns+T4x8Rigizouk3PEkQ0EZF6QHNgocdx40QkFdgJzFZVL+O/CgwEcjyMeSIFZonI985ldF5pAPwM\nvO0cYnhTRMp7GD/mxGKCjNh1m9FERCoAE4EBqrrfy9iqmq2q5+O/JKy1iHhyqEFEugA7VfV7L+IV\noq2qtgCuAfo5h128EA+0AP6rqs2BQ4Cnx+BjTSwmyIhdtxktnGN/E4H3VfXjSNXD6d7NBzp5FLIt\ncJ1zDHA8cKWIvOdR7Dyqus35uROYhP+wjxfSgfR8LfaP8CdME6RYTJARu24zGjiDJG8BP6jqyxGI\nX0NEqjiPywJXAWlexFbVJ1U1SVXr4f/e56pqLy9i5xKR8s7gGE73tgPgyRkNqrod2CIiZzlF7QFP\nBudiVcxdahjJ6zYBRGQc0A6oLiLpwGBVfcur+PhbUbcBK5zjgABPOZdqeaE2MNY5m8AHJKtqRE63\niZCawCT/3ynigQ9U9TMP4z8AvO80DjYAd3gYO+bE3Gk+xhgTKrHYxTbGmJCwBGmMMS4sQRpjjAtL\nkMYY48ISpDHGuLAEaaKKiNSL1ExIxpzIEqTxhHNepDEnFUuQpkAi8vf8c0mKyFAR6V/Aeu1EZIGI\nTBKR1SLyhoj4nNcOisizIrIQaCMiF4jIF84kDjOdqdlwypeJyLdAP6/20ZiiWII0bt4CegM4Ce8m\n4H2XdVsDjwBNgTOA653y8sBKVb0Q/4xC/wZuVNULgNHAUGe9t4H+qmqTu5qoEnOXGprQUNVNIvKL\niDTHf/ncUlX9xWX1Raq6AfIutbwE/0QJ2fgnzQA4C2gCzHYuw4sDMkSkMlBFVb9w1nsX/yw4xkSc\nJUhTmDeBPkAt/C0+Nyder5r7/IiqZjuPBVh1YivRmdjCrnc1Ucm62KYwk/BPVdYK/+Qfblo7syf5\ngJ74b7Nwoh+BGrn3SBGRBBE515kSbZ+IXOKsd2voqm/MH2MtSONKVX8TkXnA3nwtwYJ8CzyP/xjk\nAvyJtaD3uhEY4XSr4/HP/r0K/4wzo0XkMIUnYmM8ZbP5GFdOi3AJ8CdVXeuyTjvg0UjcHMuYcLMu\ntimQiJwDrAPmuCVHY2KdtSBNsYhIU/wjzPkddU7hMSYmWYI0xhgX1sU2xhgXliCNMcaFJUhjjHFh\nCdIYY1xYgjTGGBf/DzXwVsHCI89KAAAAAElFTkSuQmCC\n",
      "text/plain": [
       "<Figure size 360x360 with 2 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "xg = xgb.XGBClassifier(n_estimators = 10)\n",
    "xg.fit(X_train,y_train)\n",
    "xg_score=xg.score(X_test,y_test)\n",
    "y_predict=xg.predict(X_test)\n",
    "y_true=y_test\n",
    "print('Accuracy of XGBoost: '+ str(xg_score))\n",
    "precision,recall,fscore,none= precision_recall_fscore_support(y_true, y_predict, average='weighted') \n",
    "print('Precision of XGBoost: '+(str(precision)))\n",
    "print('Recall of XGBoost: '+(str(recall)))\n",
    "print('F1-score of XGBoost: '+(str(fscore)))\n",
    "print(classification_report(y_true,y_predict))\n",
    "cm=confusion_matrix(y_true,y_predict)\n",
    "f,ax=plt.subplots(figsize=(5,5))\n",
    "sns.heatmap(cm,annot=True,linewidth=0.5,linecolor=\"red\",fmt=\".0f\",ax=ax)\n",
    "plt.xlabel(\"y_pred\")\n",
    "plt.ylabel(\"y_true\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 44,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "xg_train=xg.predict(X_train)\n",
    "xg_test=xg.predict(X_test)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {
    "collapsed": true
   },
   "source": [
    "### Stacking model construction"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 45,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>DecisionTree</th>\n",
       "      <th>ExtraTrees</th>\n",
       "      <th>RandomForest</th>\n",
       "      <th>XgBoost</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "      <td>5</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "      <td>3</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>2</td>\n",
       "      <td>2</td>\n",
       "      <td>2</td>\n",
       "      <td>2</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   DecisionTree  ExtraTrees  RandomForest  XgBoost\n",
       "0             5           5             5        5\n",
       "1             3           3             3        3\n",
       "2             5           5             5        5\n",
       "3             3           3             3        3\n",
       "4             2           2             2        2"
      ]
     },
     "execution_count": 45,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "base_predictions_train = pd.DataFrame( {\n",
    "    'DecisionTree': dt_train.ravel(),\n",
    "        'RandomForest': rf_train.ravel(),\n",
    "     'ExtraTrees': et_train.ravel(),\n",
    "     'XgBoost': xg_train.ravel(),\n",
    "    })\n",
    "base_predictions_train.head(5)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 46,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "dt_train=dt_train.reshape(-1, 1)\n",
    "et_train=et_train.reshape(-1, 1)\n",
    "rf_train=rf_train.reshape(-1, 1)\n",
    "xg_train=xg_train.reshape(-1, 1)\n",
    "dt_test=dt_test.reshape(-1, 1)\n",
    "et_test=et_test.reshape(-1, 1)\n",
    "rf_test=rf_test.reshape(-1, 1)\n",
    "xg_test=xg_test.reshape(-1, 1)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 47,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "x_train = np.concatenate(( dt_train, et_train, rf_train, xg_train), axis=1)\n",
    "x_test = np.concatenate(( dt_test, et_test, rf_test, xg_test), axis=1)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 48,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Accuracy of Stacking: 0.9955881055325156\n",
      "Precision of Stacking: 0.9955944258266153\n",
      "Recall of Stacking: 0.9955881055325156\n",
      "F1-score of Stacking: 0.9955625491897051\n",
      "              precision    recall  f1-score   support\n",
      "\n",
      "           0       0.99      1.00      1.00      4547\n",
      "           1       1.00      0.97      0.98       393\n",
      "           2       0.99      1.00      1.00       554\n",
      "           3       1.00      1.00      1.00      3807\n",
      "           4       1.00      0.71      0.83         7\n",
      "           5       0.99      1.00      1.00      1589\n",
      "           6       1.00      0.98      0.99       436\n",
      "\n",
      "    accuracy                           1.00     11333\n",
      "   macro avg       1.00      0.95      0.97     11333\n",
      "weighted avg       1.00      1.00      1.00     11333\n",
      "\n"
     ]
    },
    {
     "data": {
      "image/png": "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\nBz+7T3nvSrVLAAAAAElFTkSuQmCC\n",
      "text/plain": [
       "<Figure size 360x360 with 2 Axes>"
      ]
     },
     "metadata": {},
     "output_type": "display_data"
    }
   ],
   "source": [
    "stk = xgb.XGBClassifier().fit(x_train, y_train)\n",
    "y_predict=stk.predict(x_test)\n",
    "y_true=y_test\n",
    "stk_score=accuracy_score(y_true,y_predict)\n",
    "print('Accuracy of Stacking: '+ str(stk_score))\n",
    "precision,recall,fscore,none= precision_recall_fscore_support(y_true, y_predict, average='weighted') \n",
    "print('Precision of Stacking: '+(str(precision)))\n",
    "print('Recall of Stacking: '+(str(recall)))\n",
    "print('F1-score of Stacking: '+(str(fscore)))\n",
    "print(classification_report(y_true,y_predict))\n",
    "cm=confusion_matrix(y_true,y_predict)\n",
    "f,ax=plt.subplots(figsize=(5,5))\n",
    "sns.heatmap(cm,annot=True,linewidth=0.5,linecolor=\"red\",fmt=\".0f\",ax=ax)\n",
    "plt.xlabel(\"y_pred\")\n",
    "plt.ylabel(\"y_true\")\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "anaconda-cloud": {},
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.6.8"
  },
  "toc": {
   "base_numbering": 1,
   "nav_menu": {},
   "number_sections": true,
   "sideBar": true,
   "skip_h1_title": false,
   "title_cell": "Table of Contents",
   "title_sidebar": "Contents",
   "toc_cell": false,
   "toc_position": {
    "height": "calc(100% - 180px)",
    "left": "10px",
    "top": "150px",
    "width": "328px"
   },
   "toc_section_display": true,
   "toc_window_display": true
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}

