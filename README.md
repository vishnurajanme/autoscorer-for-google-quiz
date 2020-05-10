# autoscorer-for-google-quiz
google forms quiz mode does offer auto grading by it's own but that is for a single quiz. What if you want to get grades from some 10 quizzes and prepare the final grade based on the same?That is what autoscorer is for
These are the step by step process in this code
1. Authenticate as the owner of spreadsheets

```python
#A piece of code to combine all quiz scoresheets to 1
#Generate a 2nd scoresheet with the difference of old data and new data
#Authenticating its me :P
from google.colab import auth
auth.authenticate_user()
import gspread
from oauth2client.client import GoogleCredentials
gc = gspread.authorize(GoogleCredentials.get_application_default())
```

2. Importing required libraries
```python
import pandas as pd
import matplotlib.pyplot as plt
import re
```
3. Creating functions for writing data back to google sheets. You can skip this if you want to write in excel directly and use df.to_excel instead 
```python
def iter_pd(df):
    for val in df.columns:
        yield val
    for row in df.to_numpy():
        for val in row:
            if pd.isna(val):
                yield ""
            else:
                yield val
def pandas_to_sheets(pandas_df, sheet, clear = True):
    # Updates all values in a workbook to match a pandas dataframe
    if clear:
        sheet.clear()
    (row, col) = pandas_df.shape
    cells = sheet.range("A1:{}".format(gspread.utils.rowcol_to_a1(row + 1, col)))
    for cell, val in zip(cells, iter_pd(pandas_df)):
        cell.value = val
    sheet.update_cells(cells)
```
4. Loading the sheets
```python
#Reading 3 sheets 1st. 1 sheet for storing the full scoresheet till data (olddata), 1 sheet for storing only passed cases
#and 1 sheet for storing the difference between olddata and newdata with a whether passed or not mask.
#This sheet is further linked with autocrat for certificate generation. I have made this difference sheet just coz,
#if I do not do so, autocrat will send mails to all in the list once a new person has completed the course.
directout = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx')
worksheet = directout.sheet1
directout1 = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx')
passandfail = directout1.sheet1
difference = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx')
diff = difference.sheet1
#reading my full data I already have
qq = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx').sheet1.get_all_values()
olddata = pd.DataFrame.from_records(qq[1:],columns=qq[0])
olddata.set_index("Email Address", inplace = True)
#loading all sheets
qq1 = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx').sheet1.get_all_values()
qq2 = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx').sheet1.get_all_values()
qq3 = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx').sheet1.get_all_values()
qq4 = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx').sheet1.get_all_values()
qq5 = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx').sheet1.get_all_values()
qq6 = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx').sheet1.get_all_values()
qq7 = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx').sheet1.get_all_values()
qq8 = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx').sheet1.get_all_values()
qq9 = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx').sheet1.get_all_values()
qq10 = gc.open_by_url('xxxxxxxxxxyoursheeturlherexxxxxxxxxxx').sheet1.get_all_values()
#converting sheets to df
q1 = pd.DataFrame.from_records(qq1[1:],columns=qq1[0])
q2 = pd.DataFrame.from_records(qq2[1:],columns=qq2[0])
q3 = pd.DataFrame.from_records(qq3[1:],columns=qq3[0])
q4 = pd.DataFrame.from_records(qq4[1:],columns=qq4[0])
q5 = pd.DataFrame.from_records(qq5[1:],columns=qq5[0])
q6 = pd.DataFrame.from_records(qq6[1:],columns=qq6[0])
q7 = pd.DataFrame.from_records(qq7[1:],columns=qq7[0])
q8 = pd.DataFrame.from_records(qq8[1:],columns=qq8[0])
q9 = pd.DataFrame.from_records(qq9[1:],columns=qq9[0])
q10 = pd.DataFrame.from_records(qq10[1:],columns=qq10[0])
```
5. Setting Email address as the index so that I can merge scores based on email (Merging can happen only with same indices)
```python
q2.set_index("Email Address", inplace = True)
q3.set_index("Email Address", inplace = True)
q4.set_index("Email Address", inplace = True)
q5.set_index("Email Address", inplace = True)
q6.set_index("Email Address", inplace = True)
q7.set_index("Email Address", inplace = True)
q8.set_index("Email Address", inplace = True)
q9.set_index("Email Address", inplace = True)
q10.set_index("Email Address", inplace = True)
```

6. Creating base df from quiz1 and copying only values needed for creating certificate
```python
df = q1[['Email Address','Full Name','Institution','Score']].copy()
df.set_index("Email Address", inplace = True)
```

7. Appending scores from other sheets to the end of this df
```python
df['Score2'] = q2[['Score']]
df['Score3'] = q3[['Score']]
df['Score4'] = q4[['Score']]
df['Score5'] = q5[['Score']]
df['Score6'] = q6[['Score']]
df['Score7'] = q7[['Score']]
df['Score8'] = q8[['Score']]
df['Score9'] = q9[['Score']]
df['Score10'] = q10[['Score']]
```

8. data cleaning scores
```python
for col in df.columns[2:]:
  df[col] = df[col].str.split(re.escape(' ')).str[0] + ''
  df = df.fillna(0)
  df[col]=pd.to_numeric(df[col])
```

9. Final score calculation
```python
col_list = list(df)
col_list.remove('Full Name')
col_list.remove('Institution')
df['Total'] = df[col_list].sum(axis=1)
#Setting cutoff to 50percent
df.loc[(df['Score'] >= 50) & (df['Score2'] >= 50) & (df['Score3'] >= 50) & (df['Score4'] >= 50) & (df['Score5'] >= 50) & (df['Score6'] >= 50) & (df['Score7'] >= 50) & (df['Score8'] >= 50) & (df['Score9'] >= 50) & (df['Score10'] >= 50) ,'Verdict'] = 'Passed'
pandas_to_sheets(df, passandfail)
newdata = df[(df['Verdict'] == 'Passed')]
```

10. calculating difference between old data and new data
```python
newonly = newdata[~newdata.astype(str).apply(tuple, 1).isin(olddata.astype(str).apply(tuple, 1))]
#For debugging
with pd.option_context('display.max_rows', None, 'display.max_columns', None):
  #print(newdata)
  #print(olddata)
  print(newonly)
#Saving data to 2 separate spreadsheets
```
11. updating olddata sheet and difference sheet for autosending certificates
```python
newdata = newdata.reset_index()
pandas_to_sheets(newdata, worksheet)
newonly = newonly.reset_index()
pandas_to_sheets(newonly, diff)
```
