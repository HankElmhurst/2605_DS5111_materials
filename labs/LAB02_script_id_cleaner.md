# LAB 2 Youtube ID Cleaner
Write a script that takes can process a list of youtube ids as part of a linux pipeline.  For example if you have a file with the youtube ids named `weekly_youtube_ids`, and your script is named `clean_ids.py`, we should be able to call it with `cat weekly_youtube_ids | clean_ids.py` and the result should be output at the console of the cleaned ids.

## Intuition
Imagine a project to parse websites and scrub them for data.  You 'might' receive the html of the websites.  As a Data Scientist you might be positioned somewhere in the pipeline where you will process a stack of html files.  However, in a traditional data pipeline it is very common that the originating data is just a list of urls.  Setting this up is usually the domain of Data Engineers, but not always.  So in order to get a sense of what the full data pipeline looks like we'll build this script and in a later lab we'll set up a simple 'stage', a folder where a list of youtube ids can be dropped off and automation will trigger the processing.

For this lab, our focus will be to create the cleaning script, test it and have it ready for use when we assemble the pieces.

## Script description
You'll write the script in python.  You could write it in bash, but writing it in python also highlights the fact that in order to leverage the linux pipe feature you don't need to write bash.

You'll rely on `sys.stdin`, `sys.stderr` and `sys.stdout` or `print`.  That will give you input/output you need to plug your script into the pipeline.

## Requirements

### Definition of a valid youtube id
* Length: Exactly 11 characters.
* Character Set: Uses a modified Base64 encoding consisting of the following 64 possibilities:
    - Uppercase letters (A-Z)
    - Lowercase letters (a-z)
    - Numbers (0-9)
    - Hyphen (-)
    - Underscore (_)


Create a script named `clean_ids.py`
```
Given a file with youtube ids named youtube_ids
When I cat youtube_ids into clean_ids.py
Then I should only see valid youtube ids at the command line
    And there should be a log file named pipeline_autid.log created indicating any errors

Given the file test_ids with the contents
     abcd
     CctJNYYCPo0
     1234
When I cat the file at clean_ids.py
Then I should only see CctJNYYCPo0 at the command line
    And the log should indicate abcd and 1234 were not valid ids

Given the script clean_ids.py
When I call it directly with 'python3 clean_ids.py'
Then the command line should wait for input
    And when I type 1234 I should see no echo
    And when I type CctJNYYCPo0, I should see it echoed
    And when I use CTL-C I should return to the command line
```

## Grading
* 10 points possible as long it works as per the Given/When/Then statements.

## **NOTES:**
For this lab **please DO NOT** use any LLMs for help with the code.  We'll work with these later in the course, but for this assignment it's important to get familiar with a couple of points so you can reuse them later.
* Using sys.stdout and sys.stdin...
* Adding logging to your files.  For this, you can/should google 'how to add logging to my script' or just look at the python documentation.

The hardest part is going to be the error checking.  The point here, is that if you do get stuck, or are unsure, create a pull request and send it to me or the TA so we can start communicating via github as you would in a real work setting.  We'll help you get it running, **the point is not to see how fast you can get it running, but to use the process of writing code in github and cycling feedback**

