# Click_And_Impressions

### Task:    
Script a program for loading the json data source(s) in s3 bucket to a table(s) in Redshift  

  
### Some observations and assumptions from the data:    
  
1. On downloading a sample of 1000 json files from S3, I found there were 2 types of files. One for **advertising impressions** on users visiting the site and the second for **clicks on advertisments** by the users
2. Apart from the common columns in the 2 files, the impressions table has 7 additional columns that are missing from the clicks table. While the clicks table has 2 additional columns, `ClickCount` and `ImpressionId`.
3. Further investigation of the data revealed the `id` column from **impressions** file matched the `impression_id` column in the **clicks** file. And for the rows where these columns matched,the entries across common columns in both files were identical
4. The **clicks** table appears to be a subset of the **impressions** table as it contains all the impressions that were converted into clicks
5. Uploading the data to the Redshift table would require the data to be transfered in chunks as new files are received in the S3 bucket continuously at intervals of every few minutes
6. As direct uploading of the pandas dataframe to Redhift tables takes longer, the dataframe first needs to be uploaded to a separate S3 bucket and then copied from S3 into Redshift


### My stepwise appoach  to the task:

1. Prepared a notebook to create 2 tables in Resdhift DB, **clicks** and **impressions**. The data from json files in S3 bucket would be uploaded into these tables. The **Create_Redshift_Tables.ipynb** notebook is kept separate from the rest of the process as it needs to be run only once   
2. Created another notebook, **LL_Download_Transform_Upload**, to download, transform and upload the data in batches. The step wise process in the notebook is detailed below:     
     i) Downloads batches of 20 files from S3 bucket. Batch size can be changed to optimize the process 
     ii)Performs several steps to transform the raw data into a format that can be easily queried for further analysis.
        - Converts json files to dataframe using ensuring each key in the nested dictionaries is a separate column.       
        - Change date format. 
        - Drops repetitive column `User.Key`. 
        - Extracts data from list columns as comma separated strings.  
        - Renames columns to ensure names are compliant with the Redshift column naming conventions.   
    iii)Upload the processed dataframe to a separate S3 bucket in csv format.    
    iv) Append the processed S3 file to Redshift table. 
3. Delete the processed file from S3 (save on storage space)
