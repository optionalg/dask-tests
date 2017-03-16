---
title       : Dask examples for DC
description : This is gonna be sooooo goooooooood.
attachments :
  slides_link : https://s3.amazonaws.com/assets.datacamp.com/course/teach/slides_example.pdf

---
title: Dask Tests
description: Dask exercising in DataCampLand.

--- type:NormalExercise lang:python xp:100 skills:2 key:3efae752e4
## Dask delayed I

This is a typical example using `dask.delayed`. It is drawn from material provided by Continuum in [this notebook](https://github.com/datacamp/courses-dask/blob/nbs/Notebooks/babynames.ipynb). It seems to work!

More on dask delayed [here](http://dask.pydata.org/en/latest/delayed.html).

*** =instructions

*** =hint

*** =pre_exercise_code
```{python}

```

*** =sample_code
```{python}
# Imports
import glob
import zipfile
import requests
import io
from dask import delayed
import pandas as pd

# File
url = 'https://137.200.4.16:443/oact/babynames/names.zip'
r = requests.get(url, verify=False)

z = zipfile.ZipFile(io.BytesIO(r.content))
z.extractall(path='./babynames')

# Do some dask stuff
dfs = []
for year in glob.glob("babynames/yob*.txt"):
    df = delayed(pd.read_csv)(year, header=None, names=['name','sex','count'])
    year = year.split('yob')[1].replace('.txt','')
    df = delayed(df.assign)(year = year)
    dfs.append(df)
    
# Inspect the first file
print(dfs[0].compute().head())

```

*** =solution
```{python}

```

*** =sct
```{python}

```

--- type:NormalExercise lang:python xp:100 skills:2 key:e70c65db40
## Dask delayed II

This follows up from the previous exercise to use dask delayed to do some computation: compute the
number of babies named Khaleesi as a function of year.

The line `df = df.set_index('year')` (which is needed) times out on DataCamp. There's discussion about this with the experts
in [an issue in our repo](https://github.com/datacamp/courses-dask/issues/6). Let's definitely discuss this!

*** =instructions

*** =hint

*** =pre_exercise_code
```{python}
import glob
import zipfile
import requests
import io
from dask import delayed
import pandas as pd

# File
url = 'https://137.200.4.16:443/oact/babynames/names.zip'
r = requests.get(url, verify=False)

z = zipfile.ZipFile(io.BytesIO(r.content))
z.extractall(path='./babynames')

# Do some dask stuff
dfs = []
for year in glob.glob("babynames/yob201*.txt"):
    df = delayed(pd.read_csv)(year, header=None, names=['name','sex','count'])
    year = year.split('yob')[1].replace('.txt','')
    df = delayed(df.assign)(year = year)
    dfs.append(df)
    
# Inspect the first file
#print(dfs[0].compute().head())
```

*** =sample_code
```{python}
# Import
import dask.dataframe as dd

# 
df = dd.from_delayed(dfs, meta=dfs[0].compute())
#df = df.set_index('year') #this times out the DC exercise,  even on 5 years
#df = df.persist() #
df = df.repartition(npartitions=4)

#
print(df.loc[df['name']=='Khaleesi', 'count'].compute()) #this times out, even on 5 years

#Note: including the lines that time out here, the above code runs locally for me in < 3seconds

```

*** =solution
```{python}

```

*** =sct
```{python}

```



--- type:NormalExercise lang:python xp:100 skills:2 key:5e86ee67ac
## Dask delayed visualization

Dask delayed has a cool way of visualizing the chain of calculations. This doesn't work in DataCamp
but, in the case in the code here, will look like this:

![Check it out!](https://s3.amazonaws.com/assets.datacamp.com/production/course_3541/datasets/Screenshot 2017-03-16 13.49.30.png)

it's the line `total.visualize()` that would be great to execute.

[This example is from here](https://github.com/dask/dask-tutorial/blob/master/02_foundations.ipynb).

*** =instructions

*** =hint

*** =pre_exercise_code
```{python}

```

*** =sample_code
```{python}
from dask import delayed

@delayed
def inc(x):
    return x + 1

@delayed
def add(x, y):
    return x + y
    
x = inc(15)
y = inc(30)
total = add(x, y)

#total.visualize() # doesn't work in DC
total.compute()


```

*** =solution
```{python}

```

*** =sct
```{python}

```


--- type:NormalExercise lang:python xp:100 skills:2 key:e898ac4292
## Dask distributed

The 2nd big aspect of `dask` that we'll use in the course is dask distributed. Thisis an example exercise. There are several ways to do this. See more in the [issue mentioned in a previous exercise](https://github.com/datacamp/courses-dask/issues/6): Matt Rocklin states an option:

> Don't have them create their own cluster with Client(), but rather have DC engineers stand up dask-scheduler and dask-worker processes for them (this is easy) and instead tell them to connect to "the cluster" at `Client('localhost:8786')`

*** =instructions

*** =hint

*** =pre_exercise_code
```{python}
import glob
import zipfile
import requests
import io

# File
url = 'https://137.200.4.16:443/oact/babynames/names.zip'
r = requests.get(url, verify=False)

z = zipfile.ZipFile(io.BytesIO(r.content))
z.extractall(path='./babynames')
```

*** =sample_code
```{python}
from distributed import LocalCluster, Client
import dask.dataframe as dd

cluster = LocalCluster(nanny=False, n_workers=4, threads_per_worker=1)
c = Client(cluster)
df = dd.read_csv('babynames/yob18*.txt', header=None, names=['name','sex','count'])
df = df.set_index('name')
print(df.head())
```

*** =solution
```{python}

```

*** =sct
```{python}

```