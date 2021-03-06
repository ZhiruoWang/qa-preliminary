# Data Pre-processing and Result Reproduction

## Environment Configuration
```bash
conda env create --file scripts/env.yml             # create a new environment called 'tab'
conda activate tab
pip install --editable .                            # utilize the 'setup.py'
conda env update --name tab --file data/env.yml     # update the 'tab' env with nsm libraries
```

For Pengcheng-processed WikiTable data, follow
```bash
wget http://www.cs.cmu.edu/~pengchey/pytorch_nsm.zip
unzip pytorch_nsm.zip -d data
```

Install necessary python libraries
```bash
pip install -r preprocess/requirements.txt
python -m spacy download en_core_web_sm
```


## Pre-Training Data
### Web Tables (WDC)
Refer to this [website](http://data.dws.informatik.uni-mannheim.de/webtables/2015-07/englishCorpus/compressed/). 
For example:
```bash
mkdir -p data/datasets
wget http://data.dws.informatik.uni-mannheim.de/webtables/2015-07/sample.gz -P data/datasets
gzip -d < data/datasets/sample.gz > data/datasets/commoncrawl.sample.jsonl
```
This results with a 'commoncrawl.sample.jsonl' file in the 'data/preprocessed_data' directory.

Further, for a WDC subfolder(00):
```bash
cd data/datasets
wget http://data.dws.informatik.uni-mannheim.de/webtables/2015-07/englishCorpus/compressed/00.tar.gz -P ./
tar -xvf 00.tar.gz     # results a '00.tar' file
tar -xvf 00.tar        # results a './0/' folder, having 1M single json files
cat ./0/*.json > ./wdc00.jsonl   # may not work, use python then
python -m preprocess.common_crawl --worker_num 10 --input_file ./data/datasets/wdc00/0-0.jsonl --output_file ./data/preprocessed_data/wdc00-0.jsonl
```

### Wiki Table (Common Crawl)
Refer to this [website](https://dumps.wikimedia.org/enwiki/) for the latest dump.
More details lies in the 'scripts/extract_wiki_tables.sh'.
```bash
bash scripts/extract_wiki_tables.sh
```
This creates a file called 'wiki_tables.jsonl', also in the 'data/preprocessed_data' directory.


## Reproduce TaBERT Results (on QA)
First, download the TaBERT models follow
```bash
cd models

pip install gdown

# TaBERT_Base_(k=1)
gdown 'https://drive.google.com/uc?id=1-pdtksj9RzC4yEqdrJQaZu4-dIEXZbM9'

# TaBERT_Base_(K=3)
gdown 'https://drive.google.com/uc?id=1NPxbGhwJF1uU9EC18YFsEZYE-IQR7ZLj'
```

```bash
bash train_vanilla_bert.sh    # or 'train_tabert_k1/3.sh'

python -m table.experiments test --cuda    \
--model=runs/vanilla_bert/model.best.bin   \
--test-file=data/wikitable/wtq_preprocess_0805_no_anonymize_ent/test_split.jsonl
```

Current Results:
| Model | Dev. Acc. | Test Acc. |
| :---: | :-------: | :-------: |
| Vanilla BERT | 47.19 / 64.85 | 47.86 / 61.19 |
| TaBERT (k=1) | --.-- / --.-- | 48.20 / 61.53 |
| TaBERT (k=3) | --.-- / --.-- | 49.68 / 62.62 |
