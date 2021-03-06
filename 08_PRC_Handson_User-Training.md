# Introduction to Python iRODS Client (PRC) and VSC-PRC tools

*Prerequisites:*  
*-A vsc-account (or your own Linux client iRODS environment)*  
*-A KU Leuven account to access to the KU Leuven iRODS active data repository*  
*-Basic knowledge of command line (Bash)*   
*-Basic knowledge of Python is useful*    

This training introduces you to the basics of using the iRODS client API implemented in Python, as well as the additional functions and tools developed by Flemish Supercompuitng Center (VSC) to extend the iRODS Python API functionalities. The main feature of the VSC extensions is the possibility of using wildcards ("\*") and tildes ("~") for specifying iRODS data objects and collections. 

## Goal of this training
You will learn how to use the Python iRODS API (PRC) to interact with the KU Leuven iRODS infrastructure.
The following functionalities will be covered:

- Uploading and downloading data
- Uploading and downloading data collections
- Adding and editing metadata
- Setting access permissions for data objects and collections
- Querying for data using user defined metadata
- Using the VSC-PRC command line tools


### Login into VSC Tier-2 system
In this course we will use the Tier-2 Genius login node as our user iRODS client. But you can also use any other Linux system that has a iRODS client installed. 
You can find the available iRODS linux clients in the iRODS website: https://irods.org/download/ 

You have been provided during the training with a temporary vsc-account that you can use to connect to the system.

```sh
ssh login1-tier2.hpc.kuleuven.be
```
Use the temporary vsc-account and the password that you received at the start of the training. 


### Configuration of the iRODS connection

Connect to the KU Leuven iRODS portal (https://irods-demo.t.icts.kuleuven.be) and follow the instructions of the section iRODS Linux Client

You will be then start an iRODS session that will last 7 days. 

After 7 days the created temporary password will expire and you will need to repeat this procedure to reconnect to iRODS.

### Environment setup

- Connect to Genius with your temporary vsc-account using your favourite client. 
- Go to your data folder:
```sh
cd $VSC_DATA
```
  It is good practice to store files not in your home folder, but in your data folder, since a filled-up home folder can cause login issues.
  The home directory is mainly meant for configuration files.
  
- Copy the course material in your data directory

```sh
git clone https://github.com/hpcleuven/iRODS-local-admin-training.git
``` 
- The  Python iRODS API (PRC) and the VSC extensions are available as a module in the Tier-2 Genius Compute system.
Before using it you will need to load the corresponding module:

```sh
module load Python/3.7.2-foss-2018a
module load vsc-python-irodsclient/0.1-python-irodsclient-0.8.4
``` 

Note that vsc-python-irodsclient/0.1-python-irodsclient-0.8.4 will load a python module if there is none already loaded.
In order to have full control about which Python version is used it is recommended to prior to load the vsc-prc-python module 
to load the Python version you want to work with.


These command will activate a temporary token for a period of 7 days. After the 7 days have passed you will need to reactivate 
your access by re-executing the command again.

To do the exercises, make pythonscripts by using your favourite editor (vi, nano,...) and execute them with python <filename>. Alternatively, you can open the python interpreter and work interactively. (The python interpreter is not installed by default, so you will need to install it usinmg pip): e.g: pip install --user ipython) 

###  Working with collections and data objects

The first thing we need to do is to import the VSCIRODSSession module and create an iRODS session.
This class is derived from PRC's irods.iRODSSession class, and as such you can still use it to do what PRC is capable of (see https://github.com/irods/python-irodsclient). Here, we will focus on the functionalities that are added by VSC-PRC.  

```py
from vsc_irods.session import VSCiRODSSession
session = VSCiRODSSession(txt='-')
```
In addition to the keyword arguments for irods.iRODSSession, it also accepts a txt argument. This specifies where the session's print output should be directed to, with the default '-' referring to stdout.

Note that when using the VSCiRODSSession class is best practice to initiate it using the with construct to 
ensure that the session is cleanly terminated, even if an error occurs:

```py
from vsc_irods.session import VSCiRODSSession
with VSCiRODSSession(txt="-") as session:
    
    [your code here]
```

