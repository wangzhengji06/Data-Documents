# End to end machine Learning Manual

## Git and Github

### Add SSH key

Follow the following link:

https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

And add your key. As simple as that.

### Basic Workflow

Create your work on Github, and git clone to local. 

Or

Create folder locally, then `git init`, Notice that your branch would be called master....

```
git remote add origin git@github.com:wangzhengji06/xx.git
git branch -M main  #This change the branch name
git push -u origin main
```

To revert the add, use `git restore --stage second_file.py`  

To revert to previous commit, `git checkout xxx` `git checkout -b xxx` `git reset --hard xxx` 

To undo commit and sync, use `git revert <commit>` and `git push`

![image-20250816181444839](C:\Users\lzabry\AppData\Roaming\Typora\typora-user-images\image-20250816181444839.png)

`git rebase -i HEAD~3` to combine the last 3 commits. A now more friendly way of this graph is to use `git fetch origin` `git checkout main` `git merge origin/main` `git checkout mybranch` `git rebase origin/main` and then solve the conflict, instead of using the git pull request. 

`git stash push -m ""` is a good way to stash 

`git tag -a v0.1.1 -m "the initial version of the package"` to tag the version of the package `git push --follow-tags` to push the tag

`git cherry-pick `  to take something only you want..



## Docker QuickStart

Please refer to the container cheatsheet. 

```
# Dockerfile
FROM python:3.8-slim

ARG DEV_MODE=0  #dynamically assign value during build time, this only works for the docker file

ENV SOME_ENV=`some value`\ #this is going to be statis
	OTHER_env = ``
			
RUN apt-get update && apt-get install -y vim gcc

COPY ./somefile.txt /

WORKDIR /

CMD["/bin/bash"]
```

`docker build -t xxx .`

**you should always put those frequently changing code at the bottom.** 

**Stash as many commands into one layer(line) as possible**



### Persistent data

```
# Dockerfile
FROM python:3.8-slim

ARG DEV_MODE=0  #dynamically assign value during build time, this only works for the docker file

ENV SOME_ENV=`some value`\ #this is going to be statis
	OTHER_env = ``
			
RUN apt-get update && apt-get install -y vim gcc

COPY ./somefile.txt /

WORKDIR /

CMD["/bin/bash"]
```

Named volume: `docker run --rm -it --name named-volumes -v my-named-volumes:/persistent-volume my_image bash` 

Bind mount: `docker run --rm -it --name bind-mounting -v .xxxx:/bind-mount my_image bash`

### Best practice in docker file

1. Do not store unique data -> Use volumes
2. Do not use latest image tags -> always define the image tag
3. Specify versions of dependency in docker file
4. Order steps from least to most frequently changing content
5. When use copy, be very specific
6. Update package information and install packages together, like `apt-get update && apt-get install vim`
7. DO not install something you don't need
8. Clean up caches
9. Use official images when possible. You do not need to do everything by your own. 

## DVC

### Data Versioning

1. `git init` `dvc init`
2. `dvc add ./data` `git add data.dvc .gitignore`
3.  `dvc remote add -d <name-of-the-storage> <storage-uri>` `dvc remote add -d gdrive gdrive://xxx`
4. add`dvc push` to push the data
5. `dvc pull` to get the data
6. `dvc add .`  and `git add . & git commit -m `  to again to stage the new data change 
7. `dvc list git@github.com ./data` for remote repo control
8. `dvc get git@github.com ./data` to just donlaod the raw data. `dvc import git@github.com ./data` for other project, which will also include images.dvc.  `dvc update` to update the data if other people's repo was updated

### DVC pipeline

An ordinary pipeline looks like this:

```
# params.yaml  this is the default file name

data:
  csv_file_path: ./archive/imdb-dataset.csv
  test_set_ratio: 0.3
  train_csv_save_path: ./archive/train.csv
  test_csv_save_path: ./archive/test.csv

features:
  vectorizer: tfidf-vectorizer
  train_features_save_path: ./archive/train.joblib
  test_features_save_path: ./archive/test.joblib

train:
  penalty: l2
  C: 1.0
  solver: lbfgs
  model_save_path: ./archive/model.joblib

evaluate:
  metric: f1_score
  results_save_path: ./archive/results.yaml
```

```
#prepare_data.py
import pandas as pd
from omegaconf import OmegaConf
from sklearn.model_selection import train_test_split


def prepare_data(config):
    print("Preparing data...")
    df = pd.read_csv(config.data.csv_file_path)
    df["label"] = pd.factorize(df["sentiment"])[0]

    test_size = config.data.test_set_ratio
    train_df, test_df = train_test_split(df, test_size=test_size, stratify=df["sentiment"], random_state=1234)

    train_df.to_csv(config.data.train_csv_save_path, index=False)
    test_df.to_csv(config.data.test_csv_save_path, index=False)


if __name__ == "__main__":
    config = OmegaConf.load("./params.yaml")
    prepare_data(config)
```

