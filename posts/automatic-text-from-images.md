# Automatically Analysing Phishing SMS by Extracting Text from Images

![Programmer](https://raw.githubusercontent.com/BarracudaByte/blog/refs/heads/main/images/text-from-images.webp?raw=true)

Automatically analysing phishing emails is often discussed and probably every single SOAR tool has an in-built workflow to do exactly that. Most tools though, don't have a specific flow to automatically analyse suspicious text messages, as these are often just send as images to the security team. 

Working of an image instead of having all the information in an `eml` is definitely a restriction, but a big part can still be automated. In this blog post, we will look specifically at how to extract text from images taken of a potentially malicious text message.

## Extracting Text from Images with Tesseract

**Tesseract**, besides being a four-dimensional hypercube in geometry, is an open source **OCR** (Optical Character Recognition) engine. It supports more than 100 languages and can run on different image formats, which makes it perfect to extract text from images with it.

[Tesseract](https://github.com/tesseract-ocr/tesseract) is written in C++ but there are wrappers for all sorts of languages like `pytesseract` for Python and `tesseract` or `tesseract-rs` for Rust. In this blog we will look at the Python library as this is the most common language for automation.

### Installing pytesseract

Installing `pytesseract` is simple and straight-forward, simply run the following command:

```bash
pip install pytesseract
```

To work with images in Python, it is often the easiest to sue the `Pillow` library. `pytesseractt` has in-built support for common image types, but not all of them. To avoid any issues with unusal image types it's easier to use `Pillow` and pass its object to `pytesseract`.

```bash
pip install pillow
```

Usually with just the pip package you should be good to go. In case you are running this, for example, on a docker image for a remote agent, you may run into some issues though. The reason is, the Python package relies on **Tesseract** being installed on your system. Depending on your setup you can install **Tesseract** either via pre-built binary package or build it from source. Both ways are outlined on **Tesseracts** GitHub page [here](https://github.com/tesseract-ocr/tesseract?tab=readme-ov-file#installing-tesseract). You also may need to install the [Leptonica](http://leptonica.com/) Library.

### Extracting Text from Images with PyTesseract

Let's start by importing `Pillow` (which package name is `PIL`) and `pytesseract`.

```python
from PIL import Image

import pytesseract
```

Now we can just load the image with `Pillow` and then extract the text with `pytesseract` with two simple calls:

```python
img = Image.open('test/sms_phish.png')

extracted_text = pytesseract.image_to_string(img)

print("Extracted Text:\n", extracted_text)
```

Depending obviously on the image you are using the output could look similar to this:

```text
Extracted Text:
(ALERT) Your Credit Card
has been temporarily
suspended. To unlock
your account, click here
http://
tdcanadatrustloginpage.
com/cc/
```

### Find URLs in Text

At first, this may seem like a straight-forward task, just use a regex on the extracted text and get all the URLs. The problem starts with the regex, which is prefectly summarised in the blog "In search of the perfect URL validation regex" on [mathiasbynens.be](https://mathiasbynens.be/demo/url-regex). The gernal problem is, that URLs can be structured in lots of different ways making it difficult to find the perfect regex. 

Using, for example, [Imme Emosols](https://gist.github.com/731338/810d83626a6d79f40a251f250ed4625cac0e731f) regex, we can extract most of the URLs we should find.

```python
# import re to use regex
import re

# regex to find URLs by Imme Emosol
URL_REGEX = r'(?:(?:https?|ftp)://)(?:\S+(?::\S*)?@)?(?:(?:[1-9]\d?|1\d\d|2[01]\d|22[0-3])(?:\.(?:1?\d{1,2}|2[0-4]\d|25[0-5])){2}(?:\.(?:[1-9]\d?|1\d\d|2[0-4]\d|25[0-4]))|(?:(?:[a-z\u00a1-\uffff0-9]+-?)*[a-z\u00a1-\uffff0-9]+)(?:\.(?:[a-z\u00a1-\uffff0-9]+-?)*[a-z\u00a1-\uffff0-9]+)*(?:\.(?:[a-z\u00a1-\uffff]{2,})))(?::\d{2,5})?(?:/[^\s]*)?'

# use regex on extracted text
print("Found URLs:", re.findall(URL_REGEX, extracted_text))
```

This may or may not find the URL, not necessarily because of the regex but because often URLs in phishing SMS are split into mulitple lines. We can try to remedy that by simply removing all newline characters, which will then correctly find the URL.

```python
print("Found URLs:", re.findall(URL_REGEX, extracted_text.replace('\n', '')))
```

Removing all newline characters can, in rare cases, cause the issue of text after the URL being appanded to the URL. To avoid this a more carefull approach to replace could be taken, by only removing `\n` if it is after a `/` or a `.` which are usually the places a URL is split, however, this may not always work. 

```python
extracted_text.replace('/\n', '/').replace('.\n', '.').replace('\n', ' ')
```

## Why not use AI?

Setting up and using `pytesseract` is normally easy and straightforward, but the question may stil persist: "Why not use AI directly?". Obviously AI may be able to directly extract the URLs from the images and potentially deal even better with broken up URLs but using AI may cause potentially new attack vectors. An example of this would be the prompt inject attacks discovered by researches from [Trails of Bits](https://blog.trailofbits.com/2025/08/21/weaponizing-image-scaling-against-production-ai-systems/) in August 2025, where images could contain hidden prompts to mislead AI and this likely won't be the last way of misusing AI. Therefore, especially in security using AI always needs to be used with caution.

With that in mind, technically, the images of phishing SMS should always come from internal users, that likely won't try to add a injected prompt to it. Nevertheless it is worth considering when setting up a workflow like this, especially if it can also be achieved with **Tesseract**.

## Conclusion

Phishing SMS may not be as common as phishing emails but are still important to quickly triage and respond to. Automation allows to quickly extract text and URLs from an image of a text message which can then be used for further automation, to quickly respond to these cases. 