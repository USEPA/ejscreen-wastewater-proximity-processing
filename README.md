# **Generating EJScreen Wastewater Proximity**

EJScreen uses Apache Hadoop pig scripts to generate wastewater discharge proximity. The Pig scripts were developed using Esri's [GIS Toolkit for Hadoop](https://esri.github.io/gis-tools-for-hadoop/) toolkit. It was run in an AWS EMR cluster environment. The source data came from EPA's Office of Pollution Prevention and Toxics (OPPT). The proximity process involves Pre-Hadoop processing, running Hadoop Pig scripts, and Post-Hadoop processing. The end results are Census block-group based proximity scores.

**Sources:**

The wastewater discharge indicator takes into account pollutant loadings from the Discharge Monitoring Report (DMR) Loading Tool (which include NPDES DMR discharges and TRI releases) for toxic chemicals reported to the Toxics Release Inventory. The data were input into the RSEI model (Version 2.3.10) to incorporate chemical toxicity and fate and transport in order to estimate concentrations of pollutants in downstream water bodies (i.e., stream reaches) and derive a toxicity-weighted concentration. The stream reaches are from the National Hydrography Dataset (NHD Version 2.0). Coverage is for NHD Region 01 through Region 21, but does not include Alaska Region 19.

**Pre-Hadoop Processing:**

- Re-project national feature class (EJScreen2020) to WGS 1984 and Rename to: DMRres\_WGS84
- Create Regional Subsets: Select DMRS\_WGS84 by NHD Region to create feature classes: DMRres01, … , DMRres18, DMRres20, DMRres21
- Export to JSON using ESRI Hadoop toolbox for each NHD region (DMRres01.json, … , DMRRes18.json, DMRRes20.json, DMRRes21.json). Note that toolbox is provided by Esri geoprocessing-tools-for-hadoop-master/HadoopTools.py (use Features to JSON with default settings).
- Upload each JSON file to own s3 folder. For example: s3://ejscreen2023/WaterProximity/Waterlines/DMRres01/DMRres01.json

**AWS Hadoop Processing:**

- Start an AWS EMR cluster.
- Step 0: Convert each Region JSON file (with DMRres shapes) to Hive data structures. For example: s3://ejscreen2023/WaterProximity/Waterlines/DMRres\_01 to s3://ejscreen2023/WaterProximity/Waterlines/DMRres\_01\_hive/. See **WP\_Hive\_01.txt** for sample Hive code.
- Run step 1 for each Region. See **WaterProximity\_Step1\_01.txt** for Pig script example. Note that for each Region, the block pop weight table includes a selection of States overlapping the Region.
- Run step 2 for each Region. See **WaterProximity\_Step2\_01.txt** for Pig script example.
- Use Athena to create BG-level results tables and export BG\_Scores\_01.csv, etc. to the OutputfromHadoop folder.
- Repeat all steps for each Region.

**Post-Hadoop Processing:**

- Combine all BG score text files in OutputfromHadoop folder into one file (Wastewater\_BG\_Scores\_US.csv).
- Prep with text editor (Capitalize first header row and remove all other header rows).
- Import US csv file to Excel; make sure BLKGRP is text.
- Port US Excel file into geodatabase table (Wastewater\_Work/Wastewater\_BG\_Scores\_All).
- Some BG's are processed in more than one NHD region, so use the Frequency tool on BLGRP with sum(blkgrp\_score) to produce Wastewater\_BG\_Scores\_Final.
- Provide datasets for testing. First Create Wastewater\_Testing.gdb.
- Include Wastewater\_BG\_Scores\_all and Wastewater\_BG\_Scores\_Final tables.
- Add US\_WastwaterProx\_BG with BG shapes.
- Rename columns to STCNTRBG and BG\_SCORE, set NULL Scores to 0.

**EPA Disclaimer**

The United States Environmental Protection Agency (EPA) GitHub project code is provided on an "as is" basis and the user assumes responsibility for its use. EPA has relinquished control of the information and no longer has responsibility to protect the integrity, confidentiality, or availability of the information. Any reference to specific commercial products, processes, or services by service mark, trademark, manufacturer, or otherwise, does not constitute or imply their endorsement, recommendation or favoring by EPA. The EPA seal and logo shall not be used in any manner to imply endorsement of any commercial product or activity by EPA or the United States Government.
