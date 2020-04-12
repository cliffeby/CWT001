# CWT001
Constant Warning Times


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
tokenParam = "505d24081b61bdf9c89c38f4c2da4506"  # FRA supplied token.  Needs to be hardcoded below
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
                token = {'$filter':"CrossingId eq '"+str(xing[0])+"'",'token':"505d24081b61bdf9c89c38f4c2da4506"}
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

    725401E 1  of ?
    {'odata.metadata': 'https://safetydata.fra.dot.gov/MasterWebService/PublicApi/frads/v1/odata/gcis/$metadata#Crossings', 'value': [{'ReportBaseId': 4519050, 'MultipleFormsFiled': 0, 'ReportingAgencyId': 1622, 'ReportingAgencyTypeId': 1, 'RevisionDate': '2019-11-11T00:00:00', 'PostmarkDate': None, 'ReasonId': 14, 'CrossingId': '725401E', 'CrossingIdSuffix': 'NS', 'ReportStatus': 'Published', 'ReportType': 'Major', 'CreatedDate': None, 'CreatedBy': 'ENDCCrossings@nscorp.com', 'LastUpdatedDate': '0001-01-01T00:00:00', 'Railroad': 'NS', 'StateCD': '01', 'CntyCD': '01073', 'Nearest': '0', 'CityCD': '010730330', 'Street': '14TH ST', 'BlockNumb': '', 'Highway': 'SR 150', 'SepInd': '2', 'SepRr1': '', 'SepRr2': '', 'SepRr3': '', 'SepRr4': '', 'MultFrmsFiled': '2', 'SameInd': '1', 'SameRr1': 'ATK', 'SameRr2': ' ', 'SameRr3': ' ', 'SameRr4': ' ', 'RrID': '', 'Ttstn': '472600', 'TtstnNam': 'BIRMINGHAM', 'RrMain': '-1', 'XingOwnr': '-1', 'TypeXing': '3', 'XPurpose': '1', 'PosXing': '1', 'OpenPub': '', 'TypeTrnSrvcIDs': '11,12', 'DevelTypID': '13', 'XingAdj': '2', 'XngAdjNo': '', 'Whistban': '0', 'WhistDate': None, 'HscoRrid': '-1', 'Latitude': '33.3953043', 'Longitude': '-86.9548082', 'LLsource': '1', 'RrNarr1': '', 'RrNarr2': '', 'RrNarr3': '', 'RrNarr4': '', 'StNarr1': '', 'StNarr2': '', 'StNarr3': 'State Phone# updated - date updated: 2020-02-24', 'StNarr4': 'AGS', 'PolCont': '8009464744', 'RrCont': '8009464744', 'HwyCont': '3342426234', 'OperatingRailroadCode': 'NS', 'OperatingRailroadType': 'Primary', 'RrDiv': 'ALABAMA', 'RrSubDiv': 'AGS SOUTH', 'Branch': '#N\\A', 'PrfxMilePost': 'AG', 'MilePost': '0154.900', 'SfxMilePost': ' ', 'Lt1PassMov': '2', 'PassCnt': '2', 'DayThru': '20', 'NghtThru': 9.0, 'TotalSwt': 8.0, 'TotalLtr': 0.0, 'Lt1Mov': '2', 'WeekTrnMov': None, 'YearTrnMov': 2017.0, 'MaxTtSpd': '79', 'MinSpd': '50', 'MaxSpd': '79', 'MainTrk': '2', 'SidingTrk': '0', 'YardTrk': '0', 'TransitTrk': '0', 'IndustryTrk': '0', 'OthrTrk': '1', 'OthrTrkDes': 'IND', 'SpselIDs': '11', 'Sgnleqp': '1', 'EMonitorDvce': '2', 'HealthMonitor': '2', 'NoSigns': '1', 'XBuck': '0', 'StopStd': '0', 'YieldStd': '0', 'AdvWarn': '1,2,3,4,11,12', 'AdvW10_1': '2', 'AdvW10_2': '1', 'AdvW10_3': '0', 'AdvW10_4': '0', 'AdvW10_11': '0', 'AdvW10_12': '0', 'Low_Grnd': '2', 'Low_GrndSigns': '0', 'PaveMrkIDs': '1', 'Channel': '5', 'Exempt': '2', 'EnsSign': '1', 'OthSgn': '1', 'OthSgn1': '2', 'OthDes1': '45', 'OthSgn2': '0', 'OthDes2': '0', 'OthSgn3': '0', 'OthDes3': '', 'PrvxSign': ' ', 'Led': '', 'Gates': '2', 'GatePed': '0', 'GateConf': '1', 'FlashOv': '4', 'FlashNov': '0', 'CFlashType': '1', 'FlashPost': '0', 'FlashPostType': '0', 'Bkl_FlashPost': '2', 'Sdl_FlashPost': '2', 'FlashPai': '12', 'AwdIDate': '082009', 'AwhornChk': '2', 'AwhornlDate': '', 'HwyTrafSignl': '2', 'Wigwags': '0', 'Bells': '1', 'SpecPro': '0', 'FlashOth': '0', 'FlashOthDes': '', 'HwynrSig': '1', 'Intrprmp': '2', 'PrempType': '2', 'HwtrfPsig': '2', 'HwtrfPsigsdis': '0', 'HwtrfPsiglndis': '0', 'MonitorDev': '0', 'WdCode': '8', 'TraficLn': '5', 'TraflnType': '2', 'HwyPved': '1', 'Downst': '2', 'Illumina': '2', 'XSurfDate': None, 'XSurfWidth': '27', 'XSurfLength': '64', 'XSurfaceIDs': '13', 'XSurOthr': '', 'HwyNear': '1', 'HwynDist': '0', 'XAngle': '3', 'ComPower': '1', 'HwySys': '2', 'HwyClassCD': '1', 'HwyClassrdtpID': '13', 'StHwy1': '1', 'HwySpeed': '40', 'HwySpeedps': '1', 'LrsRouteid': None, 'LrsMilePost': None, 'Aadt': '19800', 'AadtYear': '2011', 'PctTruk': '5', 'SchlBusChk': '2', 'SchlBsCnt': '0', 'HazmtVeh': None, 'EmrgncySrvc': '1'}]}
    725401E 2 1 0 8 11
    749707C 2  of ?
    749707C 2 1 0 8 0
    050381C 3  of ?
