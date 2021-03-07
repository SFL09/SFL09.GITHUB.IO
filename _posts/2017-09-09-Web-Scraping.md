---
layout: post
title: "Web Scraping - SEC EDGAR"
description: "Extracted large amounts of data from SEC EDGAR."
categories: [DataScience]
tags: [SEC EDGAR, Web Scraping]
redirect_from:
  - /2017/09/09/
---

## Web Scraping

Explored the SEC EDGAR website for all firms’ 10-Ks included in the Dow Jones Industrial Average filed during the calendar year 2016; determined and tabulated the following information for each filing:

- The number of words in the overall 10-K filing;
- The number of words in the filing’s “Management Discussion and Analysis” (MD&A) section;
- The number of times the word “competition” is mentioned in the MD&A section;
- The ten words before and after the date in the MD&A section.


### Load Web Page

```python
from urllib import request
from bs4 import BeautifulSoup

def load_web_page(url):
    soup = BeautifulSoup(request.urlopen(url), 'html.parser')
    page = soup.text

    for char in '\n.,()':
        page = page.replace(char,' ')

    return page
```

### Parse Data

#### 0. MD&A section

```python
pre = (
    'Management’s Discussion and Analysis of '
    'Financial Condition and Results of Operations'
).lower()
suf = (
    'Quantitative and Qualitative Disclosures About Market Risk'
).lower()
text_MDA = text[text.lower().rfind(pre)+len(pre) : text.lower().rfind(suf)]
```

#### 1. The number of words in the overall 10-K filing

```python
data['num_of_words'] = len(text.split())
```

#### 2. The number of words in the filing’s “Management Discussion and Analysis” (MD&A) section

```python
data['words_in_MDA'] = len(text_MDA.split()) - 2
```

#### 3. The number of times the word “competition” is mentioned in the MD&A section

```python
data['competition_in_MDA'] = text_MDA.lower().count('competition')
```

#### 4. The ten words before and after the date in the MD&A section

```python
from re import compile, IGNORECASE

p = compile(
    (
        '(((Jan(uary)?|Mar(ch)?|May|Jul(y)?|Aug(ust)?|Oct(ober)?|Dec(ember)?)( |-|\/)(30|31))| '
        '((Apr(il)?|Jun(e)?|Sep(tember)?|Nov(ember)?)( |-|\/)30)|'
        '((Jan(uary)?|Feb(ruary)?|Mar(ch)?|Apr(il)?|May|Jun(e)?|Jul(y)?|Aug(ust)?|Sep(tember)?|Oct(ober)?|Nov(ember)?|Dec(ember)?)'
        '( |-|\/)((0|1|2)?[0-9])))|(((0?1|0?3|0?5|0?7|0?8|10|12)(-|\/)(30|31))|((0?4|0?6|0?9|11)(-|\/)30)|'
        '((0?1|0?2|0?3|0?4|0?5|0?6|0?7|0?8|0?9|10|11|12)(-|\/)((0|1|2)?[0-9])))(-|\/)(\d{2}(\d{2})?)'
    ),
    flags=IGNORECASE,
)
data['date_text'] = []
for m in p.finditer(text):
    pre = text[m.start()-400:m.start()].split()
    suf = text[m.end():m.end()+400].split()
    data['date_text'].append({
        'dat': m.group(),
        'pre': pre[-10:],
        'suf': suf[:10]
    })
```

#### Data parsing function