The VSC-PRC library provides a number of functions grouped by functionality:

- path functions: functions related to navigation and path information 
  - get_irods_home
  - get_irods_cwd
  - get_absolute_irods_path
  - ichdir
  - imkdir 

- bulk functions: functions to perform actions on multiple collections or dataobjects in once by using wildcards ("\*") and tildes ("~") for specifying iRODS data objects and collections. 
  - get
  - put
  - remove
  - metadata
  - size 
  - add_job_metadata

- search functions: advance search capabilities either by filename or by metadata entries in the format Attribute, Value:
  - iglob
  - walk
  - find 

Let's try some of those functions:

Printing the iRODS home directory and the current working directory:

```py
from vsc_irods.session import VSCiRODSSession
with VSCiRODSSession(txt="-") as session:
home = session.path.get_irods_home()
cwd = session.path.get_irods_cwd()
print(home,cwd)
```

Create a collection `training` on your iRODS home directory:

```py
from vsc_irods.session import VSCiRODSSession
with VSCiRODSSession(txt="-") as session:
session.path.imkdir('training')
```

Try the following additions to the previous code: 

We can verify that the collection has been created by requesting the absolute path (the full path from the root):

```py
abs_path = session.path.get_absolute_irods_path('training')
print(abs_path)
```
Now try to change to the directory we just created:


```py
session.path.ichdir('training')
```

We can now check the current working directory, which is different from the home directory:

```py
cwd = session.path.get_irods_cwd()
home = session.path.get_irods_home()
print(cwd,home)
```

Let's upload some data into our training iRODS collection!

First let's use the iRODS Python Client to upload a single dataobject from the molecules folder, alcl3.xyz, to iRODS.
You will need to fill in the correct path to the file you want to upload:

```py
session.data_objects.put( [location of the file]  , [target destination of the file])
```
For example, if user vsctmp10 wants to upload this to his training folder, it might look like this:

```py
session.data_objects.put('/data/leuven/tmp/vsctmp10/iRODS-local-admin-training/molecules/alcl3.xyz','/icts_icts/home/u00XXXXX/training/')
```
where u00XXXXX should be replace by your KU Leuven u-account number. 

This demonstrates that even using the VSCIRODSSession class all the functionalities of the Python iRODS Client (PRC) are still available. 

Let's use the search.find() function to verify that the file has been uploaded to iRODS:

```py
irods_path = '/icts_icts/home/u00XXXXX/training/'
session.search.find(irods_path,types='f')
```
This command returns a list of iRODS data objects paths (types='f' means list files) which match a given pattern. As we have not defined a pattern then the default value ("\*") is used. So, the result will be a list containing the iRODS paths of all the files in the collection `irods_path`.  

We can loop over the list to print the search results:

```py
for item in session.search.find(irods_path, types='f'):
	print(item)
```

While this PRC function works well, it has the disadvantage that only works with single data objects or collections.
Here the VSC-PRC bulk functionalities provide more flexibility, as they allow to select files using wildcards. 

So, let's now upload to the training iRODS collection all files with extension .xyz:

```py
session.bulk.put('./molecules/*.xyz', irods_path)
```
We can execute again the same find command we used before to verify that indeed all files with extension .xyz have been uploaded. 

```py
for item in session.search.find(irods_path, types='f'):
	print(item)
```
Let's now create a subdirectory `results` on our molecules collection and see how we can both list all 
subcollections and files of a given collection with the find function. As 'f' stands for files, 'd' stands for directories or collections.

```py
session.path.imkdir('results')
for item in session.search.find(irods_path, types='d,f'):
        print(item)
```


###  Adding metadata to files 

Until now we have seen how to use the search.find function to search for collections and data objects 
based on their paths and filenames. But iRODS also offers the possibility to add metadata to them 
in the form of tuples or triples (Attribute-Value-[Unit]), also called AVUs. 

Let's start adding metadata to our `training` collection to identify it with the associated research project.
We will add a tuple Attribute-Value (Project, Training) and we will apply this metadata to the training collection
and all its subcollections and data objects by using the recursive option. As we have also selected the verbose 
option, we will see to which collections and dataobjects the metadata is added: 

