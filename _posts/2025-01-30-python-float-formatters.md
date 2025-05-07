---
layout: post
title: random code tidbits
date: 2025-01-30 11:00:00
description: useful to have on hand
tags: code
categories: 
featured: false
---

Random bits of code I reuse every so often. 

<!-- ````markdown
```c++
code code code
```
```` -->
## Python Function Tidbits
Formatting a Python float with a given number of significant figures,
```python
number = 1.2345
print(f"Number formatted to 2 decimal places: {number:.2f}")
```
Writing a string to file, 
```python
my_string = "lie_groups"
with open("Output.txt", "w") as f:
    f.write(my_string)
```
Read/write with pickle, 
```python
test_dict = {"test": 1}
with open('euler.pickle', 'wb') as f:
    pickle.dump(test_dict, f, protocol=pickle.HIGHEST_PROTOCOL)

with open('euler.pickle', 'rb') as f:
    b = pickle.load(f)
```
Number formats: 
```python
".1e" #: scientific notation with 1 decimal point (standard form)
".2f" #: 2 decimal places
".3g" #: 3 significant figures
".4%" #: percentage with 4 decimal places
```
with more details <a href="https://docs.python.org/3/library/string.html#formatspec">here</a>. 
## Plotting
Plotting timestamps as vertical lines, 
```python 
import pandas as pd
plt.figure()
ax: plt.Axes = plt.gca()
ax.vlines([stamps_rel_pos], label="Measurement stamps", ymin=0, ymax=1, colors="red")
```
## Pandas Dataframe Manipulation
Nicely formatting a multilevel pandas dataframe to input into a paper. 
Given a dataframe with columns Dims and Method, as well as some metrics of interest, 
we create a table that breaks down these metrics by Dims and Method. 

```python 
import pandas as pd


def highlight_min(data: pd.DataFrame, highlight_max=False):
    """
    func : function
        ``func`` should take a Series if ``axis`` in [0,1] and return a list-like
        object of same length, or a Series, not necessarily of same length, with
        valid index labels considering ``subset``.
        ``func`` should take a DataFrame if ``axis`` is ``None`` and return either
        an ndarray with the same shape or a DataFrame, not necessarily of the same
        shape, with valid index and columns labels considering ``subset``.
    """
    attr = "textbf:--rwrap;"
    ndim = data.ndim
    # ndim = len(data.index.names)
    if ndim == 1:  # Series from .apply(axis=0) or axis=1
        if highlight_max is False:
            is_min = data == data.min()
        else:
            is_min = data == data.max()
        ret_val = [attr if v else "" for v in is_min]
    else:
        if highlight_max is False:
            is_min = data.groupby(level=0).transform("min") == data
        else:
            is_min = data.groupby(level=0).transform("max") == data
        ret_val = pd.DataFrame(
            np.where(is_min, attr, ""), index=data.index, columns=data.columns
        )
    return ret_val


def format_multiindexed_df(
    df_metric: pd.DataFrame,
    floating_point_format_dict: Dict,
    column_format="|*{7}{c|}",
    min_columns=None,
    max_columns=None,
    drop_columns=["Dims", "NEES", "Method", "Run Name", "Case"],
):

    df_metric = df_metric.copy()
    if drop_columns is not None:
        df_metric.drop(columns=drop_columns, inplace=True)

    if min_columns is not None:
        df_styled: pd.DataFrame.style = df_metric.style.apply(
            highlight_min, axis=None, subset=min_columns
        )
    df_styled: pd.DataFrame.style = df_styled.apply(
        highlight_min, axis=None, subset=max_columns, highlight_max=True
    )

    df_styled = df_styled.format(floating_point_format_dict)

    latex_string = df_styled.to_latex(column_format=column_format, hrules=True)
    latex_string = latex_string.replace(
        "\\\n\end{tabular}", "\\ \hline \n\end{tabular}"
    )
    latex_string = latex_string.replace(
        "\\\n\end{tabular}", "\\ \hline \n\end{tabular}"
    )

    # latex_string = latex_string.replace("{r}", "{c|}")
    latex_string = re.sub("\\\.*rule", "\\\hline", latex_string)
    print(latex_string)

index = pd.MultiIndex.from_frame(df[["Dims", "Method"]])
df = df.set_index(index)
df = df.rename(
    columns={
        "Avg Iterations": "Avg Iter.",
    }
)
print(df)
format_multiindexed_df(
    df,
    {
        "RMSE (deg)": "{:,.2f}".format,
        "RMSE (m)": "{:,.2f}".format,
        "ANEES": "{:,.2f}".format,
        "Time (s)": "{:,.2f}".format,
        "Avg Iter.": "{:,.2f}".format,
    },
    min_columns=["RMSE (deg)", "RMSE (m)", "ANEES", "Avg Iter.", "Time (s)"],
    max_columns=None,
)

```

### Cleaning up a LaTeX paper repository before submitting to arXiv
Removes unnecessary auxiliary files before uploading LaTeX code to arXiv. 
Be careful not to run this in the original repo, but rather a copy. 

```bash 
# DO NOT RUN THIS IN THE ORIGINAL. 
# RUN AFTER MOVING EVERYTHING TO THE NEW SUBMISSION DIRECTORY
#!/bin/bash/
rm main.log
rm main.pdf 
rm main.run.xml
rm main.synctex.gz 
rm main-blx.bib
rm main.aux
rm main.blg
rm -rf .git
rm -rf .vscode
rm .gitignore
rm main.fdb_latexmk
rm main.fls
rm clean_up_for_arxiv_v2.sh # Name of current script
```