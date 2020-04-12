## Constant Warning Times
### Introduction
This is the first in a possible series of essays on the general topic of constant warning time.  For the past several years, I’ve been working on a project that involved constant warning time devices used at 4-quad gates.  The project is generally focused on regulations and definitions in 49 CFR 234 and the referenced MUTCD.
Prior to offering opinions on warning times, warning time devices, regulations, etc., the first task and subject of this effort is to gather data to assess the effectiveness of 4-quad gate in reducing accidents and fatalities.  I will summarize the results in the conclusion, but my primary purpose is to document an automated approach to retrieving data from FRA’s crossings and accidents databases.
## Background
A 4-quad gate has at a minimum four barrier arms that block the entrance and exit approaches at a highway railroad grade crossing.  The purpose is to stop motorist from using the opposing traffic lane to bypass the gate and drive around a typical two-barrier gate on a two-lane road.  Unfortunately, this maneuver is frequently the cause of fatalities at gated crossings.  
In addition to the barrier arms, most 4-quad crossings have supplemental safety features to discourage poor driver behavior.  Vehicle fatalities at 4-quad gates are as shown in the data below rare and are often deliberate actions of the vehicle operator.
The North Carolina sealed corridor report studied various improvements to crossings including the upgrade of 69 crossings to 4-quad gates. The Illinois High-Speed Rail Four-Quadrant Gate Reliability Assessment focused on adequate warning times and exit detection for high-speed operations.  Both studies confirmed the benefit of 4-quad gates and greater improvement with other supplemental safety measures to “seal” the crossing to drive arounds.  These studies were completed in 2009 with a relatively small number of 4-quad gates. 
## Data 
To get current experience, FRA has an online database of accidents at crossings among other accident and fatality reports.  These data can be queried online by year, railroad, and other predefined filters.  When querying FRA’s Highway Crossing Accident Database, you can obtain a list of crossings that meet FRA preselected filter criteria.  Unfortunately, the query download page is limited in the extent of filtering allowed.  This limitation can often be overcome by exporting a larger set of results to a Excel file and then applying additional Excel filters to the data.  For my specific case, I was looking for fatalities involving highway vehicles at gated crossings.  After downloading all accidents for a specific year, I filtered the following fields:
-	TYPEVEH   # I excluded K – Pedestrians
-	SIGNAL   # Gates, I included entries 1-6 and excluded blanks
-	TOTKLD # Casualties/Killed – I excluded 0

Additionally, I wanted to know how many occurred at 4-quad gates and how many were in quiet zones and or had constant warning devices.   These data are not recorded in the accident report but are part of the crossing inventory.  FRA provides a tool to download a specific crossing’s data by its Id, but if you have a large list, it is a tedious process to enter and scan each report for the desired features.

To automate the process, the FRA Developer website at https://safetydata.fra.dot.gov/MasterWebService/PublicApi has an API to access crossing inventory.  At present and from the samples offered, I found most of them limited in the number of crossings that could be returned in any one request.  The single crossing API worked well and returned a JSON object of all that crossing’s fields.
To use the API, you need to have a free FRA account and a token.  https://safetydata.fra.dot.gov/MasterWebService/PublicApi/Tokens

For this specific task statement, I am querying the inventory database and looping through a list of crossings.  I extract these data for each:
-	the number of gates,
-	gate configuration
-	quiet zone T/F
-	train detection system

FRA also tracks activation failures reported by the railroads.  Where such failures result in fatalities, this data can provide the root cause of the failure. 

## Geek Stuff - Automation and Sudo code 
I first looked at using Excel and PowerBi to call the API but found that looping through a crossing id filter in the API/query, was beyond my Excel skill level.  I then chose to use Python but expect that a JavaScript solution would be equally simple.

Step 1:  Save the column of crossing ids from the accident excel query to a separate worksheet.  Save that worksheet in Excel as a csv.

Step 1a: Open the file in Excel and check for duplicate entries.  Eliminate any duplicates 
(multiple accident reports at the same crossing) so the crossing Id can be used as a key to merge the data.

Step 2: Import the following modules into your python code:  csv, json and requests

Step 3:  Read the csv file created in Step 1.  Skip the header record and save the ids in a list

Step 4:  Open a file for writing/output and write header row.

