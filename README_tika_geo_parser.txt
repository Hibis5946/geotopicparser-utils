This is the updated readme from https://cwiki.apache.org/confluence/display/tika/GeoTopicParser to install the following Packages:
lucene-geo-gazetteer
geotopicparser
tika-server-standard

#By Sena Londoon
MS/APDS @ USC

# Recommanded JDK at the time of this edit: JDK 17 for windows or 11 for MAC users
# You have to make sure that your system is configured to use JDK 17 and the path correctly exported before this installation

##lucene-geo-gazetteer
git clone https://github.com/chrismattmann/lucene-geo-gazetteer.git

# Change the directory into the newly downloaded package folder
cd lucene-geo-gazetteer

#Build the lucene-geo-gazetteer from that directory
mvn clean package
mvn install assembly:assembly

#Export path (command for Linux distribution)
export PATH=$PATH:/root/lucene-geo-gazetteer/src/main/bin

#Open the ~/.bashrc file (which is currently /root/.bashrc):
nano /root/.bashrc

#Append this line at the end (Depending on the OS, this is for Linux distribution):
export PATH=$PATH:/root/lucene-geo-gazetteer/src/main/bin
##Save and exit the file (Ctrl + O, then Enter, and Ctrl + X to exit).

#Reload the shell configuration:
source /root/.bashrc

#Build gazetteer
#Change into the lucene directory
cd lucene-geo-gazetteer

#download the repository
curl -O http://download.geonames.org/export/dump/allCountries.zip

#unzip
unzip allCountries.zip
lucene-geo-gazetteer -i geoIndex -b allCountries.txt


##You can verify that the Gazetteer build worked by searching e.g., for Pasadena, and/or Texas:
lucene-geo-gazetteer -s Pasadena Texas -json

# Ouput sample
{"Texas":[{"name":"Texas","countryCode":"US","admin1Code":"TX","admin2Code":"","latitude":31.25044,"longitude":-99.25061}],"Pasadena":[{"name":"Pasadena","countryCode":"US","admin1Code":"CA","admin2Code":"037","latitude":34.14778,"longitude":-118.14452}]}

#start the lucene-geo-gazetteer-server
lucene-geo-gazetteer -server

#Open another terminal to verify if REST API is responding by searching e.g., for Pasadena, and/or Texas
curl "http://localhost:8765/api/search?s=Pasadena&s=Texas"

#output sample
{"Texas":[{"name":"Texas","countryCode":"US","admin1Code":"TX","admin2Code":"","latitude":31.25044,"longitude":-99.25061}],"Pasadena":[{"name":"Pasadena","countryCode":"US","admin1Code":"CA","admin2Code":"037","latitude":34.14778,"longitude":-118.14452}]}

###
#Search the lucene-geo-gazetteer version

## Navigate into the lucene-geo-gazetteer directory
cd lucene-geo-gazetteer

##Search the version in the pom file
grep -m 1 "<version>" pom.xml
##

# Then search the index (e.g., for Pasadena and Texas): java -cp target/lucene-geo-gazetteer-<version>-jar-with-dependencies.jar edu.usc.ir.geo.gazetteer.GeoNameResolver -i geoIndex -s Pasadena Texas
# The version at the time of this edit is 
 <version>0.3-SNAPSHOT</version>
# Replace the version in the command
java -cp target/lucene-geo-gazetteer-0.3-SNAPSHOT-jar-with-dependencies.jar edu.usc.ir.geo.gazetteer.GeoNameResolver -i geoIndex -s Pasadena Texas

#Launch Server
 lucene-geo-gazetteer -server
# Search for Pasadenas, Texas
curl "localhost:8765/api/search?s=Pasadena&s=Texas&c=2"
#output sample
{"Texas":[{"name":"Texas","countryCode":"US","admin1Code":"TX","admin2Code":"","latitude":31.25044,"longitude":-99.25061}],"Pasadena":[{"name":"Pasadena","countryCode":"US","admin1Code":"CA","admin2Code":"037","latitude":34.14778,"longitude":-118.14452}]}


######

#Installing and downloading an NER model

The next thing you'll need is a Named Entity Recognition model for places. The GeoTopicParser uses Apache OpenNLP and with its 1.5 version, OpenNLP provides already trained models for location names in text data. You can download the en-ner-location.bin file already pre-trained by the OpenNLP folks. One thing to note is that OpenNLP's default name finder is not accurate, so building your own NER location model is highly recommended. In this case, please follow these instructions.

The model needs to be placed on the classpath for your Tika installation in the following directory:

