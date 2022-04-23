# Introduction

## Goals

_First Impressions_ is an exploratory project that aims to answer the following two sets of questions (though many more arose throughout the process):

+ What do internet users find when searching for "transgender"?
	+ How does the website or platform used influence the results?

+ How can Digital Humanities scholars approach archiving search results?
	+ What ethical and practical questions shoulb be raised in this type of research?

## Project Overview

First Impressions is an attempt to address these questions using machine learning technology. Fives instances of GPT-2 will be created, with each being trained using text from a different website. The training documents will be composed of the first pages of results from JSTOR, Reddit, YouTube, CBC.ca and BBC.co.uk. Each instance will then generate a short piece of prose informed by an algorithmic analysis of the language and writing style of its source material. These texts will be displayed on an interactive website created with Twine.

## Programs Used

+ Acrobat (for Optical Character Recognition)
+ ffmpeg (for editing and converting audio files)
+ Google Speech-to-Text API (attempted, unsuccesfull)
+ grepWin (editing text files)
+ GPT-2
+ OBS Studio
+ Otter AI
+ yt-dlp

# Process

## Step 1: Collecting and Preparing Input Data

All input data was collected on 27 March 2022. All searches were done in Firefox's "In-Private" mode while connected to a virtual private network (VPN) based in Toronto, Ontario, Canada. Firefox and the VPN were both restarted before conducting each search. Initial data collection was started on 19 March, but a decision was made to start over and collect all input information during the same 24-hour period.

### Reddit

Reddit scrapers were researched, but all appear to focus on scraping from a specific subreddit or user with no capacity for searches. The Reddit API is available to premium users, but I do not have a premium account and would require additional coding knowledge.

"Transgender" was searched from the Reddit homepage and the first fifty results were utilized. Default search settings (Sort: By Relevance, Time: Any Time) were used. If a post contained text, the entirety of its content was used. The first five parent comments when filtered by "best" (the default setting) were used, but the responses to those comments were not. If any of these parent comments were from subreddit moderators or from bots, they were ignored.

### JSTOR 

JSTOR does not make their API public, which means there is no easily accesible scraper. Using a generic web scraping tool would work, but this process would have to be conducted one article at a time and would take comparably as long as manual collection.

"Transgender" was searched from the JSTOR homepage and the first page of results were utilized (24 in total). Default search settings (Sort by: Relevance) were used. If an article was open access, I immediately gathered the text. If there was a paywall, I searched for it in another browser where I was logged into JSTOR through my university library. Appendices, citations and front matter were not included. If a result was an ebook, only the introduction was used.

If an article did not have OCR functionality (Optical Character Recognition), I utilized the OCR tool within Adobe Acrobat. 

One entry was not available through my university library, so I utilized the first result from the second page as an alternative.

### Youtube

"Trasgender" was searched from the Youtube homepage and the first fifty results were utilized. Default search settings were used. The video urls were gathered to use with yt-dlp using the following script:

```yt-dlp --skip-download --write-auto-subs --sub-lang en --convert-subs srt [(url(s)]```

This created .srt caption files, which were converted to .txt with ffmpeg. I then used grepWIN to delete the time stamps by searching for and deleting all lines that began with a number.

### CBC

No scraper is available for CBC.ca.

"Transgender" was searched from the CBC.ca homepage and the first fifty results were utilized. Default search settings (Category: All, Media: All) were used. Results included written articles, radio broadcasts, and one video. Since the radio broadcasts and video were not downloadable or scrapeable, I played the first five minutes of each and captured my desktop using OBS Studio. 

I decided to try Google's Speech-to-Text API since Mozilla's Project Deepspeech, which I have used in the past, does not function well on Windows machines. I was able to succesfully generate and authenaticate an API key and prepare the appropriate programs and depenecies, but came up against a wall when I lacked the Python knowledge to create a sufficient API request script. I then turned to Otter AI, which quickly generated a transcript.

The video and radio captions were then compiled into one .txt document together with the text gathered from written articles.

There were several dead links in the search results. Every deadlink was substituted with the next functioning result.

### BBC

No scraper is available for BBC.co.uk.

"Transgender" was searched from the BBC.co.uk and the first fifty results were utilized. Default search settings were used. Results were predominately written articles, with a small number of radio programmes. A majority of the radio programmes contained a title, one line description, and air date, with no way to listen to them. These results were not usable, so the next functioning result was chosen when required.

The recording and transcription process described under "CBC" was also utilized for available audio files on BBC.co.uk.

## Step 2: Generating Texts with GPT-2

