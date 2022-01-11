
## A High Level View
This README will walk you through all the steps required to run the HCOT SAP HANA storage validation test.  SAP HANA is NOT used in this validation,the workload is synthetic.  

Included are:
* Details on the included files and how to use
* Configuration Details (Instance Types, OS Distros, Mount Options, Tunables)
* Details on how to run the test
 
### Included Files:
* Six files were included in this REPO:
    * README.md
    * hcmtsetup.exe,   
    * two execution plan json files,  
    * HCMT_V201_TDI_10K_adapted.xlsx, 
    * sysctl.conf
* hcmtsetup-50beta.exe 
    >  a type of compressed archive, execute it to extract all pre-packaged files from SAP
    >  Extraction of the files from the hcmtsetup is as simple as ./hcmtsetip-50beta.exe
    >  All files will be extracted to the pwd, the most important file to you is ./hcmt which is the tool you will need to use. 
    >  More on that in a minute.
 * executionplan.json files
    > these are execution plans used by ./hcmt, more on how to use in a minute
    > The json executionplan-data-64.json is a short test which you should use to ensure the test
    > suite works in your env, its fairly short. The json executionplan-storage-full.json is the 
    > ACTUAL storage test, this test takes about 3 hours per iteration
* HCMT_V201_TDI_10K_adapter.xlsx 
    > The workbook where in all results are to be loaded into.
    > Open the included .xlsx file and see follow steps 2 onward as shown in the How To Use Tab of the workbook. 
   
## Configuration Requirments:
### Instance Types:
* Azure: m1_128s
* GCP: n1-highmem-32
* AWS: m.24xlarge | m6.24x | i4.x?
### Linux Distro:
* SLES
### Mount Options: 
* for SLES15SP2+
    ``` 
    nfs nointr,vers=4.1,rsize=262144,wsize=262144,bg,nconnect=[8 or 16 seem the best so far]
    ```
* Earlier versions
    ```
    nfs nointr,rsize=262144,wsize=262144,bg
    ```
### Packages:
* sysstat
* python

## How To Modify The Execution Plan JSON Files:
The contents of both json files look something like below, pay attention to 0th, 1st, 2nd, and 4th posiitons in the list
  
### 0th and 1st: Mount Points:
* replace the value associated with the key "Value" with your SAP HANA log and data path respectively.
* As the tests are run one at a time, the same mount point may be used for both.

 ### 2nd: Scale-Up or Scale Out:
* Only fill in the value associated with the "Value" key if you plan on doing a scale out test, 
* leave it empty if the test is scale up.
* The value, if used, is in the format of "<ip | hostname>,<ip | hostname>,..."

 ### 4th: Test Repeat Count: 
 * Replace the value associated with the key "Value" with the number of tests to run
  
 ### Sample Content From Included Execution Plans
```
  {
     "Variables": [
       {
       "Comment": "The <Value> should be adapted to the HANA log volume path",
       "Name": "LogVolume",
       "Value": "/mnt/kpi",
       "Request": "false"
        },
       {
       "Comment": "The <Value> should be adapted to the HANA data volume path",
       "Name": "DataVolume",
       "Value": "/mnt/kpi",
       "Request": "false",
       "Profile": "LNX"
       },
       {
       "Comment": "Hosts for scale-out measurements, keep this empty for scale-up",
       "Name": "Hosts",
       "Value": "",
       "Request": "false"
       },
       {
       "Comment": "Persistent memory mount pathes, keep this empty fo non nvm systems",
       "Name": "NvmBasePath",
       "Value": "",
       "Request": "false"
       },
       {
       "Comment": "The <Value> is the general test repeat count",
       "Name": "TestRepeatCount",
       "Value": "3",
       "Request": "false"
       },
```

## How To Run:
### The quick validation run:
```
suse12sp4:/home/mchad# ./hcmt -v -p executionplan-data-64.json  
```
### The ACTUAL storage test:
```
suse12sp4:/home/mchad# ./hcmt -v -p executionplan-storage-full.json  
```
## The Results Of Each Test:
Each test will create a zip file named for the YearMonthDayHrMinSec to the pwd such as 
hcmtresult-20190815142636.zip. The results of each round of testing will end up in  the
Results/ directory as shown below, so you may want to move each hcmtresult*.zip file to its 
own directory to keep the unziped files from clobbering one another

### Contents of zip file:
_Note_ The results json files will always contain A300F54B104B6ADB.json and A904F304AEB67906.json in the their names
```
manifest.json
hcmtresult-20190815142636.zip
config/
Results/
       168B7333-86D4-1334-A300F54B104B6ADB.json
       D664D001-933D-41DE-A904F304AEB67906.json
RawInfo/
```
### How To Extract Content From The Results JSON Files:
```
  for file in `ls Results/*A904F304AEB67906.json`; do 
      echo $file 
      cat $file 
  done > test.txt

  for file in `ls Results/*A300F54B104B6ADB.json`; do
      echo $file 
      cat $file 
  done >> test.txt
```

### How To Use The Data Above:
The content in the test.txt file from above is used to popluate column A in worksheet "Raw Result HCMT" in the workbook