org/apache/tika/parser/geo/

#/root (But it depends on home directory choice)

#Make the directory for the ner model
mkdir -p /root/location-ner-model && cd /root/location-ner-model

#download the pretrained model
curl -O https://opennlp.sourceforge.net/models-1.5/en-ner-location.bin

#place the model in the required classpath directory
mkdir -p org/apache/tika/parser/geo/
mv en-ner-location.bin org/apache/tika/parser/geo/


#Download and set up MIME Type configuration
mkdir -p /root/geotopic-mime && cd /root/geotopic-mime
mkdir -p org/apache/tika/mime
curl -L -o custom-mimetypes.xml https://raw.githubusercontent.com/chrismattmann/geotopicparser-utils/master/mime/org/apache/tika/mime/custom-mimetypes.xml
mv custom-mimetypes.xml org/apache/tika/mime

#Download .geot file for testing
curl -L -o /root/geotopic-mime/polar.geot https://raw.githubusercontent.com/chrismattmann/geotopicparser-utils/master/geotopics/polar.geot
curl -L -o /root/geotopic-mime/cnn.geot https://raw.githubusercontent.com/chrismattmann/geotopicparser-utils/master/geotopics/cnn.geot

#Start the lucene-geo-gazetteer server if it is off
cd lucene-geo-gazetteer
lucene-geo-gazetteer -server

#Open another terminal

#Now download the required tika package(app, jar,nlp --version 2.6.0)
 mkdir -p /root/tika && cd /root/tika
 curl -L -o tika-app-2.6.0.jar https://archive.apache.org/dist/tika/2.6.0/tika-app-2.6.0.jar
 curl -L -o tika-parser-nlp-package-2.6.0.jar https://repo1.maven.org/maven2/org/apache/tika/tika-parser-nlp-package/2.6.0/tika-parser-nlp-package-2.6.0.jar
 java -jar /root/tika/tika-app-2.6.0.jar --version
 curl -L -o tika-server-2.6.0.jar https://archive.apache.org/dist/tika/2.6.0/tika-server-standard-2.6.0.jar

#Run the geoparser to process the sample files
java -classpath /root/tika/tika-app-2.6.0.jar:/root/tika/tika-parser-nlp-package-2.6.0.jar:/root/location-ner-model:/root/geotopic-mime org.apache.tika.cli.TikaCLI -m /root/geotopic-mime/polar.geot
java -classpath /root/tika/tika-app-2.6.0.jar:/root/tika/tika-parser-nlp-package-2.6.0.jar:/root/location-ner-model:/root/geotopic-mime org.apache.tika.cli.TikaCLI -m /root/geotopic-mime/cnn.geot

#Output sample
Content-Length: 3164
Content-Type: application/geotopic
Geographic_LATITUDE: 26.0112
Geographic_LONGITUDE: -80.14949
Geographic_NAME: Hollywood
Optional_LATITUDE1: 40.92877
Optional_LATITUDE2: 44.60715
Optional_LONGITUDE1: -74.96032
Optional_LONGITUDE2: -69.04576
Optional_NAME1: New Jersey State Police Troop B Hope Station
Optional_NAME2: Town of Monroe
X-TIKA:Parsed-By: org.apache.tika.parser.DefaultParser
X-TIKA:Parsed-By: org.apache.tika.parser.geo.GeoParser
X-TIKA:Parsed-By-Full-Set: org.apache.tika.parser.DefaultParser
X-TIKA:Parsed-By-Full-Set: org.apache.tika.parser.geo.GeoParser
resourceName: cnn.geot


Additional info 
#run the python file that include the tika module to process the dataset
pyhon your_script.py

######

# Command to see and manage all java version (Well known public command)
update-alternatives --list java
# Show the path of all installed version
/path/to/java -version
#search the system for all java executable
sudo find / -name "java" -type f 2>/dev/null
#Verifiy installed Packages (for package Manager)
dpkg --list | grep -i java
#configure java alternatives (choose from the installed java)
sudo update-alternatives --config java
#verify the version
java -version
#add jdk to the path
nano ~/.bashrc
#add the following lines
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
#save and close the file and apply the change
source ~/.bashrc

####

# Other helpful command
# directories removal (for Linux based OS)
# delete the tika repository
rm -rf ~/tika
# remove the cached maven dependency
rm -rf ~/.m2/repository/org/apache/tika
#verify the removal
ls ~/.m2/repository/org/apache/tika
##
#command to find tika version (Linux based OS)
grep -m 1 "<version>" pom.xml