```python
from re import compile, IGNORECASE

def get_data_dict(text, name, time):
    data = {'name': name, 'time': time}

    # MD&A section
    pre = (
        'Management’s Discussion and Analysis of Financial Condition and '
        'Results of Operations'
    ).lower()
    suf = 'Quantitative and Qualitative Disclosures About Market Risk'.lower()
    text_MDA = text[text.lower().rfind(pre) + len(pre) : text.lower().rfind(suf)]

    # The number of words in the overall 10-K filing
    data['num_of_words'] = len(text.split())

    # The number of words in the filing’s “Management Discussion and Analysis” (MD&A) section
    data['words_in_MDA'] = len(text_MDA.split()) - 2

    # The number of times the word “competition” is mentioned in the MD&A section
    data['competition_in_MDA'] = text_MDA.lower().count('competition')

    # The ten words before and after the date in the MD&A section
    p = compile(
    (
        '(((Jan(uary)?|Mar(ch)?|May|Jul(y)?|Aug(ust)?|Oct(ober)?|Dec(ember)?)( |-|\/)(30|31))| '
        '((Apr(il)?|Jun(e)?|Sep(tember)?|Nov(ember)?)( |-|\/)30)|'
        '((Jan(uary)?|Feb(ruary)?|Mar(ch)?|Apr(il)?|May|Jun(e)?|Jul(y)?|Aug(ust)?|Sep(tember)?|Oct(ober)?|Nov(ember)?|Dec(ember)?)'
        '( |-|\/)((0|1|2)?[0-9])))|(((0?1|0?3|0?5|0?7|0?8|10|12)(-|\/)(30|31))|((0?4|0?6|0?9|11)(-|\/)30)|'
        '((0?1|0?2|0?3|0?4|0?5|0?6|0?7|0?8|0?9|10|11|12)(-|\/)((0|1|2)?[0-9])))(-|\/)(\d{2}(\d{2})?)'
    ),
        flags=IGNORECASE
    )
    data['date_text'] = []
    for m in p.finditer(text):
        pre = text[m.start()-400:m.start()].split()
        suf = text[m.end():m.end()+400].split()
        data['date_text'].append({
            'dat': m.group(),
            'pre': pre[-10:],
            'suf': suf[:10]
        })

    return data
```

### Main Function

```python
from csv import reader
from json import dumps

def main():
    dataset = []

    with open('TASK1.csv', 'r') as csv:
        spamreader = reader(csv)
        next(spamreader, None) # Skip the headers

        for row in spamreader:
            print(row[0], end='')
            # Ignore the empty URL
            if row[3] == '':
                print('... [skip]')
                continue

            # Extract and save data
            page = load_web_page(row[3])
            data = get_data_dict(page, row[0], row[2])
            dataset.append(data)
            print('... [done]')
   
    with open('dataset.json', 'w') as f:
        f.write(dumps(dataset))
```

```python
if __name__ == '__main__':
    main()
```

### JSON Sample Output
```json
[
    {
        "name":"The 3M Company ",
        "time":"2/9/2017",
        "num_of_words":76096,
        "words_in_MDA":19001,
        "competition_in_MDA":1,
        "date_text":[
            {
                "dat":"December 31",
                "pre":[
                    "Firm",
                    "54",
                    "Consolidated",
                    "Statement",
                    "of",
                    "Income",
                    "for",
                    "the",
                    "years",
                    "ended"
                ],
                "suf":[
                    "2016",
                    "2015",
                    "and",
                    "2014",
                    "55",
                    "Consolidated",
                    "Statement",
                    "of",
                    "Comprehensive",
                    "Income"
                ]
            },
            {
                "dat":"December 31",
                "pre":[
                    "55",
                    "Consolidated",
                    "Statement",
                    "of",
                    "Comprehensive",
                    "Income",
                    "for",
                    "the",
                    "years",
                    "ended"
                ],
                "suf":[
                    "2016",
                    "2015",
                    "and",
                    "2014",
                    "56",
                    "Consolidated",
                    "Balance",
                    "Sheet",
                    "at",
                    "December"
                ]
            },
            {
                "dat":"December 31",
                "pre":[
                    "31",
                    "2016",
                    "2015",
                    "and",
                    "2014",
                    "56",
                    "Consolidated",
                    "Balance",
                    "Sheet",
                    "at"
                ],
                "suf":[
                    "2016",
                    "and",
                    "2015",
                    "57",
                    "2",
                    "Table",
                    "of",
                    "Contents",
                    "BeginningPage",
                    "ITEM"
                ]
            },
            ...
        ]
    }
]
```
