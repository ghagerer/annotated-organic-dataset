# annotated-organic-dataset
The dataset was crawled in late 2017 from Quora with the search terms being "organic", "organic food", "organic agriculture", and "organic farming" are used. The comments deal with organic food or agriculture and discussed characteristics, advantages, and disadvantages of organic food production and consumption. Each sentence has sentiment (positive, negative, neutral) and entity, the sentiment target, annotated.


## Description ##

### Source

The dataset was crawled in late 2017 from Quora, a social question-and-answer website. To retrieve relevant articles from the platform, the search terms "organic", "organic food", "organic agriculture", and "organic farming" are used. The texts are deemed relevant by a domain expert if articles and comments deal with organic food or agriculture and discussed characteristics, advantages, and disadvantages of organic food production and consumption. 
From the filtered data, 1,373 comments are chosen with 10,439 sentences to be annotated.

### Annotation Scheme

Each sentence has sentiment (positive, negative, neutral) and entity, the sentiment target, annotated. We isolate sentiments expressed about organic against non-organic entities, whereas for classification only samples annotated as organic entity are considered. Consumers discuss organic or non-organic products, farming practices, and companies.

### Annotation Procedure

The data is annotated by each of the 10 coders separately; it is divided into 10 batches of $1,000$ sentences for each annotator and none of these batches shared any sentences between each other. 
4616 sentences contain organic entities with 39% neutral, 32% positive, and 29% negative sentiments.
After annotation, the data splits are 80% training, 10% validation, and 10% test set. The data distribution over sentiments, entities, and attributes remains similar among the splits.



## Pre-Processing

Terminology:

- a comment is relevant when it contains at least one relevant sentence
- a sentence is relevant when it is annotated as relevant

Tasks:

- relevance classification of comments
  - all comments from the Annotated Organic Dataset
- aspect-based sentiment analysis on sentences of relevant comments
  - only relevant comments from the Annotated Organic Dataset
  - there is an additional target "n/a" for not applicable
  - the F1 score must be reported with and without consideration of the "n/a" class
- aspect-based sentiment analysis only on relevant sentences
  - only relevant sentences from the Annotated Organic Dataset


## Calculating Loss and F1 Scores

