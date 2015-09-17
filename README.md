# RHAI Complete Package #

### About this file ###

This README is written using the Markdown standard. It can be 
displayed as a plain text file, but you can open it in a browser
to get an html-styled text. Everything about Markdown standard
is available at http://daringfireball.net/projects/markdown/.

### About RHAI ###

RHAI is a Java tool developed to recognize household appliances'
patterns while they're running, by examinating power consumption.
Any question about this project or its usage will be welcome: 
just mail to Simone Colucci (simone.colucci01@universitadipavia.it).

### Archive organization ###

The complete package structure is represented below:

* README.md

* redd_dataset

* rhai_dataset

    * training
    * validation
    * fake
    * training_creation.xml
    * validation_creation.xml
    * fake_creation.xml
    
* rhai_data
    * settings
        * identification.properties
        * abstraction.properties

##### redd_dataset #####

RHAI works on appliances' data files, either for training and
execution. Though you're free to use any particular dataset, 
we used the Reference Energy Disaggregation Dataset (REDD) to 
test the identifier. To use REDD as a source of data, download
the latest low_freq dataset at http://redd.csail.mit.edu/, and 
save it in this folder. To do this, you may require a login
access. Follow the instructions available within the website.

##### rhai_dataset #####

RHAI works on appliances data files. Theese files must be inside
one of theese three folders, depending of their purpose:

* training: to teach the algorithm what shape appliances have
	                        
* validation: to analyze overall identification performance
	                        
* fake: to teach the algorithm not to recognize anything while 
reading meaningless data files
				
In addition to theese directories, rhai_dataset contains other four 
files, whose meaning are shown in the proper section:

* training_creator.xml
	
* validation_creator.xml
	
* fake_creator.xml
	
* redd.xsd

This package has no example dataset inside. Follow the intructions below
(Dataset creation using REDD and Dataset creation from other sources)
to define and build your data.

##### rhai_data #####

This folder contains every data file that let RHAI run. You generally 
need not change the content of this folder, with two exceptions, 
described in the dedicated section.

### Dataset creation using REDD ###

A RHAI data file must start when the appliance starts and end when 
it ends. So, if you want to use the REDD as a source of data, as we 
did, you need to convert the informations from REDD standard to RHAI. 
To accomplish this, you use the bin/dataset_creator.jar routine, like:

	java -jar bin/dataset_creator.jar $xml-file $target-directory
	
where

* $xml-file is one of the .xml files of rhai_dataset, and is used to 
tell the process how to create the dataset

* $target-directory is one of the folders of rhai_dataset
	
Through the .xml files you can define the building process of your dataset.
You need to determine which REDD files are to be used, and how to extract 
single appliances instance from that file (starting and ending time). The
schema of such .xml is saved in ./rhai_dataset/redd.xsd and it's:

	<redd:appliance name="appliance_name">
		<redd: appliance-file path="redd_file_path">
			<redd:appliance-instance name="optional">
				<redd:start> $start </redd:start>
				<redd:end> $end </redd:end>
			</redd:appliance-instance>
		</redd: appliance-file
	</redd:appliance>	
		
You may have many instances for each file, many files for each appliance,
and many appliances as you want in your .xml file. For each instance, you 
must insert two variables, taken from the REDD file:

* $start: the timestamp when the appliance starts
	                        	
* $end: the timestamp when the appliance ends
	            
The timestamp is represented under POSIX convention and counts the
seconds since Epoch (Jenuary 1st, 1970), and can be read from REDD.
Each REDD file indeed consists in lines like

	$timestamp      $power_consumption
        
Once you finished the .xml file, you can build your RHAI dataset.

### Dataset creation from other sources ###

You simply have to put folders in the dataset directory (let's say, 
for example, ./rhai_dataset/validation), and name each folder as the
appliance it represents. In the folder, put one or many .dat files. 
Each file must represent the power consumption of the appliance since
it turns on till the very ending, and must be composed of such lines:

	$day/$month/$year $hour:$min:$sec	$power_consumption

### How to use RHAI ###

To execute RHAI engine, run 
	
	java -jar bin/rhai.jar
				
and add required arguments. General usage has this shape:

	<option-id> <option-args> <command> <routine-id> <routine-args>
	
To know the set of available options and commands, run 

	java -jar bin/rhai.jar --help

To know the set of available routines for a certain command, run 

	java -jar bin/rhai.jar $command -help
			
For example, with command --trainer, this is the result:

	[ command --trainer with option -help started, mar 15 09 2015 18:35 ]
	Welcome to RHAI training system
	Theese are the available routines:
	----------
	option: -l , -ld , -load
	description: loads fingerprints from a directory
	required args: a dataset directory
	----------
	option: -o , -optimize
	description: computes the best acceptance likelihood
	required args: a real dataset , a fake dataset
	----------
	option: -h , -man , -help
	description: prints a simple help
	required args: 
	----------
	[ process terminated in about 0,00 seconds ]

	
##### Changing abstraction time (_not recommended_) #####

RHAI works by abstracting the measures read from input files in a high-level
label, in order to smooth data and improve performances. The abstraction time
parameter is the number of seconds that are to be abstracted into one single
label. The default value is set to 20 seconds, but you can change it. Please
note that values too much smaller than default will cause problems with the
sampling time of the input files (tests were run till t_abs ~ 10 * t_sampl) and
will result in a higher computation time, but values too much bigger than default
will decrease identification performance and will cause problems with small-duration
appliances (like, for example, microwave). To change this parameter you simply have 
to edit the relative value in the properties file rhai_data/settings/abstraction.properties.

##### Changing acceptance threeshold #####

RHAI will always identify the appliance of the library set that better fits the
given data file. That means it identifies even meaningless noise as an appliance.
To change this behavior, you can edit the acceptance threeshold parameter in the
rhai_data/settings/identification.properties file. This value can go from 0 (inclusive)
to 1 (exclusive). When 0, the engine will identify anything as an appliance. When 1, 
no appliance will be identified (null will then be returned). Since the perfect parameter
changes as the library dataset changes, there is no general rule to choose that value.
To determine the best one, you use the optimization routine available within RHAI.
First, you need to load your training dataset to teach RHAI what shape your applainces have;
to do this, just run:

	java -jar bin/rhai.jar --trainer -load $training_data
	
and wait for the dataset to be completely loaded. Then run: 

	java -jar bin/rhai.jar --trainer -optimize $real_data $fake_data
	
where 

* $training_data is a dataset of real appliances used for learning. You should use rhai_dataset/training

* $real_data is a dataset of real appliances. You should use rhai_dataset/validation. Do not use rhai_dataset/training

* $fake_data is a dataset of fake appliances. You should use rhai_dataset/fake

Since for decent-size datasets this computation may require a lot of time, we suggest
the use of --save option, to print the output in a persistent file. Run this:

	java -jar bin/rhai.jar --save $output_file --trainer -optimize $real_data $fake_data
	
As the computation goes on, you should see something like this in your terminal:

	[ command --trainer with option -optimize started, mar 15 09 2015 18:38 ]
      		25%        50%        75%       100%
	===============================
	
Once finished, you get the value that better recognize real appliances _and_ reject fake ones.
Now simply edit the .properties file by updating the value with the new one.
	
### Credits ###

RHAI is a project developed by Simone Colucci within the Laboratory
of Robotics at the University of Pavia. Latest version of the source 
can be found at the GitHub repository, at https://github.com/simone3991/rhai.
