# Cornea-NLP

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

The repository provides the source code to reproduce the results of the paper "Natural Language Processing to Quantify Microbial Keratitis Measurements" (*currently under review*). The data are not posted because they contain protected health information.

Abstract: A natural language processing (NLP) algorithm to extract microbial keratitis morphology measurements from the electronic health record (EHR) was 75-96% sensitive and 91%-96% specific. NLP is an accurate method to quantity data from free-text fields.

Authors: Nenita Maganti BS, Huan Tan, Leslie M. Niziol MS, Sejal Amin MD, Andrew Hou MD, Karandeep Singh MD, Dena Ballouz, Maria A. Woodward MD MSc

Financial support: This work was supported by a grant from the National Institutes of Health, Bethesda, MD (Woodward; Clinical Scientist Award K23EY023596). The funder had no role in study design, data collection and analysis, decision to publish, or preparation of the manuscript.  

Conflict of interest: The authors have no proprietary or commercial interest in any of the materials discussed in this article.

## Overview

The project has a series of well defined stages in the pipeline.To reproduce,you will have to:

* **Clean** the source data to remove incorrect,inaccurate,irrelevant parts of the data.
* **Expand** the Abbreviations.
* **Correct** the misspellled words in the sentences.
* **Represent** the words in the appropriate format.


## Usage 

### Requirements
* Python 3

**Example 1**

Run the measure_size function to extract **both** the maximum defect and infiltrate size measure from the corneal examination results. The code was run on the corneal examination results provided in the example file.
``` python
def measure_size(sents,re_float=re_float, rgx=[ed_rgx,inf_rgx]):
    '''
        input:
            sents: string of sentences
            rgx: regex pattern of 0 cases
            re_float: regex matching any numerical tokens
        output: dictionary of valid measurements at encounter level, both defect and infiltrate
    '''
    ed_rgx = rgx[0]
    inf_rgx = rgx[1]
    ed_size=measure_max(sents, 'defect',re_float=re_float, rgx=ed_rgx).get('defect')
    st_size=measure_max(sents, 'stain',re_float=re_float, rgx=ed_rgx).get('stain')
    if ed_size==(None,None) and st_size!=(None,None):
        ed_size=st_size
    inf_size=measure_max(sents, 'infiltrat',re_float=re_float, rgx=inf_rgx).get('infiltrat')
    uc_size=measure_max(sents, 'ulcer',re_float=re_float, rgx=inf_rgx).get('ulcer')
    if inf_size==(None,None) and uc_size!=(None,None):
        inf_size=uc_size
    sents = sents.replace("'","")
    if re.findall(r'defect[\s\w*]*\,? [\w*\s]*with[\s\w*]* infiltrate', sents)!=[] and ed_size==(None,None) and inf_size!=(None,None):
        ed_size=inf_size
    if re.findall(r'defect[\s\w*]*\,? [\w*\s]*with[\s\w*]* infiltrate', sents)!=[] and ed_size!=(None,None) and inf_size==(None,None):
        inf_size=ed_size
    if re.findall(r'infiltrate[\s\w*]*\,? [\w*\s]*with[\s\w*]* defect', sents)!=[] and ed_size!=(None,None) and inf_size==(None,None):
        inf_size=ed_size
    if re.findall(r'infiltrate[\s\w*]*\,? [\w*\s]*with[\s\w*]* defect', sents)!=[] and ed_size==(None,None) and inf_size!=(None,None):
        ed_size=inf_size
    return{'defect':ed_size, 'infiltrate': inf_size}


```

The python command to execute the function:
```
measure_size(<Corneal Examination report sentence>,<Regular expression numerical cases>,<regular expression for 0 cases>)
```



**Example 2**

Run the measure_max function to extract **either** the maximum defect measure for "defect" or "infiltrate" measure. This function is used internally by the measure_size function collect both defect and infiltrate measures. The code was run on the corneal examination results provided in the example file.

```python
def measure_max(sents, kwd, rgx,re_float=re_float):
    '''
        input:
            sents: string of sentences
            kwd: keyword you are looking for, ex: defect, infiltrate
            rgx: regex pattern of 0 cases
            re_float: regex matching any numerical tokens
        output: dictionary for the numerical measures related to kwd at encounter level
    '''
    measures = measure(sents, kwd,rgx, re_float=re_float)
    if measures == None:
        return {kwd: (None,None)}
    measures = apply_f(measures,eval)
    if len(measures)==1 and type(measures[0])==list:
        return {kwd: tuple(measures[0])}
    elif len(measures)>=2:
        mean = list(map(stat.mean,measures))
        idx = mean.index(max(mean))
        return {kwd: tuple(measures[idx])}
    else:
        return {kwd: (None,None)}
```

The python command to execute the function:

```
measure_max(<Corneal Examination report>,<keyword:"defect" or "infiltrate">,<Regular expression pattern of 0 cases>, <Regular expression matching numerical tokens>)
```

Screenshot of the output generated by the function:

<img width="683" alt="Screen Shot 2019-05-29 at 12 47 15 PM" src="https://user-images.githubusercontent.com/33362260/58650725-75f6c280-82dd-11e9-9626-10cc8714d01d.png">