GPT-2 requires a decently powerful GPU to effeciently train. Since I was concerned about using my own computer and did not have access to a virtual machine, I utilized the free Google Collaboratory service. Unfortunately this means I was subject to algorithmically determined usage limitations that are not communicated to the user. I hit this limit several times, which prolonged this stage of the project.

I do not have the neccesary Python knowledge to create my own complicated scripts, but I am able to understand and alter variables. For this project I used [code written by Max Woolf](https://minimaxir.com/2019/09/howto-gpt2/) and changed/added/removed variables when neccesary.

The following script was used to train each instance, with ```run_name=``` and ```dataset=``` denoting the name of the instance and the name of the dataset respectively:

```
sess = gpt2.start_tf_sess()

gpt2.finetune(sess,
              dataset=file_name,
              model_name='124M',
              steps=1000,
              restore_from='fresh',
              run_name='run1',
              print_every=10,
              sample_every=200,
              save_every=500
              ) 
```

Each trained instance was then used to generate fifteen texts. The scripts used, along with platform-specific changes to process, are detailed below:

### Reddit
``` 
gpt2.generate(sess, run_name='bbcrun1',
              length=350,
              temperature=1.0,
              nsamples=10,
              batch_size=1
              )
```
Minimal changes were required to generate legible texts. I experimented with a higher temperature to limit repitition.

The following changes were made to the generated texts to ensure a degree of uniformity between the different platforms:
  + Line spacing, word tense, and punctuation were altered
  + Hyperlinks were removed
  + Instances of deadnaming were removed

### JSTOR
```
gpt2.generate(sess, run_name='jstorrun1',
              length=350,
              temperature=1.1,
              nsamples=10,
              batch_size=1,
              top_k=20,
              top_p=0.8
              )

```
Generating texts from JSTOR was the most difficult. I predict my issues came from my JSTOR corpus being the largest file created from the smallest number of articles. My first several attempts created texts that drew appriximately 90% of tokens from a single article. I increased the temperature to remedy this, which resulted in texts that were mostly illegible. I kept the high temperature and added limiters to my sampling (top_k for top sampling and top_p for cumulative probability). This worked for some articles, but many were still either nonsensical or repititive. As a result, I had to generate 40 texts total, as compared to 15 for other platforms.

The following changes were made to the generated texts to ensure a degree of uniformity between the different platforms:
  + Line spacing and punctuation were altered
  + Citations, page numbers, and hyperlinks were removed.

### YouTube
```
gpt2.generate(sess, run_name='youtuberun1',
              length=700,
              temperature=0.9,
              nsamples=10,
              batch_size=1
              top_p=0.9
              )

```
Generating texts from Youtube was relatively simple, but I had to make large changes to the generated texts. I relied on YouTube's automatic Closed Captioning data for the training input, which led to strange line spacing, time stamps, and the duplication of every line. I used grepWIN to remove the timestamps before training, but didn't address the other issues. I decided to generate texts that were twice as long and manually cut duplicated lines. This situation could have been avoided with more consistent corpora formatting.

The following changes were made to the generated texts to ensure a degree of uniformity between the different platforms:
  + Line spacing was heavily reformated
  + Punctuation and word tense were altered
  + Duplicated words/tokens were removed
  + Instances of deadnaming were removed 

### CBC
```
gpt2.generate(sess, run_name='cbcrun1',
              length=350,
              temperature=1.0,
              nsamples=10,
              batch_size=1
              )

```
Minimal changes were required to generate texts. I made an effort to use the same settings for CBC and BBC as they are the most comparable platforms.

The following changes were made to the generated texts to ensure a degree of uniformity between the different platforms:
  + Line spacing, word tense, and punctuation were altered
  + Duplicated words/tokens were removed


### BBC 
```
gpt2.generate(sess, run_name='bbcrun1',
              length=350,
              temperature=1.0,
              nsamples=10,
              batch_size=1,
              )
```
Minimal changes were required to generate texts. I made an effort to use the same settings for CBC and BBC as they are the most comparable platforms.

The following changes were made to the generated texts to ensure a degree of uniformity between the different platforms:
  + Line spacing, word tense, and punctuation were altered
  + Duplicated words/tokens were removed

## Step 3: Designing the Website

I first planned to create a website using Wordpress, but I felt my final product could be strengthened through more interactivity. This led me to Twine, a tool traditionally used to create narrative hypertext games. 

First, I designed the look of the website using the Javescript and CSS style sheets. Changes from the default include:
+ Colours and fonts
+ Removal of "back" button
+ Implementation of collapsable menu
+ Creation of tags (to apply sets of changes to whole pages or sections of text)

I then wrote content for the introductory pages, placed each text on a page, and then ensured all pages were appropriately linked.

I originally hosted the website on a custom domain through Github pages, but switched to traditional static hosting after several testers encountered HTTP errors.