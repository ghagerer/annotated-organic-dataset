# Organic Dataset

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

## Annotations

![annotation-distribution](https://user-images.githubusercontent.com/875050/116995733-ba194b80-acda-11eb-8ed6-acd000e072c4.png)


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