Step 5: Loop through the Id list calling the API and extracting the following fields from the JSON object:

| Field Names  | Values                                                                                                             |
|--------------|--------------------------------------------------------------------------------------------------------------------|
| Gates        | Number of gate arms                                                                                       |
| Gate Conf    | 1 = 2 Quad; 2 = 3 Quad; 3 = 4 Quad                                                                                 |
| Whstban      | Quiet Zone 0= none, 1= 24hr  2 = Partial 3 = Chicago Excused     |
| WdCode       | Warning device code 1 = No signs or signals; 2 = Other signs or signals; 3 = Crossbucks; 4 = Stop signs; 5 = Special Active Warning Devices; 6 = Highway traffic signals, wigwags, bells, or other activated; 7 = Flashing lights; 8 = All other Gates; 9 = Four Quad (full barrier) Gates                                                                                                                |
| SpselIDs     | Train Detection 0 = None 11 = Constant Warning Time 12 = Motion Detection 14 = Other 16 = AFO 17 = PTC 18 = DC      

Step 6: Save in a csv text object and write text to file.

Appendix A is the Python code in a Jupyter Notebook.  A source copy is available at … https://1drv.ms/u/s!AgsEoWikEEW_ntl8zQvKaZVNNhPW5g?e=QnwKjg.  This link requires that you have access to a Jupyter notebook.

## Other notes:
- I found the API response to be rather slow and to take about 15 seconds per crossing.  Terminal window shows progress.
- If you are new to Python, there is very little obscure code.  To extract other fields simply find the filed name and add it to the extracted field variables.  Formatting the variable 'token' was the only tricky part.  Since the token is assigned to me personally, I have changed it to a non-working value.  Simply replace it with your token.  
- Each request has a 10-15 second cycle.  If you have many crossings, be sure to check your computer’s sleep and logoff settings.
## Summary of aggregated data
Trespassing/pedestrian fatalities can occur anywhere along the track.  The Crossing Accident database reports both pedestrian and in-vehicle fatalities.  For the tables below At gates, 4-quad, and activation failures represent only in-vehicle fatalities.

### Crossing fatalities by year
| Year | Fatalities | Pedestrian | In Vehice | At gates and in vehicle | 4-Quad and  in vehicle | Activation Failure |
|------|------------|------------|-----------|-------------------------|------------------------|--------------------|
| 2019 | 329        | 105        | 224       | 80                      | 1                      | 0                  |
| 2018 | 300        | 137        | 163       | 95                      | 5                      | 0                  |
| 2017 | 309        | 134        | 175       | 96                      | 0                      | 0                  |
| 2016 | 300        | 125        | 175       | 83                      | 1                      | 0                  |
| 2015 | 287        | 130        | 157       | 102                     | 5                      | 2                  |

## Observations
Without adjusting for bias, it appears that gates and in particular 4-Quad gates are very effective in reducing fatalities at crossings.  Activation Failures, irrespective of type, are not a significant cause of fatalities.  Pedestrian deaths at crossings are significant.
To adjust for bias, we need to know the following:
- Percentage of gated crossings to total by year
- Percentage of 4-Quad crossings to total year

With these adjustments, ....

### Adjusted crossing fatalities by year 2015 = 100
| Year | At gates and in vehicle | 4-Quad and  in vehicle | Activation Failure |
|------|------------|------------|-----------|-------------------------|------------------------|--------------------|
| 2019 |                         |                        |                    |
| 2018 |                         |                        |                    |
| 2017 |                         |                        |                    |
| 2016 |                         |                        |                    |
| 2015 | 100                     |                        |                    |

## 4-Quad Observations
The accident reports contain a narrative section and often describe known details of the accident and fatality.  For the five-year period and at 4-Quad gates, three fatalities were from vehicles stuck between the gates, two were from vehicles going under the gates, and one from vehicles entering under the exit gate.

While not statistically significant, the 4-Quad data point to some known issues at these gates.  First is the use of median barriers.  If drivers are prevented from using the delayed exit gate as an access point, the safety of the gate is improved.  Unless constrained by road infrastructure, median gates should be required at 4-Quad gates.  Second is detection loops.  For these data it is unclear if the gates had functioning detection loops.  “Stuck between the gates” could simply mean the vehicle was between the two entrance gates.