```
#make_features.py


import pandas as pd
from omegaconf import OmegaConf
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer
import joblib


def make_features(config):
    print("Making features...")
    train_df = pd.read_csv(config.data.train_csv_save_path)
    test_df = pd.read_csv(config.data.test_csv_save_path)

    vectorizer_name = config.features.vectorizer
    vectorizer = {
        "count-vectorizer": CountVectorizer,
        "tfidf-vectorizer": TfidfVectorizer
    }[vectorizer_name](stop_words="english")

    train_inputs = vectorizer.fit_transform(train_df["review"])
    test_inputs = vectorizer.transform(test_df["review"])

    joblib.dump(train_inputs, config.features.train_features_save_path)
    joblib.dump(test_inputs, config.features.test_features_save_path)


if __name__ == "__main__":
    config = OmegaConf.load("./params.yaml")
    make_features(config)
```

```
# train.py

import pandas as pd
import joblib
from sklearn.linear_model import LogisticRegression
from omegaconf import OmegaConf


def train(config):
    print("Training...")
    train_inputs = joblib.load(config.features.train_features_save_path)
    train_outputs = pd.read_csv(config.data.train_csv_save_path)["label"].values

    penalty = config.train.penalty
    C = config.train.C
    solver = config.train.solver

    model = LogisticRegression(penalty=penalty, C=C, solver=solver)
    model.fit(train_inputs, train_outputs)

    joblib.dump(model, config.train.model_save_path)


if __name__ == "__main__":
    config = OmegaConf.load("./params.yaml")
    train(config)
```

```
#evaluate.py

import joblib
import pandas as pd
from omegaconf import OmegaConf
from sklearn.metrics import accuracy_score, f1_score


def evaluate(config):
    print("Evaluating...")
    test_inputs = joblib.load(config.features.test_features_save_path)
    test_df = pd.read_csv(config.data.test_csv_save_path)

    test_outputs = test_df["label"].values
    class_names = test_df["sentiment"].unique().tolist()

    model = joblib.load(config.train.model_save_path)

    metric_name = config.evaluate.metric
    metric = {
        "accuracy": accuracy_score,
        "f1_score": f1_score
    }[metric_name]

    predicted_test_outputs = model.predict(test_inputs)

    result = metric(test_outputs, predicted_test_outputs)
    result_dict = {metric_name: float(result)}
    OmegaConf.save(result_dict, config.evaluate.results_save_path)


if __name__ == "__main__":
    config = OmegaConf.load("./params.yaml")
    evaluate(config)
```



DVC pipeline looks like this:

This sweet thing is, when you change parameters in yaml file, it it does not affect a stage, that stage would not be run again. 

```
stages:
  prepare_data:
    cmd: python ./prepare_data.py
    deps:
      - ./prepare_data.py
      - ./archive/imdb-dataset.csv
    params:
      - data
    outs:
      - ./archive/train.csv
      - ./archive/test.csv
  make_features:
    cmd: python ./make_features.py
    deps:
      - ./make_features.py
      - ./archive/train.csv
      - ./archive/test.csv
    params:
      - features
    outs:
      - ./archive/train.joblib
      - ./archive/test.joblib
  train:
    cmd: python ./train.py
    deps:
      - ./train.py
      - ./archive/train.csv
      - ./archive/train.joblib
    params:
      - train
    outs:
      - ./archive/model.joblib
  evaluate:
    cmd: python ./evaluate.py
    deps:
      - ./evaluate.py
      - ./archive/test.csv
      - ./archive/test.joblib
      - ./archive/model.joblib
    params:
      - evaluate
    metrics:
      - ./archive/results.yaml:
          cache: false
```

`dvc repro` one command to run everything

## Hydra

### Hydra from CLI

```
from omegaconf import DictConfig, OmegaConf
import hydra

@hydra.main(version_base=None, config_path=None)
def main(config:DictConfig) -> None:
    print(OmegaConf.to_yaml(config))


if __name__ =="__main__":
    main()
```

` python main.py +training.batch_size=129 +training.nrof_epcohs=30 +training.lr=5e-3` without a config

`python main.py training.batch_size=1024` is allowed for only modification



### Omegaconf

create and load config super flexible

`???` means you have to set the value for this key

It is good that it can let you select which to use, also it can call the os environment variable like `${oc.env:USER}`

It can also merge multiple config files. `config = OmegaCong.merge(config_to_merge_1, config_to_merge_2)`

### Grouping config files

Config file path has meanings, and using config path to merge the final config is a smart thing to do. 

```
defaults:
  - experiment: experiment-with-resnet18
  - _self_

experiment:
  optimizer: SGD
```

Here this `__self__` is actually referring to the experiment you later defined. 

### Multirun

`python main.py -m experiment= xx, xx, xx loss_function = xx, xx, xx`

Or you want to run everything?

`python main.py -m experiment='glob(*)' loss_function-='glob(*, exclude=soft*)'`
