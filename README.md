# Open Event Data Alliance Phoenix Pipeline Install & Test

A step-by-step guide to running Phoenix for the purpose of updating the dictionaries for Actors, Agents, Verbs, Issues and Discards. 

These instructions will set up the following tools:

* MongoDB: a popular database tool
* OEDA scraper: crawls RSS newsfeeds and stores the articles in a MongoDB database
* OEDA stanford_pipeline: reads sentences from the articles in MongoDB and builds sentence parse trees using the Stanford Core NLP library.
* OEDA phoenix_pipeline: reads the sentences and parse trees and uses the Petrarch 2 library to create coded events using the CAMEO codebook and the Actor,Agent, and Issues dictionaries.
* Actor Generator: similar to the phoenix pipeline but produces actor labels.
* Sentence-Tester: runs the Petrarch coder on a list of sentences and produces events using either null actors mode or standard mode.

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

Scraper crawls RSS feeds to gather news content and store it in a MongoDB instance. 

### Clone the Scraper Repository

Install the Scraper tool using the instructions here:

https://github.com/dgmurphy/scraper

Be sure to activate your Python 2.7 virtual environment befor running scraper.

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

Follow the installation instructions in the git repo:

https://github.com/dgmurphy/stanford_pipeline


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

Install the Phoenix Pipeline into the oeda directory. Follow these instructions to install:

https://github.com/dgmurphy/phoenix_pipeline


## Install and Test the Actor Generator

Install the actor generator into the oeda directory.

Follow these instructions to install:

https://github.com/dgmurphy/phoenix_actor_gen


## Install and Test the Sentence Tester

The sentence tester will let you run the petrarch coder on a list of sentences in one of two modes:

1) Standard Mode: Two coded actors and a coded verb must be found in the dictionaries, othwerwise no events will be generated.

2) Null Actors Mode:  If two uncoded actors and a coded verb are found, then events are generated using the actor names from the sentence. You can use these names to add the actors to the dictionaries.

### Install

Install the sentence tester in the oeda directory. Follow these instructions to install and test:

https://github.com/dgmurphy/petrarch_sentence_tester

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

If the Petrarch coder does not find at least two actors in a sentence then it will not generate any events.  Many named entities that should count as actors or agents do not get coded because their names are not in the dictionary. To update these dictionaries we need to identify missing actors/agents and add them to the dictionary. 

See the documentation here for instructions on how to edit the dictionaries:

https://petrarch.readthedocs.io/en/latest/dictionaries.html


### Actor Dictionary Update Procedure

1. Run scraper to get stories into MongoDB.
1. Run stanford-pipeline to generate parse trees for the stories.
1. Run phoenix-pipeline to generate a 'baseline' events file using the existing actor dictionary.
1. Run actor-generator to generate an actor-label list for the stories.
1. Review the actor-label list for uncoded actors and use the CAMEO codebook to create new actor codes.
1. Update the actor dictionary in the *sentence-tester project* with the new actor codes.
1. Update the sentence-tester input file with sentences from the actor-label list output.
1. Run the sentence tester in standard mode to see if the updated actor dictionary succeeded. 
1. Once the sentences are generating events in standard mode, merge the dictionary updates into the phoenix-pipeline actor dictionary and also into the actor-generator actor dictionary.
1. Re-run the phoenix-pipeline and compare the output to the baseline events list. The new events list should include addtional events for the newly-coded actors.

Repeat this process daily, for the rest of your life, to add new actors.


Notes:
* Scraper: Check to makes sure you are using the desired url whitelist for the target RSS Feeds.
* Scraper: If you add a new RSS feed to the whitelist you need to all the source label in the source_keys.txt file in the phoenix-pipeline.
* Stanford-pipeline: Make sure the processor is configured to process all stories rather than a selected day.
* Phoenix-Pipeline: Rename or copy the events files you want to keep so they don't get overwritten.
* Actor-generator: Make sure to use the same run_date for both the phoenix-pipeline and the actor-generator.
* Sentence Tester: Running the sentence tester in standard mode should generate events for each sentence if the updates to the actor dictonary were done properly. If no events were generated, try running the sentence-tester in null actors mode to check the actor labels and fix the dictionary.


### Other dictionary updates

A similar procedure can be followed to update the dictionaries for Agents, Issues, Discards and Verbs.