```py
avu1= ('Project', 'Training')
session.bulk.metadata(irods_path, collection_avu=avu1, action='add', recurse=True, verbose=True)
session.bulk.metadata(irods_path + '/*' , object_avu=avu1, action='add', recurse=True, verbose=True)
```

We will now create two other AVUs pairs to add to the xyz files the experiment information.
The molecules (c6h6.xyz, ch2och2.xyz and ch3cooh.xyz) where used in experiment1 while the file sih4.xyz in experiment2.
The rest of the molecules has not yet being used in any experiment. 

```py
avu2 = ('Experiment', 'Experiment1')
avu3 = ('Experiment', 'Experiment2')
session.bulk.metadata(irods_path + '/c*.xyz' , object_avu=avu2, action='add', recurse=True, verbose=True)
session.bulk.metadata(irods_path + '/sih4.xyz' , object_avu=avu3, action='add', recurse=True, verbose=True)
```

Now we can use this metadata information to perform searches on the iRODS collections and to download all the selected files to our local directory.
Let's download all the files that were used in Experiment1 using the bulk.get function: 


```py
iterator = session.search.find(irods_path, object_avu=avu2)
session.bulk.get(iterator, local_path='.', verbose=True)
```

To finish let's clean up by removing all the `training` collection. 
We will use the option recurse=True to remove the collection and all its subcollections 
and dataobjects and the option force=True to completely remove the files (iRODS by default move it to a Trash area). 

```py
session.bulk.remove(irods_path, recurse=True, force=True, verbose=True)
```


##  Exercises 

### Exercise 1 

VSC-PRC also comes with a set of scripts which make it easy to use the
Python module from a Unix shell:

- vsc-prc-find
- vsc-prc-iget
- vsc-prc-iput
- vsc-prc-imkdir
- vsc-prc-irm
- vsc-prc-imeta
- vsc-prc-add-job-metadata

Typing e.g. :code:`vsc-prc-find --help` will show a description of the
recognized arguments. The command-line equivalents of the three Python
snippets above, for example, would look like this:

```sh 
vsc-prc-iget '~/my_irods_collection/*.txt' -d .
vsc-prc-find '~' -n '*.txt' --object_avu='Author;Me'
vsc-prc-find '~' -n '*.txt' --object_avu='Author;Me' | xargs -i vsc-prc-iget {} -d .
```

Try to reproduce what we have done in the previous section on ipython, but this time using the vsc-prc
command line tools

### Exercice 2 

Let's put everything together. Write a python program that does the following:

- Create a directory in iRODS named `training`
- Upload the directory `molecules` as subcollection of the training directory
- Add the following metadata to the `training` collections and its subcollections:
  - Project: Training
  - PI: Homer Simpson

- Add the following metadata to the the c\*.xyz files:
  - SoftwareName: OpenBabel
  - SoftwareVersion: 3.1.1
  - SoftwareLicense: GNU General Public License, version 2

- Modify the permissions of the training collection to be readable by all the lt1_es2020 group. 
Hint: You will need to use the following  PRC functions:

  - Get the collection information: 

  ```py
  coll = session.collections.get('/data/leuven/tmp/vsctmp10/iRODS-local-admin-training/training')
  ```

  This command will store in the coll variable all the information related to the collection. 
  The information that can be obtained:

  - coll.id 
  - coll.path
  - coll.data_objects
  - coll.subcollections


  - Set up the new permissions by doing:

  ```py
  from irods.access import iRODSAccess
  acl = iRODSAccess('read', coll.path, 'lt1_es2020', session.zone)
  session.permissions.set(acl)
  ```

  where:

  - 'read' is the kind of permission we want to give (other options are 'own', 'write' and 'null')
  - coll.path is the path of the collection we want to modify obtained the previous session.collection.get()
  - 'lt1_es2020' is the name of the group to whom we want to give read access
  - session.zone is the iRODS zone to which this collection belongs and will be taken from the session information (in our case: icts_icts)

- Finally create a new local directory named `molecules_copy` and download all the files that were created with the OPenBabel software.
