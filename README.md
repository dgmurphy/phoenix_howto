# Open Event Data Alliance Phoenix Pipeline Install & Test

A step-by-step guide to running Phoenix for the purpose of updating the dictionaries for Actors, Agents, Issues and Discards. 

These instructions will set up the following tools:

* MongoDB: a popular database tool
* OEDA scraper: crawls RSS newsfeeds and stores the articles in a MongoDB database
* OEDA stanford_pipeline: reads sentences from the articles in MongoDB and builds sentence parse trees using the Stanford Core NLP library.
* OEDA phoenix_pipeline: reads the sentences and parse trees and uses the Petrarch 2 library to create coded events using the CAMEO codebook and the Actor,Agent, and Issues dictionaries.

Note: The Git repos listed here were originally forked from the OEDA Github account here: https://github.com/openeventdata. My versions have minor modifications to facilitate the dictionary update tasks (e.g. geocoding is removed, scraper's old ntlk library is included locally, Petrarch code is included in the Phoenix Pipeline, and a few other things.)

## PreRequisites

These instructions were tested on Ubuntu 18.04.  You should have basic knowledge of Linux, Git, and Python.

Make sure you have at least 4 GB of disk space available for this project.

Note: These instructions have known problems on Ubuntu 20 due to limited Python2 lib support.


### Install Git

```sudo apt-get install git```

Check:

```git --version```

### Install pip

```sudo apt-get install python-pip```

### Install virtualenv

```sudo apt install virtualenv```

Check:

```virtualenv --version```

### Install python2
Comes with Ubuntu.

Check:

```python2 --version```

### Install Java 8
Tested with OpenJDK 8.

`apt-get install openjdk-8-jdk`

Check:

`java -version`

### Make a project dir
Create a directory for this project:

```mkdir oeda```



## Install MongoDB

Familiarize yourself with the purpose and operation of the MongoDB database:

https://www.mongodb.com/

Follow these instructions for installing MongoDB on Ubuntu:

https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

Check: run the Mongo command-line interface to make sure it's installed

```mongo```  to enter the MongoDB CLI.

```quit()``` to exit MongoDB.


### MongoDB User Interface: Robo 3T
Install the "Robo 3T" user-interface for MongoDB:

Visit: ```https://www.robomongo.org/download``` and download the 
Robo 3T tool only. 
*Skip the Studio 3T download.*

Copy the tar.gz file to your home directory and unzip it.

After unzipping, find the file `robo3t` in the bin directory and launch it. E.g.

```~/robo3t-1.3.1-linux-x86_64-7419c406/bin/robo3t```

In the MongoDB Connections panel use the Create link to make a new database connection. 

In the Connection tab Name field enter the name "Event Pipeline"

Keep all the other connection defaults and click Save.

Check:  Select the Event Pipeline connection to make sure you can connect to your new empty MongoDB.

## Install & Test Scraper

Scraper crawls RSS feeds to gather news content and store it in a MongoDB instace. View the scraper README on github: https://github.com/dgmurphy/scraper

### Clone the Scraper Repository

Change to the oeda directory:

```cd oeda```

In the oeda directory clone the forked version of the scraper repo:

```git clone https://github.com/dgmurphy/scraper.git```

```cd scraper```

### Create Python Environment & Install libraries

Create a Python2 virtual environment:

```virtualenv -p /usr/bin/python2.7 venv```

Activate the virtual environment:

```source venv/bin/activate```

Check: make sure your linux command prompt starts with '(venv)'

Install the setuptools library (this step required for nltk install)

```pip install setuptools==9.1```

Install the libxml dependecies:

```sudo apt-get install libxml2-dev libxslt-dev python-dev```

Unzip the nltk library file:

```tar -xzvf nltk-2.0.4.tgz```

Install the nltk 2.0.4 library:

```pip install ./nltk-2.0.4```

*Q: Why not use the requirments.txt file to install the nltk library?*

*A: This is an old library version that requires a manual installation.*

Install the other required libraries for the scraper project:

```pip install -r requirements.txt```

