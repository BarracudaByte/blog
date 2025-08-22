# Working with CSVs in Python

![Programmer](https://raw.githubusercontent.com/BarracudaByte/blog/refs/heads/main/images/working_with_csvs.webp?raw=true)

**CSV** (Comma-Separated Value) files can be found nearly everywhere. Every data in a tabular format can usually be exported as a CSV. Opening these files in Excel or Google Sheets works usually for small datasets, but parsing them manual can become tedious and error-prone. Automating CSV parsing, can save time, reduce mistakes, and open further possibilities for automation.

Python offers some great packages like the `csv` module to work with CSVs in just a few lines of code. In this blog post I will cover, how to read CSV files effortlessly using Python and why its best to use libraries.

## CSV Structure

CSV files are technically just text files, which means you can open them with any text editor and we could just read them line by line in Python, but this is were it gets a bit tricky. 

The cell contents in each lines are usually separated by a delimiter (which can be different for each CSV file, but in one file it is always the same). The problem is, when the delimiter appears inside the cell content itself. 

### Delimiters

For example, if the CSV delimiter is `,` (comma) and the CSV contains three columns: To, From, and Subject a line could look like this:

```csv
from@sender.com,to@recipient.com,and a subject
```

On this line, we could simply use `line.split(',')` and we would have a list of each value, but what is the subject itself would contain a comma?

```csv
from@sender.com,to@recipient.com,"and a subject, with comma!"
```

In these cases CSV values are being wrapped in a quote character. As the name suggested these are often quotes, but technically it could be any character. Now the `split()` function would fail, which is why we will make use of the `csv` module.

## Using the CSV Module

With the `csv` module you can avoid issues with delimiters and quote characters and super quickly parse CSV files in Python. 


### reader() vs DictReader()

There are two main functions to read CSV files:

- `csv.reader()`: returns each row as a list
- `csv.DictReader()`: uses the first row as the header row and creates a dictionary for each following row, where the keys are the column names of the first row

For example, when using the `csv.reader()` and printing each line, the result would look like the following:

```python
['timestamp', 'sender_email', 'recipient_email', 'subject', 'status', 'delivery_time_ms']
['2024-08-15 10:01:23', 'sender1@example.com', 'recipient1@example.com', 'Meeting Schedule', 'SENT', '150']
```

With the `csv.DictReader()` it would instead look like this:

```python
{'timestamp': '2024-08-15 10:01:23', 'sender': 'sender1@example.com', 'recipient': 'recipient1@example.com', 'subject': 'Meeting Schedule', 'status': 'SENT', 'delivery_time_ms': '150'}
{'timestamp': '2024-08-15 10:05:45', 'sender': 'sender2@example.org', 'recipient': 'recipient2@example.net', 'subject': 'Project Update', 'status': 'FAILED', 'delivery_time_ms': '0'}
```

To use either of the functions simply open the file normally and pass it as first argument. Further arguments can be, for example, `delimiter`, `quotechar`, or the `dialect`. After that we can simply iterate over the results:

```python
import csv

with open(file_path, encoding='utf-8-sig') as file:
    csv_data = csv.DictReader(csv_file, delimiter=',')
    for row in csv_data:
        cell_content = row['Column Name']
```

### Common Issues with Encoding

In the above example the file encoding is hard-coded to use `utf-8-sig`. While this will likely work in most of the cases, you may run into the issue that your CSV file uses a different encoding. In case your CSV file always come from different sources ypu may have different encodings and you need to dynamically determine the encoding.

To dynamically detect the encoding of a file you can use a package like `chardet`. `chardet` has a `detect` function that gives you an encoding and a confidence level.

```python
with open(file_path, 'rb') as file:
    print('Detected: ', chardet.detect(file.read()))
    # prints for example: Detected:  {'encoding': 'ascii', 'confidence': 1.0, 'language': ''}
```

## Using Pandas

An alternative to the `csv` module is the `pandas` module. Anyone working in data sciene or machine learning will know this one, and if you are trying to get into these fields, this module is a good one to know!

First install `pandas` with `pip`.
```bash
pip install pandas
```

Opening a CSV file with `pandas` is even easier than with `csv`:

```python
import pandas

dataframe = pandas.read_csv(file_path)
```

That's it! Pandas does a lot of heavy lifting in the background, like automatically inferring data types, handling missing value, and dealing with headers. 

For example run `data.frame.info()` and it will tell you how many entries there are in the CSV (without header), what the columns are and what their type is. 

Some useful functions are:
```python
dataframe['Column Name'] # returns a single column from the DataFrame, which returns a Pandas Series

df.dropna() # remove rows or columns containing missing values (NaN).

df.fillna(value) # replace missing values with the specified value

for index, row in dataframe.iterrows():
        print(f'Index: {index}, row: {row}')
        # Accessing specific value in row
        print(row['Column Name'])
```

## CSV vs Pandas

For simple scripts I often find the `csv` module more straight-forward as you don't have to deal with the `dataframe` object, which is very powerful but sometimes it needs some getting used to. Use the `csv` module with small datasets where you don't care about performance and where you don't worry too much about the format of the data.

On the other hand, use the `pandas` module, when you have big datasets and or need a good performance. As shown in the example also really shines when dealing with missing values.

## Conclusion

No matter what library you go for, reading CSV files in Python is easy and you only need a few lines of code. The `csv` module is perfect for quick, lightweight tasks without external dependencies while `pandas` is quicker and perfect for big datasets.
