`actionable-recourse` is a python library to check recourse in linear classification models. 

### [Recourse](https://github.com/ustunb/actionable-recourse/blob/master/README.md#recourse) | [Installation](https://github.com/ustunb/actionable-recourse/blob/master/README.md#installation) | [Development](https://github.com/ustunb/actionable-recourse/blob/master/README.md#development) | [Reference](https://github.com/ustunb/actionable-recourse/blob/master/README.md#reference)

## Recourse

*Recourse* is the ability to flip the prediction of a ML model by changing *actionable* input variables (e.g., `income` instead of `age`).

#### When should models provide recourse?

In tasks such as lending, ML models should provide all individuals with an actionable way to change their prediction. In other tasks, models should let individuals flip their predictions based on specific types of changes. For example, a recidivism prediction model that includes `age` should allow every person who is predicted to recidivate with the ability to flip their prediction without altering their `age`.

#### Highlights

The tools in this library let you check recourse without interfering in model development. 

They can answer questions like:

- What can a person do to obtain a favorable outcome from a model?
- How many people can change their prediction?
- How difficult for people to change their prediction?

**Functionality**

- Customize the set of feasible action for each input variable of an ML model.
- Produce a list of actionable changes for a person to flip the prediction of a linear classifier.
- Estimate the *feasibility of recourse* of a linear classifier on a population of interest.
- Evaluate the *difficulty of recourse* for a linear classifier on a population of interest.

----
### Usage
```
import pandas as pd
import numpy as np
from sklearn.linear_model import LogisticRegression
import recourse as rs

# import data
url = 'https://raw.githubusercontent.com/ustunb/actionable-recourse/master/examples/paper/data/credit_processed.csv'
df = pd.read_csv(url)
y, X = df.iloc[:, 0], df.iloc[:, 1:]

# train a classifier
clf = LogisticRegression()
clf.fit(X, y)
yhat = clf.predict(X)

# customize the set of actions
A = rs.action_set.ActionSet(X)  ## matrix of features. ActionSet will learn default bounds and step-size.

# specify immutable variables
A['Married'].mutable = False

# can only specify properties for multiple variables using a list
A[['Age_lt_25', 'Age_in_25_to_40', 'Age_in_40_to_59', 'Age_geq_60']].mutable = False

# education level
A['EducationLevel'].step_direction = 1  ## force conditional immutability.
A['EducationLevel'].step_size = 1  ## set step-size to a custom value.
A['EducationLevel'].step_type = "absolute"  ## force conditional immutability.
A['EducationLevel'].bounds = (0, 3)

A['TotalMonthsOverdue'].step_size = 1  ## set step-size to a custom value.
A['TotalMonthsOverdue'].step_type = "absolute"  ## discretize on absolute values of feature rather than percentile values
A['TotalMonthsOverdue'].bounds = (0, 100)  ## set bounds to a custom value.

## get model coefficients and align
A.align(clf)  ## tells `ActionSet` which directions each feature should move in to produce positive change.

# Get one individual
i = np.flatnonzero(yhat <= 0)[0]

# build a flipset for one individual
fs = rs.flipset.Flipset(x = X[i], action_set = A, clf = clf)
fs.populate(enumeration_type = 'distinct_subsets', total_items = 10)
fs.to_latex()
fs.to_html()

# Run Recourse Audit on Training Data
auditor = rs.auditor.RecourseAuditor(A, coefficients = w, intercept = b)
audit_df = auditor.audit(X)  ## matrix of features over which we will perform the audit.

## print mean feasibility and cost of recourse
print(audit_df['feasible'].mean())
print(audit_df['cost'].mean())
```

----
## Installation

The latest release can be installed via pip by running:

```
$ pip install actionable-recourse
```

### Requirements

- Python 3
- CPLEX or [Pyomo](http://www.pyomo.org/) + [CBC](https://projects.coin-or.org/Cbc) 
 
#### CPLEX

CPLEX is fast optimization solver with a Python API. It is commercial software, but free for students and faculty at accredited institutions. To obtain CPLEX:

1. Register for [IBM OnTheHub](https://ibm.onthehub.com/)
2. Download the *IBM ILOG CPLEX Optimization Studio* from the [software catalog](https://ibm.onthehub.com/WebStore/ProductSearchOfferingList.aspx?srch=CPLEX)
3. Install the CPLEX Optimization Studio.
4. Setup the CPLEX Python API [as described here](https://www.ibm.com/support/knowledgecenter/SSSA5P_12.8.0/ilog.odms.cplex.help/CPLEX/GettingStarted/topics/set_up/Python_setup.html).

If you have problems installing CPLEX, please check the [CPLEX user manual](http://www-01.ibm.com/support/knowledgecenter/SSSA5P/welcome) or the [CPLEX forums](https://www.ibm.com/developerworks/community/forums/html/forum?id=11111111-0000-0000-0000-000000002059). 

#### CBC + Pyomo

If you do not want to use CPLEX, you can also work with an open-source solver. In this case, complete the following steps before you install the library:

1. Download and install [CBC](https://github.com/coin-or/Cbc) from [Bintray](https://bintray.com/coin-or/download/Cbc)
2. Download and install `pyomo` *and* `pyomo-extras` [(instructions)](http://www.pyomo.org/installation)

## Contributing

We're actively working to improve this package and make it more useful. If you come across bugs, have comments, or want to help, let us know. We welcome any and all contributions! For more info on how to contribute, check out [these guidelines](https://github.com/ustunb/actionable-recourse/blob/master/CONTRIBUTING.md). Thank you community!

#### Roadmap

- support for categorical variables in `ActionSet`
- support for rule-based models such as decision lists and rule lists
- [scikit-learn](http://scikit-learn.org/stable/developers/contributing.html#rolling-your-own-estimator) compatability
- [integration into AI360 Fairness toolkit](https://www.ibm.com/blogs/research/2018/09/ai-fairness-360/)

## Reference

For more about recourse and these tools, check out our paper:

[Actionable Recourse in Linear Classification](https://arxiv.org/abs/1809.06514)

```
inproceedings{ustun2019recourse,
     title = {Actionable Recourse in Linear Classification},
     author = {Ustun, Berk and Spangher, Alexander and Liu, Yang},
     booktitle = {Proceedings of the Conference on Fairness, Accountability, and Transparency},
     series = {FAT* '19},
     year = {2019},
     isbn = {978-1-4503-6125-5},
     location = {Atlanta, GA, USA},
     pages = {10--19},
     numpages = {10},
     url = {http://doi.acm.org/10.1145/3287560.3287566},
     doi = {10.1145/3287560.3287566},
     publisher = {ACM},
}
```
