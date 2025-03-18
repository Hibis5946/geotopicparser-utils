GeoTopicParser Utilities
========================
This repository contains Named Entity Recognition (NER) models for location data 
forked and augmented from the [Apache OpenNLP](http://opennlp.apache.org/) project,
trained against the [NSF Polar CyberInfrastructure](http://github.com/chrismattmann/trec-dd-polar)
data contributed to the NIST TREC Dynamic Domain Working Group. 

In addition, a custom MIME type definition for the [Tika GeoTopicParser](https://wiki.apache.org/tika/GeoTopicParser)
is provided so that *.geot files can be parsed using the GeoTopicParser. *.geot flies are just text/plain files with
text that includes location information.

Finally, a sample set of *.geot files are provided, currently there exists one from the Polar domain.

Questions, comments?
===================
Send them to [Chris A. Mattmann](mailto:chris.a.mattmann@jpl.nasa.gov).

Contributors
============
* Yun Li, USC
* Chris A. Mattmann, JPL

# Updated readme from [https://cwiki.apache.org/confluence/display/tika/GeoTopicParser] to install the following Packages:
lucene-geo-gazetteer, geotopicparser, tika-server-standard
- Update date: 3/18/2025
- By: Sena London, APDS @ USC

# Compatible Java
JDK 17

# lucene-geo-gazetteer
- git clone https://github.com/chrismattmann/lucene-geo-gazetteer.git

# Change the directory into the newly downloaded package folder
cd lucene-geo-gazetteer
# Build the lucene-geo-gazetteer from that directory
- mvn clean package
- mvn install assembly:assembly

# Export path (command for Linux distribution)
- export PATH=$PATH:/root/lucene-geo-gazetteer/src/main/bin

# Open the ~/.bashrc file (which is currently /root/.bashrc):
bash
- nano /root/.bashrc

# Append this line at the end (Depending on the OS, this is for Linux distribution):
bash
export PATH=$PATH:/root/lucene-geo-gazetteer/src/main/bin
# Save and exit the file (Ctrl + O, then Enter, and Ctrl + X to exit).

#Reload the shell configuration:
bash
- source /root/.bashrc

# Build gazetteer
# Change into the lucene directory
cd lucene-geo-gazetteer
#download the repository
curl -O http://download.geonames.org/export/dump/allCountries.zip
# unzip
- unzip allCountries.zip
- lucene-geo-gazetteer -i geoIndex -b allCountries.txt

# You can verify that the Gazetteer build worked by searching e.g., for Pasadena, and/or Texas:
lucene-geo-gazetteer -s Pasadena Texas -json
# start the lucene-geo-gazetteer-server
lucene-geo-gazetteer -server
# Open another terminal to verify if REST API is responding by searching e.g., for Pasadena, and/or Texas
curl "http://localhost:8765/api/search?s=Pasadena&s=Texas"
# output sample
- user:~/lucene-geo-gazetteer# lucene-geo-gazetteer -s Pasadena Texas -json
- {"Texas":[{"name":"Texas","countryCode":"US","admin1Code":"TX","admin2Code":"","latitude":31.25044,"longitude":-99.25061}],"Pasadena":[{"name":"Pasadena","countryCode":"US","admin1Code":"CA","admin2Code":"037","latitude":34.14778,"longitude":-118.14452}]}

# Search the lucene-geo-gazetteer version
# Navigate into the lucene-geo-gazetteer directory
cd lucene-geo-gazetteer
# Search the version in the pom file
grep -m 1 "<version>" pom.xml

# Then search the index (e.g., for Pasadena and Texas): java -cp target/lucene-geo-gazetteer-<version>-jar-with-dependencies.jar edu.usc.ir.geo.gazetteer.GeoNameResolver -i geoIndex -s Pasadena Texas
# The version at the time of this edit is 
 <version>0.3-SNAPSHOT</version>
# Replace the version in the command
java -cp target/lucene-geo-gazetteer-0.3-SNAPSHOT-jar-with-dependencies.jar edu.usc.ir.geo.gazetteer.GeoNameResolver -i geoIndex -s Pasadena Texas

# Start the lucene server
lucene-geo-gazetteer -server
# Query
curl "localhost:8765/api/search?s=Pasadena&s=Texas&c=2"
# Output sample
![image](https://github.com/user-attachments/assets/382f8aa1-346c-4726-bfe1-76e967494d2d)


# Installing and downloading an NER model
The next thing you'll need is a Named Entity Recognition model for places. The GeoTopicParser uses Apache OpenNLP and with its 1.5 version, OpenNLP provides already trained models for location names in text data. You can download the en-ner-location.bin file already pre-trained by the OpenNLP folks. One thing to note is that OpenNLP's default name finder is not accurate, so building your own NER location model is highly recommended. In this case, please follow these instructions.

The model needs to be placed on the classpath for your Tika installation in the following directory:

org/apache/tika/parser/geo/

The following instructions show how to download the model and place it on the right path:

- mkdir /root/location-ner-model && cd /root/location-ner-model
- curl -O https://opennlp.sourceforge.net/models-1.5/en-ner-location.bin
- mkdir -p org/apache/tika/parser/geo/
- mv en-ner-location.bin org/apache/tika/parser/geo/

# Test out the GeoTopicParser
Now you can run Tika and try out the GeoTopicParser. At the moment since it's a Parser and not a Content-Handler (hopefully will develop it later), the parser is mapped to the MIME type application/geotopic which is a sub-class of text/plain. So, there are two steps to try the parser out now.

# Create a .geot file, you can use this sample file from the NSF Polar data contributed to TREC. 2. Tell Tika about the application/geotopic MIME type. You can download this file and place it on the classpath in the org/apache/tika/mime directory, e.g., by doing:
- mkdir /root/geotopic-mime && cd /root/geotopic-mime
- mkdir -p org/apache/tika/mime
- curl -O https://raw.githubusercontent.com/chrismattmann/geotopicparser-utils/master/mime/org/apache/tika/mime/custom-mimetypes.xml
- mv custom-mimetypes.xml org/apache/tika/mime
# download the polar.geot from chrismattmann repository
curl -o /root/geotopic-mime/polar.geot https://raw.githubusercontent.com/chrismattmann/geotopicparser-utils/master/sample-files/polar.geot

# With those files in place, let's use the GeoTopicParser using Tika-App:
java -cp /root/tika/tika-app/target/tika-app-4.0.0-SNAPSHOT.jar:/root/tika/tika-parsers/tika-parsers-ml/tika-parser-nlp-package/target/tika-parser-nlp-package-4.0.0-SNAPSHOT.jar org.apache.tika.cli.TikaCLI -m polar.geot

# run python to search geoparser through tika
# build the tika 4.0.0 server
cd /root/tika/tiker-server
mvn clean install
# start the tika server and keep it running
- java -jar /root/tika/tika-server/tika-server-standard/target/tika-server-standard-4.0.0-SNAPSHOT.jar
or 
- nohup java -jar tika-server-standard/target/tika-server-standard-4.0.0-SNAPSHOT.jar > tika-server.log 2>&1 &

# run the python file that include the tika module to search for geo name at large scale
pyhon your_script_py

# Command to see and manage all java version
update-alternatives --list java
# search the system for all java executable
sudo find / -name "java" -type f 2>/dev/null
# Verifiy installed Packages (for package Manager)
dpkg --list | grep -i java
# configure java alternatives (choose from the installed java)
sudo update-alternatives --config java
# verify the version
java -version
# add jdk to the path
nano ~/.bashrc
# add the following lines
- export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
- export PATH=$JAVA_HOME/bin:$PATH
# save and close the file and apply the change
source ~/.bashrc

## command to install tika from the maven repository
mvn dependency:get -Dartifact=org.apache.tika:tika-app:2.6.0
# locate the jar file
~/.m2/repository/org/apache/tika/tika-app/2.6.0/tika-app-2.6.0.jar
# update permission if there is any
chmod +x ~/.m2/repository/org/apache/tika/tika-app/2.6.0/tika-app-2.6.0.jar
# run the tika app
java -jar ~/.m2/repository/org/apache/tika/tika-app/2.6.0/tika-app-2.6.0.jar

# step for tika-GeoParser
# change directory into tika (Linux based OS)
cd ~/tika
# Build the tika project by generating the target/ directory and tika-app-version-SNAPSHOT.jar
mvn clean install
# Verify the bluid
ls -lh tika-app/target/
# Run this to check the version of the tika-app-SNAPSHOT
java -jar tika-app/target/tika-app-4.0.0-SNAPSHOT.jar --version

# Run the geo command
java -classpath tika-app/target/tika-app-4.0.0-SNAPSHOT.jar:tika-parsers/tika-parsers-ml/tika-parser-nlp-package/target/tika-parser-nlp-package-4.0.0-SNAPSHOT.jar:/root/location-ner-model:/root/geotopic-mime org.apache.tika.cli.TikaCLI -m polar.geot

# Other helpful command
# delete the tika repository, directories removal (for Linux based OS)
rm -rf ~/tika
# remove the cached maven dependency
rm -rf ~/.m2/repository/org/apache/tika
# verify the removal
ls ~/.m2/repository/org/apache/tika
# command to find tika version (Linux based OS)
grep -m 1 "<version>" pom.xml

Credits
=======
This project began as the [CSCI 572](http://sunset.usc.edu/classes/cs572_2015/) 
project of [Yun Li](https://github.com/AranyaLi) on the NSF Polar CyberInfrastructure
project at USC under the supervision of Chris Mattmann. You can find Yun's original
code base [here](https://github.com/AranyaLi/GeoParsingNSF).

This work was sponsored by the [National Science Foundation](http://www.nsf.gov/) 
under funded projects 
[PLR-1348450](http://www.nsf.gov/awardsearch/showAward?AWD_ID=1348450&HistoricalAwards=false) 
and [PLR-144562](http://www.nsf.gov/awardsearch/showAward?AWD_ID=1445624&HistoricalAwards=false).


License
=======
[Apache License, version 2](http://www.apache.org/licenses/LICENSE-2.0)
