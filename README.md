If you read our recent post on language detection, you already know how easy it is to use Algorithmia’s services to identify which language a given piece of text is written in.

Now let’s put that into action to perform a specific task: organizing documents into language-specific folders.

We’ll build our Python language detection microservice using Algorithmia’s language identification algorithm. Then, we’ll look through all the .txt and .docx files in a directory to see which language each one is written in.
We’ll move those files into language-specific subfolders, and print out a count of how many files were found for each language.

Step 1: Install the Algorithmia Client
While this demo is written using our Python client, our services are equally easy to use in a variety of other programming languages, or even via cURL.

Installing the Algorithmia client is simple. Just use pip to install the package:
```cmd
pip install algorithmia
```

You’ll also need a free Algorithmia account, which includes 5,000 free credits a month – enough to organize thousands of documents by language.

To get started, simply sign up, then grab your API key.

Step 2: Call the Language Detection Microservice
It only takes a couple lines of code to call any of our algorithms. Let’s wrap them up in a function:
```python
import Algorithmia
client = Algorithmia.client('your_api_key')
def detect_language(text):
"""Detect the language of a piece of text and return the ISO 639 language code"""
algo = client.algo('nlp/LanguageIdentification/1.0.0')
result = algo.pipe({'sentence':text}).result
result_sorted = sorted(result, key=lambda r: r['confidence'], reverse=True)
return result_sorted[0]['language']
```
You can see that we’re using nlp/LanguageIdentification in our call to client.algo().

We’ve also appended the most recent version number (1.0.0) to that call. We could choose to omit the version number, and call client.algo("nlp/LanguageIdentification") instead — but this could potentially break if the algorithm ever changes. By indicating a specific version number you future-proof your code.

Sending the input and getting the results back is easy, but you’ll need to know the structure of the JSON response.

Looking at the page describing the algorithm, we can see that the expected input is, at minimum, a dictionary containing the key “sentence”.

The output contains a list of dictionaries containing the values “language” (the ISO 639 language code, such as ‘en’ or ‘fr’) and a “confidence” between 0 and 1 that this is the correct language.

We could get complicated, but for now we’ll just sort the results by confidence, and return the language with the highest confidence.

Don’t forget to replace “your_api_key” with your own key, or you’ll get an “authorization required” error.

Step 3: Extract Text From a File
It’s pretty easy to read a text file’s contents in Python, but many of us keep documents in Microsoft Word format as well. Fortunately, the python-docx Python library can easily extract the text from a Microsoft Word (.docx) file:
```cmd
pip install python-docx
```

Then we can create a simple utility function to extract the text from both .txt and.docx files:
```python
import docx
def extract_text(filename):
"""Extract and return the text from a document"""
if filename.endswith('.docx'):
document = docx.Document(filename)
text = 'n'.join([
paragraph.text for paragraph in document.paragraphs
])
return text
else:
with open(filename) as f:
return f.read()
```
This will return garbage if we send it any other file type (for example, an .exe). We’ll handle that in the next step.

Step 4: Loop Through The Documents
To wrap this all together, we simply loop through all the files in some directory, passing all .txt and .docx files to the language detector.

Once we know the language of a given file, we move it into a subdirectory specific to that language code.

Lastly, we’ll print out the number of files we found in each language.
```python
from os import listdir, mkdir, path, rename
import re
def organize_files_by_language(dirname):
"""Examine all .txt and .docx files and place them in language-specific subdirectories"""
counts = {}
for filename in listdir(dirname):
if re.match('.*.txt|.*.docx', filename):
filepath = path.join(dirname, filename)
language = detect_language(extract_text(filepath))
targetpath = path.join(dirname, language)
if not path.exists(targetpath):
mkdir(targetpath)
rename(filepath, path.join(targetpath, filename))
counts[language] = counts[language]+1 if language in counts else 1
print counts
```
…and that’s all!  We now have a script which can examine all the .txt and .docx files in a directory, rearrange them into subdirectories by language code, and print out the number of files per language.

If you want to go further, you could make filetype-detection into a more robust machine learning model using filemagic, add support for reading RTF or HTML files, or loop through files in Dropbox or Amazon S3 via our Data API.

Tools used:

# Algorithmia Client for Python
# Language Identification Algorithm
# Python-Docx Package
Here’s the whole script, ready for you to cut-and-paste, or grab it (and other fun examples) from Algorithmia’s sample-apps repository on Github
```python
import Algorithmia
import docx
from os import listdir, mkdir, path, rename
import re
client = Algorithmia.client('your_api_key')
def detect_language(text):
"""Detect the language of a piece of text and return the ISO 639 language code"""
algo = client.algo('nlp/LanguageIdentification/1.0.0')
result = algo.pipe({'sentence':text}).result
result_sorted = sorted(result, key=lambda r: r['confidence'], reverse=True)
return result_sorted[0]['language']
def extract_text(filename):
"""Extract and return the text from a document"""
if filename.endswith('.docx'):
document = docx.Document(filename)
text = 'n'.join([
paragraph.text for paragraph in document.paragraphs
])
return text
else:
with open(filename) as f:
return f.read()
def organize_files_by_language(dirname):
"""Examine all .txt and .docx files and place them in language-specific subdirectories"""
counts = {}
for filename in listdir(dirname):
if re.match('.*.txt|.*.docx', filename):
filepath = path.join(dirname, filename)
language = detect_language(extract_text(filepath))
targetpath = path.join(dirname, language)
if not path.exists(targetpath):
mkdir(targetpath)
rename(filepath, path.join(targetpath, filename))
counts[language] = counts[language]+1 if language in counts else 1
print counts
organize_files_by_language('/some/file/path/')
```
