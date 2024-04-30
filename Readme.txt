#Task 1 and 2
##Overview:
This heatmap visualization shows BFRO sightings by county across the United States. Counties with more sightings are represented in darker shades of red.


##Prerequisites:
A web browser that supports HTML5 and JavaScript (e.g., Google Chrome, Firefox, Safari).
Internet access to load D3.js and TopoJSON libraries.


##Files:
heatmap.html: The HTML file that contains the heatmap visualization.
bfro_subset.json: The JSON file containing the BFRO sightings data.
us-counties-states.json: The JSON file containing the US counties and states topology data.
Steps to View the Heatmap:
Ensure that the heatmap.html, bfro_subset.json, and us-counties-states.json files are in the same directory.
Open the heatmap.html file in a web browser.
(you can view the heatmap directly from teh html by going to the directory it is in and using the command: python -m http.server to open a server to view at the link like: http://localhost:8000/heatmap.html
The heatmap should be displayed, showing the distribution of BFRO sightings by county. Hover over a county to see the number of sightings.


##Notes:
The heatmap uses a color scale from white to dark red to represent the number of sightings. Counties with no sightings are shown in white.
The visualization is interactive. You can hover over a county to see a tooltip with the county name and the number of sightings.


##Customization:
If you wish to customize the visualization, you can edit the heatmap.html file. For example, you can change the color scale, adjust the map dimensions, or modify the tooltip content.


#General Instructions for Viewing D3.js Visualizations
##Overview:
These instructions apply to various types of D3.js visualizations, such as bar charts, line charts, pie charts, and scatter plots. Each visualization displays different aspects of your dataset.


##Prerequisites:
A web browser that supports HTML5 and JavaScript (e.g., Google Chrome, Firefox, Safari).
Internet access to load the D3.js library.


##Files:
{visualization_type}.html: The HTML file that contains the D3.js visualization.
bfro_subset.json: The JSON file containing the data to be visualized.


##Steps to View the Visualization:
Ensure that the {visualization_type}.html and {data_file}.json files are in the same directory.
Open the {visualization_type}.html file in a web browser.
The visualization should be displayed, showing the data from the {data_file}.json file.


##Customization:
If you wish to customize the visualization, you can edit the {visualization_type}.html file. For example, you can change the color scheme, adjust the dimensions of the visualization, or modify the labels and tooltips.




# Task 3
## No Python scripts were developed or used for this task, as all commands were run in PowerShell 


## Requirements:
1. Docker Desktop 
2. bfro_subset.json from Tasks 1 and 2


## Step 1: Download Docker Desktop and launch the application. Then, open PowerShell to make sure Docker is running, and run this command in PowerShell to see if Docker is in fact running:
* docker version


## Step 2: Set up Solr in a Docker container by pulling Solr image from Docker Hub. The command to do so is: 
* docker pull solr


## Step 3: Start a new Solr container, mapping the ports and setting the container name to ‘new_solr_container’. The command that was used to start a new Solr container is:
* docker run -d -p 8983:8983 --name new_solr_container solr solr-precreate mycore


## Step 4: Copy the bfro_subset.json file into the running Solr container 
* Navigate to wherever the BFRO JSON file is stored on your local machine
* Transfer the JSON file into the Solr container with this command (change the name of the JSON file in the command if you named the file differently):
   * docker cp .\bfro_subset.json new_solr_container:/var/solr/data/bfro_subset.json


## Step 5: Use the Solr container to ingest the JSON file. Run these commands (if your Solr container and JSON file are named differently, change the names accordingly):
* docker exec -it new_solr_container bash
* cd /opt/solr/bin
* ./solr post -c mycore /var/solr/data/bfro_subset.json


## Step 6:  Test querying your Solr core to verify the data ingestion
* curl "http://localhost:8983/solr/mycore/select?q=*:*&wt=json&indent=true" 


## Step 7: Query and save the Solr results in another PowerShell terminal. In the Solr results JSON file, look under the "params" section to observe what the "rows" parameter is set to. For this file, the "rows" parameter was set to "5112". The "rows" parameter was then set to "5112" in the Solr query to ensure that the full dataset was obtained in the output file. Execute these commands (navigate to the appropriate directory, check the ‘rows’ parameter, and name the output file whatever you want):
* cd PycharmProjects/DSCI550/Assignment3
* $response = Invoke-WebRequest -Uri “http://localhost:8983/solr/mycore/select?q=*:*&wt=json&indent=true&rows=5112”
* $response.Content | Out-File -FilePath "solr_results.json"


## Step 8: Tar up and gzip the Solr index using commands in PowerShell (technically this task is labeled under Task 4c, but tarring up and gzipping the Solr index can be done right after querying and saving the Solr results)


## Step 9: Enter the Solr container's bash shell. Then, navigate to the directory containing the Solr index files within the container. Commands that were used for this:
* docker exec -it new_solr_container bash
* cd /var/solr/data/mycore/data


## Step 10: Use the tar command to create a compressed tarball (mycore_index.tar.gz) of the index files
* tar -cvzf mycore_index.tar.gz index


## Step 11: Exit the container’s shell, and then navigate to the directory where you want to copy the tarball (will differ from the directory I personally navigated to). Copy the tarball from the container to a specified directory in your local filesystem using the docker cp command.
* exit
* cd PycharmProjects/DSCI550/Assignment3
* docker cp new_solr_container:/var/solr/data/mycore/data/mycore_index.tar.gz "C:\Users\kathe\PycharmProjects\DSCI550\Assignment3\mycore_index.tar.gz"  




# Task 4
## Requirements: 
1. Image Space application
2. Docker Desktop
3. BFRO images


##Step 1: Installation and Setup
1. Ensure Docker Desktop is installed on your system.
2. Clone the Image Space application repository from https://github.com/nasa-jpl-memex/image_space/wiki/Quick-Start-Guide-with-ImageCat
3. Place the BFRO images in a folder named 'Images' within the Image Space directory.


##Step 2: Docker Configuration
1. Configure Docker Desktop settings according to the requirements of the Image Space application.
2. Allocate sufficient memory and resources to Docker Desktop to ensure smooth operation of the Image Space application.
3. Set up directory paths for Image Space and the BFRO images/posts dataset within Docker Desktop settings:
```export DOCKER_DEFAULT_PLATFORM=linux/amd64```
```export IMAGE_DIR=/opt/dsci550/image_space/Images```


##Step 3: Prerequisite Docker Images Installation
1. Pull the necessary Docker images required for running Image Space, including Mongo, Postgres, and Solr:
```docker pull mongo:3.0```
```docker pull postgres:9.5.10```
```docker pull solr:6.0```
2. Verify that the Docker images are successfully downloaded and available for use.


##Step 4: Deploy Image Space
1. Follow the instructions provided in the Image Space documentation to deploy the application using Docker Compose. Define the network by executing this command in the terminal:
```docker network create deploy_imagespace-network --label "com.docker.compose.network=default``` 
2. Ensure that all services required for Image Space, including ImageCat and SMQTK, are properly deployed and running:
```cd imagespace_smqtk./smqtk_services.run_images.sh --docker-network deploy_imagespace-network
--images /opt/dsci550/image_space/Images```
3. Verify the deployment by accessing the Image Space application through the provided URLs and interfaces, then execute the last step in the Quick Start Guide from the root ImageSpace directory:
```cd scripts/deploy```
```sh ./imagespace/enable-imagespace.sh```


##Step 5: Ingest BFRO Images
1. Use the provided scripts or write custom scripts to ingest the BFRO images/posts dataset into Image Space.
2. Monitor the ingestion process and ensure that all images are successfully indexed and available for search and analysis within Image Space.


##Step 6: Explore Similar Images
1. Utilize the search functionalities provided by Image Space, such as similarity search using SMQTK.
2. Browse and find similar images/posts within the BFRO dataset to identify patterns or common themes in Bigfoot sightings.


###Step 7: Get ImageCat indices
1. Open the imagespace-imagecat container on docker, and the indices will be in the path of: '/deploy/solr4/example/solr/imagecatdev/data/index' on docker.


###Step 8：Tar and grip the Solr or ElasticSearch index
1. Enter the Docker container's command-line interface using the docker exec command:
```docker exec-it new solr container bash```
```cd /var/solr/data/mycore/data```
2. Run the tar command inside the container:
```tar -czvf mycore_index.tar.gz index```
3. After creating the tar.gz file, exit the container, and then use docker cp from PowerShell on your host machine to copy the tar.gz file from the container to your host system:
```docker cp new solr container:/var/solr/data/mycore.tar.gz "C:\\dsci550\assignment3\mycore.index.tar.gz```


# Task 5
## Requirements: 
1. Memex GeoParser
2. Tika Python
3. Lucene-geo-gazetteer
4. Use a Python 2.7.18 environment


## Step 1: Split the csv file into json files
We met some difficulties in adding the indexes from Solr to Memex GeoParser. I tried to reinstall GeoParser and use another person’s laptop to perform the work, but still can’t make it work. So I choose to directly upload the json files, which contain text that could contain location information, to GeoParser and get visualizations. Therefore, the first step is to convert the csv to json, and split it for each report. To do this, run the jupyter notebook split_json.ipynb. This will randomly sample 500 reports, select and combine the fields that could contain location information into text, and save them into 500 separate json files. These json files will be stored in a folder called “jsons” in the Data folder. 


Important Note: the “Data” folder already contains a folder called “jsons”. This folder contains the 500 json randomly chosen files that were used to create the visualizations in the report. If you want to get the exact same visualization, please use the files in this folder. Before you run the split_json.ipynd and create another 500 randomly chosen json files, backup and clear the json files that are already in the jsons folder. 


## Step 2: Install Memex GeoParser
1. Install Docker Desktop
Download Docker Desktop from https://www.docker.com/products/docker-desktop/, and install it. 


2. Install and pull Memex Geoparser
To do this, run the command ```docker pull nasajplmemex/geo-parser```. And then git clone GeoParser by running the command
```git clone https://github.com/nasa-jpl-memex/GeoParser.git```


## Step 3: Run GeoParser
In order to run the GeoParser server, cd to where you cloned GeoParser and cd to Docker, you can do this by running something like ```cd $HOME/GeoParser/Docker```. 


Then, you should be able to visit the GeoParser interface by visiting http://localhost:8000. In the interface, click on the upload files icon, then upload all the files in the json folder except the all_doc.json. The GeoParser will then start to parse the locations, get the latitudes and longitudes of these locations and show them on the map. This will take about 10 minutes to process.