Third is the trade off in length of warning times and exit gate delays.  Considerable study has been given to exit gate behavior and design – among the design features are delays.  Typically exit gates are delayed by four to 10 seconds and contain detection loops to allow a vehicle to exit without being trapped.  The Illinois High-Speed Rail Four-Quadrant Gate Reliability Assessment is the most comprehensive recent study.  Due to high speed trains, Illinois increased the warning time to 80 seconds, well beyond the maximum recommended time.  With an 80-second warning time and when a vehicle detected in the crossing, a civil-speed restriction command can be issued to allow the high-speed train to stop if necessary.  The 80 seconds also gives a disoriented or panicked driver the ability to recover.

Since detection loops fail particularly for bikers, a longer warning time and exit-gate delay would be worth consideration.

## Constant Warning Times
Constant warning time devices are required at all 4-Quad gates, but none of the fatalities reported a failure of the CWT devices.  Nothing in these data point to constant warning times as a factor that affects crossing safety at 4-Quad gates.  The Illinois HSR approach to use 80 seconds contradicts traditional thinking that anything in excess of 60 seconds is a safety risk. 



 
 





# Appendix A

## Background and Instructions

When querying FRA’s Highway crossing Accident Database, you can obtain a list of crossings that meet FRA preselected filter criteria.  Unfortunately, the query download page is limited in the extent of filtering allowed.  This limitation can often be overcome by exporting the results to a excel file and then applying filters to the data.  For my specific case, I was looking for fatalities involving highway vehicles at gated crossings.  I filtered the following fields:
-	TYPEVEH   # I excluded K – Pedestrians
-	SIGNAL  #  Gates, I included entries 1-6 and excluded blanks
-	CASKLDRR # Casualties/Killed – I excluded 0

For 2018, this resulted in an 82-record dataset.

Additionally, I wanted to know how many occurred at 4-quad gates and how many were in quiet zones and or had constant warning devices.   These data are not recorded in the accident report but are part of the crossing inventory.  FRA provides a tool to download a specific crossing by its Id, but you have a large list, it is a tedious process to enter and scan each report for the desired features.

To automate the process, the FRA Developer website https://safetydata.fra.dot.gov/MasterWebService/PublicApi has an API to access crossing inventory.  At present and from the samples offered, I found most of them limited in the number of crossings that could be returned in any one request.  The single crossing API worked well and returned a JSON object of all that crossing’s fields.

To use the API, you need to have a free FRA account and a token.  https://safetydata.fra.dot.gov/MasterWebService/PublicApi/Tokens

For this specific task statement, I am querying the inventory database and looping through a list of crossings.  I extract these data for each:
-	the number of gates,
-	gate configuration
-	quiet zone T/F
-	train detection system

I first looked at using Excel and PowerBi to call the API but found that looping through a crossing id filter in the API/query, beyond my excel skill level.  I then chose to use Python but expect that a javascript solution would be equally simple.

Python code below with comments.  Formatting the variable 'token' was the only tricky part.
Each request has a 10 to 15-second cycle.  If you have a large number of crossings, be sure to check your sleep settings. 

Other respurces:
Crossing Field/Metadata Definitions: https://safetydata.fra.dot.gov/MasterWebService/SecureAPI/Support/Datasets?ModelType=Crossings
Crossing Accident Field Definitions: https://safetydata.fra.dot.gov/OfficeofSafety/publicsite/downloadFStructure.aspx



```python
import requests
import json
import urllib3
import csv
urllib3.disable_warnings()  # The FRA API issues a security warning that the data transmission is not secure.  
```


```python
# Store constants
baseURL = "https://safetydata.fra.dot.gov/"
crossingParam = "MasterWebService/PublicApi/frads/v1/odata/gcis/Crossings?"  # Splitting the long url
tokenParam = "505d2408................"  # FRA supplied token.  Needs to be hardcoded below
crossingList = ['628168G','272584V','751254M']  #Test input data
inputFile = 'C:/Users/Admin/OneDrive/Consulting/DTPvRTD/Incidents/2015Xings.csv'
outputFile = 'GCAccidentXing2015Out2.csv'
firstRecord = True
```