The annotations imply a *multi-label multi-class* classification problem, see [here how the F1 score calculation works](https://scikit-learn.org/stable/modules/multiclass.html) and [here how to calculate the loss in Keras](https://www.depends-on-the-definition.com/classify-toxic-comments-on-wikipedia).

Since you cannot use softmax, you need to find the optimal threshold for the activations of your output neurons on the training set. Therefore, for each output, you [calculate the ROC curve](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.roc_curve.html) which gives you the rates for true positives and false positives as well as according thresholds. To find the optimal threshold for that very output, choose ```thresholds[argmax(tpr-fpr)]```. This you have to do for each output separately.

Please use the [F1 score function from sklearn](https://scikit-learn.org/stable/modules/classes.html#module-sklearn.metrics). For less than 6 classes, please use macro F1 score. For more than 10 classes, use micro F1 score, for the cases in between report both.

The gold standard as well as the predictions which you pass as numpy array parameter for ```y_true``` and ```y_pred``` to these functions should follow the following exepmlified format/dimensions without the headers of course:


| class1 | class2 | class3 | class4 |
|--------|--------|--------|--------|
|   true |  false |   true |  false |
|  false |   true |  false |  false |
|  false |  false |  false |  false |
|   true |   true |   true |   true |

Classes for aspect-based sentiment analysis can be one of the following options:

- triplets of (attribute, entity, sentiment)
- tuples of (attribute, entity), i.e., aspect classes
- only attributes for attribute extraction
- only entities for entitiy detection
- only sentiment for sentiment analysis

In order to obtain these Numpy arrays, you might want to use [MultiLabelBinarizer](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MultiLabelBinarizer.html) as it is exemplified by Omar Shouman [Jupyter Notebook](https://gitlab.lrz.de/ga89mis/thesis_omar/blob/master/F1_score/F1_sklearn.ipynb). Please note that this now assumes a multi-label classification problem (one sentence has multiple annotation labels). The following is simpler and might work, too:

```python
#Provide the column name for the column that is needed to be multi-hot-encoded
categoryColumn = "" 

multiHotEncoded = pd.get_dummies(dataframe[categoryColumn], dummy_na=True)
 
# To get it as a numpy array (Including the additional NaN class)
multiHotEncoded.values

# To get it as a numpy array and drop the introduced NaN class 
multiHotEncoded.drop(columns=multiHotEncoded.columns[-1]).values
```

### Single-Label Problem ####

Whenever possible, please treat the problem as a multi-label multi-class problem. If multi-label is not possible, you have to reduce the multi-label problem to a single-label problem. Consequentially, you have to remove annotations in the case you have multiple annotations per sample, i.e., sentence, in order to come up with only one annotation per sample. In that case, please remove all samples from the dataset which have more than one annotation.

## Annotations

### Descriptions of the Annotation Classes ###

[This is the file](https://syncandshare.lrz.de/dl/fi7p5CV5CCyZeoCXGLexgfna/Labeling_Workshop_updated_18-10-19.xlsx) provided to all annotators containing detailed descriptions of what the aspects etc actually mean.

### Entities

Mapping from dataset letters to human-readable entities

```python
od_entity_mapping = {
	'g': 'organic general',
	'p': 'organic products',
	'f': 'organic farmers',
	'c': 'organic companies',
	'cg': 'conventional general',
	'cp': 'conventional products',
	'cf': 'conventional farming',
	'cc': 'conventional companies',
	'gg': 'GMOs genetic engineering general'										
}
```

### Attributes
Mapping from dataset letters to human-readable attributes

```python
od_attribute_mapping = {
	'g': 'general',
	'p': 'price',
	't': 'taste',
	'q': 'Nutr. quality & freshness',
	's': 'safety',
	'h': 'healthiness',
	'c': 'chemicals pesticides',
	'll': 'label',
	'or': 'origin source',
	'l': 'local',
	'e': 'environment',
	'av': 'availability',
	'a': 'animal welfare',
	'pp': 'productivity'
}
```

### Sentiment
Mapping from dataset letters to human-readable sentiment categories

```python
od_sentiment_mapping = {
	'0': 'neutral',
	'p': 'positive',
	'n': 'negative'
}
```

### Coarse dataset categories
Since there are a lot of entity-attribute combinations there is a coarse entity-attribute combination which is defined in the annotation guidelines.
The following two dictionaries can be used to create the coarse categories:

```python
od_coarse_entities = {
	'g': 'organic',
	'p': 'organic',
	'f': 'organic',
	'c': 'organic',

	'cg': 'conventional',
	'cp': 'conventional',
	'cf': 'conventional',
	'cc': 'conventional',

	'gg': 'GMO'
}

od_coarse_attributes = {
	'g': 'general',
	'p': 'price',
	
	't': 'experienced quality',
	'q': 'experienced quality',

	's': 'safety and healthiness',
	'h': 'safety and healthiness',
	'c': 'safety and healthiness',

	'll': 'trustworthy sources',
	'or': 'trustworthy sources',
	'l': 'trustworthy sources',
	'av': 'trustworthy sources',

	'e': 'environment',
	'a': 'environment',
	'pp': 'environment',
}
```

The following function creates a mapping from fine-grained to coarse grained entity-attribute labels.
You can use the entity-attribute value (e.g. `p-t`) to get the new coarse category tuple (e.g. `organic-experienced quality`) for that sample.

```python
mapping = {}
for entity_key in od_entity_mapping.keys():
	for attribute_key in od_attribute_mapping.keys():
		c_ent = od_coarse_entities[entity_key]
		c_att = od_coarse_attributes[attribute_key]
		compound_key = f'{entity_key}-{attribute_key}'
		mapping[compound_key] = f'{c_ent}: {c_att}'
# add a 'missing aspect' key
mapping[''] = ''

mapping['p-t']
# yields: organic-experienced quality
```