### Do a Test Run of Scraper

Verify that the file `default_config.ini` has this line which will specify a short list of URLs to scrape for test purposes:

```file = whitelist_urls_short.csv```

Run the scraper:

`python scraper.py`

When it completes, inspect the log to verify that it added entries:

```vim scraper_log.log```

You should see a number of "added entry" lines with news article urls.

### View the Scraped Stories in MongoDB

Open the Robo 3T application. E.g

`~/robo3t-1.3.1-linux-x86_64-7419c406/bin/robo3t`

From the Connection dialog select the Event Pipeline connection.

Check: Robo 3T should connect and you should see a database called "event_scrape"

Expand the Collections folder under the Event Scrape item. You should see a collection called "stories".

Double-click the stories collection to view the entries in the database. 

Check: Expand some of the stories to browse them. Right click a story item and select View Document so see the complete item content. You should see the news article content here. Notice the field called "stanford" has value zero becuase no parse trees have been created for this story.

### Deactivate

Before proceeding, deactivate your scraper virtual environment:

```deactivate```

Check: makes sure your command prompt no longer start with (venv)


## Install and Test the Stanford Pipeline

This tool parses sentences into parse trees (sentence diagrams). This step is crucial for allowing the later parts of the event coding pipeline to work.
Read about the stanford_pipeline tool here: 
https://github.com/dgmurphy/stanford_pipeline

### Clone the Repo

In the oeda directory:

```git clone https://github.com/dgmurphy/stanford_pipeline.git```

```cd stanford_pipeline```

### Download the Core NLP Models

```
wget http://nlp.stanford.edu/software/stanford-corenlp-full-2014-06-16.zip
unzip stanford-corenlp-full-2014-06-16.zip
mv stanford-corenlp-full-2014-06-16 stanford-corenlp
cd stanford-corenlp
wget http://nlp.stanford.edu/software/stanford-srparser-2014-07-01-models.jar
```

In the file default_config.ini edit the line:

`stanford_dir = ~/stanford-corenlp`

to use the full path to the stanford-corenlp dir, e.g.:

`stanford_dir = /home/dmurphy/dev/oeda/stanford_pipeline/stanford-corenlp`

### Create Python Environment & Install libraries

In the stanford_pipeline directory create a Python2 virtual environment:

```virtualenv -p /usr/bin/python2.7 venv```

Activate the virtual environment:

```source venv/bin/activate```

Check: make sure your linux command prompt starts with '(venv)'

### Install Python Libraries

```pip install -r requirements.txt```


### Execute a Test Run

```python process.py```

Up to a minute of [Errno 111] Connection refused messages are normal during startup.

Check: The command line should show "processing story" messages.

### View Stories in MongoDB

Open the Event Pipeline connection in Robo 3T and view a document in the story collection. Verify that the field 'stanford' = 1 and that sentence parse trees are visible.

Example of a parse tree:

```
(ROOT (S (NP (NP (NNP Yangon) (NNP Region) (NNP Hluttaw))
(PRN (-LRB- -LRB-) (NP (NNP Parliament)) (-RRB- -RRB-))) 
(NP (NNP Speaker) (NN Tin) (NNP Maung) (NNP Tun)) 
(VP (VBD announced) (SBAR (IN that) (S (NP (NP (DT an) 
(NN emergency) (NN meeting)) (PP (IN of) (NP (DT the) 
(JJ regional) (NN parliament)))) (VP (MD will) (VP (VB be) 
(VP (VBN held) (PP (IN on) (NP (NNP June) (CD 18))))))))) (. .)))"
```

### Deactivate

Before proceeding, deactivate your scraper virtual environment:

```deactivate```


## Install and Test the Phoenix Pipeline

The Phoenix Pipeline takes the sentence content and the parse trees and produces events using the Petrarch event coder. Petrach includes a number of data dictionaries including the CAMEO event codes, Actor, Agent and Issues dictionaries, and a Discards list. 

Read more about Petrarch here:

https://petrarch.readthedocs.io/

Read more about the Phoenix Pipeline here:

https://github.com/dgmurphy/phoenix_pipeline 

and here:

https://phoenix-pipeline.readthedocs.io/en/latest/

Note: this version of the pipeline has Geocoding disabled. Events will not be location coded but the source sentence will be saved with each event so a seperate process can be used to perform sentence-level geocoding.


### Clone the Repo

In the oeda directory:

```git clone https://github.com/dgmurphy/phoenix_pipeline.git```

### Create Python Environment & Install libraries

In the phoenix_pipeline directory perform the usual steps to create & activate the virtual environment, then pip install -r requirements.txt.


### Edit the Config File

The pipeline will process events on a per-day basis by checking the date fields of the stories in MongoDB.
To process events for a particular day we need to specify that day in the config file.


In the file `PHOX_config.ini` :

`run_date = 20200713`

NOTE: To process events on the same day they were collected, set the run_date to today's date plus one day.

### Do a Test Run of the Pipeline

```python pipeline.py```


The pipeline should produce a csv file e.g.:

`events.full.20200713.csv`

Open this file in LibreOffice to view it.  For the seperator options use:

* Seperated by: Tab only
* String delimeter: '   (single quote)



## Use the Sentence Tester

The sentence tester will let you run the petrarch coder on a list of sentences in one of two modes:

1) Standard Mode: Two coded actors and a coded verb must be found in the dictionaries, othwerwise no events will be generated.

2) Null Actors Mode:  If two uncoded actors and a coded verb are found, then events are generated using the actor names from the sentence. You can use these names to add the actors to the dictionaries.

### Install

``` 
git clone https://github.com/dgmurphy/petrarch_sentence_tester.git
cd petrarch_sentence_tester
virtualenv -p /usr/bin/python2.7 venv
source venv/bin/activate
pip install -r requirements.txt
```
```
cd stanford_corenlp_pywrapper
wget http://nlp.stanford.edu/software/stanford-corenlp-full-2014-06-16.zip
unzip stanford-corenlp-full-2014-06-16.zip
mv stanford-corenlp-full-2014-06-16 stanford-corenlp
cd stanford-corenlp
wget http://nlp.stanford.edu/software/stanford-srparser-2014-07-01-models.jar
```

### Use

Add sentences to the file `input.txt`

Each sentence must have a date so that the actors can be coded using their valid time ranges.

EXAMPLE INPUT:

`20200701 Dr. Fauci warned governors that the pandemic could surge if masks are not worn.`

Run `python test_parse.py s`  (standard mode)

Result: No events generated because Dr. Fauci is not in the actor dictionaries.

Run `python test_parse.py n`  (null actors mode)

Result: One event is generated and we see that the word 'governors' has a code ("GOV") but Dr. Fauci does not. 
```
Actors for event 1
	Soure actor: Dr. Fauci => *1*
	Target actor: governors <GOVERNORS> =>  *2*GOV
	Verb code:  130s
```

If we add Dr. Fauci to the actor dictionary then this sentence will generate an event in standard mode.

TODO: This sentence will also code an issue for PANDEMIC. Modify the output to show the coded issues.

## Learn How to Edit the Petrarch Dictionaries

If the Petrarch coder does not find at least two actors in a sentence then it will not generate any events.  Many named entities that should count as actors or agents do not get coded because their names are not in the dictionary. To update these dictionaries we need to identify missing actors/agents and add them to the dictionary. The process will generally be as follows:

* Collect 'dead' sentences from the pipeline (sentences that did not generate any events).
* Run the dead sentences through the sentence tester in "Null Actor Mode" which will identify the labels for the named entities that are missing from the dictionary (e.g. "Dr. Fauci").
* Add the labels and actor code for Dr. Fauci (e.g. "Anthony Fauci", "Dr. Anthony Fauci" etc.) and any other missing named entities to the Actor/Agent dictionaries using the CAMEO coding scheme.
* Re-run the dead sentence through the tester to make sure the new Actor/Agent code is detected.

See the documentation here for instructions on how to edit the dictionaries:

https://petrarch.readthedocs.io/en/latest/dictionaries.html

