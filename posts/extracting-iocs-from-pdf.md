# Automatically extracting IOCs from PDF files

![Programmer](https://raw.githubusercontent.com/BarracudaByte/blog/refs/heads/main/images/extract_iocs_from_pdf.webp?raw=true)

PDF files have been around since the early 1990s and since 2008 it is an open standard published by ISO (International Organization for Standardization). Nowadays, PDF files are integrated into many different workflows and systems, like invoices or reports. Today, PDF is one of the most widely used document formats in the world, which makes it the perfect format for phishing attacks.

According to [AAG IT](https://aag-it.com/the-latest-phishing) a substantial majority of cyber attacks, commence with a phishing email, from which 36% had an attachment. PDF files are great for the inital attack, as most security tools don't block those emails and users don't generally suspect them as being malicious. 

In this blog we will look at automatically parsing PDF files and extracting IOCs (Indicators of Compromise). 

## Understanding PDF Structure

Before working with PDF files in Python, it is important to understand the general structure of a PDF. If you open any PDF file in a text editor you will in general see a lot of gibberish but there will be some readable parts in between usually looking similar to the following snippet:
```pdf
1 0 obj
<< /Type /Page /Parent 2 0 R /Resources 4 0 R /Contents 3 0 R /MediaBox [0 0 768.24 1023.84]
/Annots 11 0 R >>
endobj
```

Granted it's not the most clear text one can come accross, but it shows nicely the general structre of PDF files. Each PDF file starts with a header the indicates the version of the PDF specification that is used, for example `%PDF-1.3`. After that follows the body which contains a series of objects, usually in the format `<obj-number> <version-number> obj`. Each object can include additional information using a forward slash. 

For example, for an object to define it's type as page, it would have something like `/Type /Page`. The most interesting type for automation is probably the `/Annots` (**Annotations**), these annotations can have various properties, depending on its type (e.g.: text notes, links, highlights). A embedded link would be in a `/Annots`, then followed by a `/A` (**Action**), and then be in the `/URI`.

Part of this is shown in this next code snippet. Object `12` is a `/A` type which references object `44`(the number directly after `/A`). Object `44` is of type `/URI` but it again just references object `45` which then finally contains the actual URI.

```
12 0 obj
<< /A 44 0 R /Border [ 0 0 0 ] /Type /Annot /Subtype /Link /Rect [578.2188 803.34 615.4688 830.84]
>>
endobj
44 0 obj
<< /Type /Action /S /URI /URI 45 0 R >>
endobj
45 0 obj
(https://www.barracudabyte.de/blog)
```

## Parsing PDFs with Python

There are lots of different Python libraries to parse PDF, for this use case we will use the following two:

- **PDFium2**: This library tries to hide the underlying PDF structure, and instead concentrates on easily inspect and manipulate PDFs ([PDFium2 on PyPip](https://pypi.org/project/pypdfium2/))
- **PikePDF**: Thislibrary keeps the structure was we have seen above and represents it in Python ([PikePDF on PyPip](https://pypi.org/project/pikepdf/))

We will use **PDFium2** to extract the text of the PDF and some general information, but then use **PikePDF** for URLs and emails.


### Extracting general information & text from PDFs with PDFium2

In an analysis maybe not always necessary but **PDFium2** can give us some general information about the PDF, like the version and the number of pages. 

First open the PDF file and pass it to `pdfium` to create a new document.

```python
import pypdfium2 as pdfium

with open(filepath, "rb") as fh:
    pdf = pdfium.PdfDocument(fh)
    version = pdf.get_version()
    num_pages = len(pdf)
    print(f'Version: {version}\nPages: {num_Pages}')

```

Now we can also extract the text of the PDF, as **PDFium2** tries to hide the underlying strcuture, this mostly only extracts the visible text. Simply iterate over each page in the PDF document and get the text. 

```python
# still inside the *with*
text = [page.get_textpage().get_text_bounded() for page in pdf]
# We can also join it to one str instead of a list
full_text = '\n'.join(text)
```

Now we could use a regex to, for example, extract phone numbers or email addresses that are not embedded as URIs.

```python
# update imports
import re

# regex for phone and email addresses 
PHONE_REGEX = r'[\+\(]?[1-9][0-9 .\-\(\)]{8,}[0-9]'
EMAIL_REGEX = r'\S+@\S+\.\S+'

# inside the *with* again
# extract phone numbers from visible text
matched_phone_nums = re.findall(PHONE_REGEX, full_text)
print(f'Phone Numbers (in text): {matched_phone_nums}')

# extracting email from visible text
matched_emails = re.findall(EMAIL_REGEX, full_text)
print(f'Email Addresses (in text): {matched_emails}')
```

This all works great as long as there are no actual clickable links, as those are often not printed directly but embedded. For those we can use the understanding we have now with **PikePDF**.


### Extracting URIs from PDFs with PikePDF

Similar to **PDFium2** we can simply pass the filehandle of the file to **PikePDF** to create a new PDF object. Instead of just getting the text, we can then use this to iterate over each page, find all `/Annots` and then check if that has a `/A` and then check if that again has a `/URI`.

```python
# imports
from urllib.parse import urlparse
import re
import pikepdf


# inside *with*
pdf = pikepdf.Pdf.open(fh)
for page in pdf.pages:
    for annots in page.get('/Annots', []):
        url = annots.get('/A', {}).get('/URI')
        
```

Once we have a potential URI we can use `urllib.parse` to parse the URL and quickly check the schema. If the URI starts with `mailto` we can assume it is an email, if it starts with `tel` a phone number, and otherwise it is a normal URL. We could of course confirm this with a regex but we will keep this simple for now. 

```python
if url:
    url = str(url)
    parsed_url = urlparse(url)
    # Email address from Annotation
    if parsed_url.scheme == 'mailto':
        print(f'Email Addresses: {parsed_url.netloc or parsed_url.path}')
        continue
    # Phone number from Annotation
    if parsed_url.scheme == 'tel':
        print(f'Phone Numbers: {parsed_url.netloc or parsed_url.path}')
        continue

    # Otherwise its just a normal URI
    print(f'URL: {url}')
```

## Conclusion

PDFs don't have the easiest file structure, but Python has some great libraries that can make it very easy to extract information from them. Extracting IOCs automatically from PDFs can save a lot of time from analysts who would otherwise have to manually review the attachment and reviewing every link. 

The code of course can be improved on, for example, by better validating the URL / Domain or checking other PDF objects for interesting data. 