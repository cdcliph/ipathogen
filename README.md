# **iPathogen Docker Image Deployment Guidance**

# 1.   Pre-deployment preparation

## 1.1  Install Docker

​    Command reference

```
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh
```

## 1.2  Create a project folder

Create project folders to store project data, databases, and process output.

Example:

```
mkdir pathogen-identification
```

## 1.3 Download the database library

Data needs to be downloaded to the project folder created in 1.2. Note the database location:

For details of the database configuration, see the library documentation.

```
/{path}/pathogen-identification/library
```

## 1.5 Prepare sequencing data

Put the test data into the project folder. (Mand-Do)

Supported formats: fq, fq.gz

# 2.   Import the image

Use the docker import command to import this image file: (replace path with the path to your local server)

```
docker import - pathogen-identification < ipathogen-20230626.tar 
```

## 2.1 Check step: View the local image

```
docker image ls | grep ipathogen
```

![img](https://github.com/cdcliph/ipathogen/blob/master/png/2.1.png)

# 3.   Start the image as a container

```
docker run -itd --name container_name -v /{path}/pathogen-identification:/pathogen-identification pathogen-identification/ipathogen_pe: v1.0.0
```

The container name,can be modified customizably, if modified, you need to modify the container name in the subsequent commands (marked in red) synchronously

**-v**: host folder: /folder inside container. Mount the data volume, which can be used to directly call the data on the host in the container. Note Change the host folder path to the folder path you created in Step 1.3. 

## 3.1 Check step: Check whether the container is started successfully (the list of containers appears in the call).

```
dockerps
```

![img](https://github.com/cdcliph/ipathogen/blob/master/png/3.1.png)

# 4.   Run the pathogen identification process

```
docker exec -itdpathogen  /bin/bash -c "source /home/env.sh && python -u /home/full_pipeline/full_pipeline.py -1 "simulate_viral_with_host_1.fastq" -2 "simulate_viral_with_host_2.fastq" -o "full_pipeline_out_viral_simulate_test" -l "library" -p "/home/full_pipeline/packages" -c 8 --map_ host "false" --host_data_folder "library/hg38_data" --host_index "human.index" --viral_top_n 3 --mlst_top_n 3 > ipathogen.log 2>&1"
```

 

The above command, italics and underscored parts are modifiable parameters parts, the rest of the parts are not recommended to be modified, may throw errors.

The log output of the process is stored in a file called iPathogen.log under the folder created in step 1.3, which follows the real-time update of the program and can be viewed through less. 

The program parameters are described as follows: 

| Parameter name     | Corresponding content                                        |
| ------------------ | ------------------------------------------------------------ |
| -1                 | run1 file path                                               |
| -2                 | run2 file path                                               |
| -o                 | Process output folder  path (see Note 1)                     |
| -l                 | Library folder path                                          |
| -p                 | packages folder path  (no need to modify)                    |
| -c                 | Number of CPU cores                                          |
| --map_host         | Whether to perform host  alignment (optional value true/false/t/f). |
| --host_data_folder | Host folder  Format: library folder  path / host folder name (no need to modify the host) |
| --host_index       | Use the host file index  generated by bowtie2-build (no need to modify the host) |
| --viral_top_n      | Viral process analysis  of the top n species in the number of unique alignment sequences |
| --mlst_top_n       | The MLST library  identifies the top n species in the number of unique alignment sequences in  the identification process |

 **Note 1:**

To prevent typosis from affecting the file structure, the folder that is parent of the output folder path must exist for creation to succeed.

Example: If the output folder does not exist, the first parameter cannot be successfully created

（1）-o./output/full-pipeline-output

（2）-o full-pipeline-output

**Note 2:**

The paths of the commands in the use case are all relative paths, omitting the path followed by pathogen-identification (that is, the project folder created in step 1.3), it is recommended to store the input and output in the pathogen-identification folder to ensure that the files are stored on the host to prevent file loss. 

 After that, if you have new data to run, store it in the project folder created in step 1.3 and start from step 4. 

# 5.   Introduction to the output result file

## 5.1 Locate the folder result _files in the set -o output folder path


| File name                      | Content                                                      |
| ------------------------------ | ------------------------------------------------------------ |
| centrifuge_report.tsv          | Corresponds to the centrifuge report                         |
| viral_trace_analyze_ref.blast  | Corresponding virus trace analysis BLAST  comparison results |
| viral_trace_analyze_result.csv | Corresponding to the statistical results of  virus trace analysis |
| viral_iteration_identities.csv | Corresponding to the virus iterative assembly  identity results |
| *type_info.tsv                 | MLST library typing information corresponding  to the corresponding species |
| *taxonomy_info.tsv             | The 16s library level information  corresponding to the corresponding species |
| *matched_id..txt               | Corresponds to the corresponding species vfdb,  card library comparison information |



## 5.2 Introduction to the specific output structure

centrifuge_outfolder: Storage of the centrifuge output

map_host folder: Storage of the host mapping output

viral_outfolder: Storage of the output of the virus analysis process

viral_out/trace_analysis: Storage of virus trace analysis output

viral_out/iterate_*: Storage of the results of the iterative assembly of the nth loop of virus analysis

mlst_out, 16s_out, vfdb_out, card_out: folders store the corresponding database analysis pipeline output separately