```python
with open(inputFile, newline='') as csvfile:
    csvReader = csv.reader(csvfile, delimiter=' ', quotechar='|')
    next (csvfile)  #Skips header line
    with open(outputFile, 'w') as csvfile1:
        print('ID,Gates,GateConf,Whstban,WdCode,SpselIDs', file= csvfile1)  # print desired header
        for  index, xing in enumerate(csvReader):
            try:  # "try except" block to ignore two error conditions:
                  #  valid crossing but no inventory data and no entries in fields selected
                print(str(xing[0]), index+1, ' of ?')  # current crossing id and position in request list
                                                       # csvReader is an iterator and does not have a length property 
                token = {'$filter':"CrossingId eq '"+str(xing[0])+"'",'token':"505d24081.................."}
                response = requests.get(baseURL+crossingParam, params=token, verify=False)
                response = json.loads(response.text)
                if firstRecord:   # Print entire json object for first crossing.
                    firstRecord = False
                    print(response)
                gateNum= str(response['value'] [0]['Gates'])  # See json record.  'value' is an array of all fields
                # gateCT= str(response['value'] [0]['GateConfType'])  # In the spec but not in the database??
                gateC= str(response['value'] [0]['GateConf'])  # convert json value to a string
                qz = str(response['value'] [0]['Whistban'])    # convert json value to a string
                cwt = str(response['value'] [0]['WdCode'])     # convert json value to a string
                wt = str(response['value'] [0]['SpselIDs'])    # convert json value to a string

                print(str(xing[0]),gateNum,gateC,qz,cwt, wt)
                entireRow = str(xing[0])+","+gateNum+","+gateC+","+qz+","+cwt+","+wt # format as csv
                print(entireRow, file=csvfile1)  # save record to file
            except:
                print("Oops!  No/bad data.  Next crossing")
        print("Finished")
```
Output
    725401E 1  of ?
    {'odata.metadata': 'https://safetydata.fra.dot.gov/MasterWebService/PublicApi/frads/v1/odata/gcis/$metadata#Crossings', 'value': [{'ReportBaseId': 4519050, 'MultipleFormsFiled': 0, 'ReportingAgencyId': 1622, 'ReportingAgencyTypeId': 1, 'RevisionDate': '2019-11-11T00:00:00', 'PostmarkDate': None, 'ReasonId': 14, 'CrossingId': '725401E', 'CrossingIdSuffix': 'NS', 'ReportStatus': 'Published', 'ReportType': 'Major', 'CreatedDate': None, 'CreatedBy': 'ENDCCrossings@nscorp.com', 'LastUpdatedDate': '0001-01-01T00:00:00', 'Railroad': 'NS', 'StateCD': '01', 'CntyCD': '01073', 'Nearest': '0', 'CityCD': '010730330', 'Street': '14TH ST', 'BlockNumb': '', 'Highway': 'SR 150', 'SepInd': '2', 'SepRr1': '', 'SepRr2': '', 'SepRr3': '', 'SepRr4': '', 'MultFrmsFiled': '2', 'SameInd': '1', 'SameRr1': 'ATK', 'SameRr2': ' ', 'SameRr3': ' ', 'SameRr4': ' ', 'RrID': '', 'Ttstn': '472600', 'TtstnNam': 'BIRMINGHAM', 'RrMain': '-1', 'XingOwnr': '-1', 'TypeXing': '3', 'XPurpose': '1', 'PosXing': '1', 'OpenPub': '', 'TypeTrnSrvcIDs': '11,12', 'DevelTypID': '13', 'XingAdj': '2', 'XngAdjNo': '', 'Whistban': '0', 'WhistDate': None, 'HscoRrid': '-1', 'Latitude': '33.3953043', 'Longitude': '-86.9548082', 'LLsource': '1', 'RrNarr1': '', 'RrNarr2': '', 'RrNarr3': '', 'RrNarr4': '', 'StNarr1': '', 'StNarr2': '', 'StNarr3': 'State Phone# updated - date updated: 2020-02-24', 'StNarr4': 'AGS', 'PolCont': '8009464744', 'RrCont': '8009464744', 'HwyCont': '3342426234', 'OperatingRailroadCode': 'NS', 'OperatingRailroadType': 'Primary', 'RrDiv': 'ALABAMA', 'RrSubDiv': 'AGS SOUTH', 'Branch': '#N\\A', 'PrfxMilePost': 'AG', 'MilePost': '0154.900', 'SfxMilePost': ' ', 'Lt1PassMov': '2', 'PassCnt': '2', 'DayThru': '20', 'NghtThru': 9.0, 'TotalSwt': 8.0, 'TotalLtr': 0.0, 'Lt1Mov': '2', 'WeekTrnMov': None, 'YearTrnMov': 2017.0, 'MaxTtSpd': '79', 'MinSpd': '50', 'MaxSpd': '79', 'MainTrk': '2', 'SidingTrk': '0', 'YardTrk': '0', 'TransitTrk': '0', 'IndustryTrk': '0', 'OthrTrk': '1', 'OthrTrkDes': 'IND', 'SpselIDs': '11', 'Sgnleqp': '1', 'EMonitorDvce': '2', 'HealthMonitor': '2', 'NoSigns': '1', 'XBuck': '0', 'StopStd': '0', 'YieldStd': '0', 'AdvWarn': '1,2,3,4,11,12', 'AdvW10_1': '2', 'AdvW10_2': '1', 'AdvW10_3': '0', 'AdvW10_4': '0', 'AdvW10_11': '0', 'AdvW10_12': '0', 'Low_Grnd': '2', 'Low_GrndSigns': '0', 'PaveMrkIDs': '1', 'Channel': '5', 'Exempt': '2', 'EnsSign': '1', 'OthSgn': '1', 'OthSgn1': '2', 'OthDes1': '45', 'OthSgn2': '0', 'OthDes2': '0', 'OthSgn3': '0', 'OthDes3': '', 'PrvxSign': ' ', 'Led': '', 'Gates': '2', 'GatePed': '0', 'GateConf': '1', 'FlashOv': '4', 'FlashNov': '0', 'CFlashType': '1', 'FlashPost': '0', 'FlashPostType': '0', 'Bkl_FlashPost': '2', 'Sdl_FlashPost': '2', 'FlashPai': '12', 'AwdIDate': '082009', 'AwhornChk': '2', 'AwhornlDate': '', 'HwyTrafSignl': '2', 'Wigwags': '0', 'Bells': '1', 'SpecPro': '0', 'FlashOth': '0', 'FlashOthDes': '', 'HwynrSig': '1', 'Intrprmp': '2', 'PrempType': '2', 'HwtrfPsig': '2', 'HwtrfPsigsdis': '0', 'HwtrfPsiglndis': '0', 'MonitorDev': '0', 'WdCode': '8', 'TraficLn': '5', 'TraflnType': '2', 'HwyPved': '1', 'Downst': '2', 'Illumina': '2', 'XSurfDate': None, 'XSurfWidth': '27', 'XSurfLength': '64', 'XSurfaceIDs': '13', 'XSurOthr': '', 'HwyNear': '1', 'HwynDist': '0', 'XAngle': '3', 'ComPower': '1', 'HwySys': '2', 'HwyClassCD': '1', 'HwyClassrdtpID': '13', 'StHwy1': '1', 'HwySpeed': '40', 'HwySpeedps': '1', 'LrsRouteid': None, 'LrsMilePost': None, 'Aadt': '19800', 'AadtYear': '2011', 'PctTruk': '5', 'SchlBusChk': '2', 'SchlBsCnt': '0', 'HazmtVeh': None, 'EmrgncySrvc': '1'}]}
    725401E 2 1 0 8 11
    749707C 2  of ?
    749707C 2 1 0 8 0
    050381C 3  of ?
    
    
## Appendix B Resources
Crossing Field/Metadata Definitions: https://safetydata.fra.dot.gov/MasterWebService/SecureAPI/Support/Datasets?ModelType=Crossings 
Crossing Accident Field Definitions:    https://safetydata.fra.dot.gov/OfficeofSafety/publicsite/downloadFStructure.aspx
Activation Failure database
https://safetydata.fra.dot.gov/OfficeofSafety/publicsite/affp/AfBrowse.aspx
Illinois High-Speed Rail
Hellman, Adrian & Ngamdung, Tashi. (2010). Illinois High-Speed Rail Four-Quadrant Gate Reliability Assessment.
