#testing

#! //anaconda/bin/python

import urllib
import json
import datetime
from datetime import date, timedelta
import numpy as np
import pandas as pd
import socket

socket.setdefaulttimeout(18000)


#First, get the possible county numbers from REST
county_url = 'https://wccarest.wicourts.gov:443/api/v1/counties'

open_county = urllib.urlopen(county_url)
county_data = open_county.read()

#Iterate through the county JSON to get a list of county numbers
nums = list(range(0,75))
counties=[]

for i in nums:
countyx = json.loads(str(county_data))[i]
countyno = countyx[u'countyNo']
counties.append(countyno)

counties.sort()


#Make a list of dates
def daterange(start_date, end_date):
for n in range(int ((end_date - start_date).days)):
yield start_date + timedelta(n)

start_date = date(2004, 1, 1)
end_date = date(2016, 3, 31)

all_dates = list(daterange(start_date,end_date))

#Iterate through the county/day combos to get a list of case nos and counties

baseurl = 'https://wccarest.wicourts.gov:443/api/v1/cases?'
#Search takes the format countyNo=1&&&&&filingDate=2004-01-01&&&&



for c in range(len(counties)):
#Empty list to put the data into
listcasedicts = []
County = str(counties[c])
for d in range(len(all_dates)):
Date = all_dates[d].strftime("%Y-%m-%d")
urlx = baseurl + 'countyNo=' + County + '&&&&&filingDate=' + Date + '&&&&'
try:
openx = urllib.urlopen(urlx)
except IOError:
print 'IOError Cannot Open ' + urlx	
continue
datax = openx.read()
jsall = json.loads(str(datax))
if len(jsall) == 0:
print 'No Cases in County ' + County + ' on ' + Date
continue
else:
print 'Reading ' + str(len(jsall)) + ' Cases in County ' + County + ' on ' + Date
for j in range(len(jsall)):
try:
jsx = json.loads(str(datax))[j]
except:
print 'ERROR in json load ' + urlx
caseurl = jsx[u'meta'][u'href']
try:
opencase = urllib.urlopen(caseurl)
except IOError:
print 'IOError Cannot Open ' + caseurl	
continue	
datacase = opencase.read()
jscase = json.loads(str(datacase))
#create empty dictionary to put case data - IMPORTANT - empties out after each url
casedict = {}
casedict['Caseurl'] = str(caseurl)
#"Flatten" JSON - Pull data into 1-level Python Dict per Case
try:
casedict['CaseNo'] = str(jscase[u'caseNo'])
except:
pass
try:
casedict['County'] = str(jscase[u'county'][u'countyNo'])
except:
pass
try:
casedict['Status'] = str(jscase[u'status'][u'statusCode'])
except:
pass
try:
casedict['Type'] = str(jscase[u'type'][u'caseType'])
except:
pass
try:
if len(jscase[u'parties']) > 0:
for party in range(len(jscase[u'parties'])):
try:
casedict['PartyDOB' + str(party)] = str(jscase[u'parties'][party][u'dob'])
except KeyError:
pass
try:
casedict['PartySex' + str(party)] = str(jscase[u'parties'][party][u'sex'])
except KeyError:
pass
try:
casedict['PartyRace' + str(party)] = str(jscase[u'parties'][party][u'race'][u'raceCode'])
except KeyError:
pass
try:
casedict['PartyType' + str(party)] = str(jscase[u'parties'][party][u'type'][u'partyType'])
except KeyError:
pass
try:
casedict['PartyFirstName' + str(party)] = str(jscase[u'parties'][party][u'name'][u'first'])
except KeyError:
pass
try:
casedict['PartyMiddleName' + str(party)] = str(jscase[u'parties'][party][u'name'][u'middle'])
except KeyError:
pass
try:
casedict['PartyLastName' + str(party)] = str(jscase[u'parties'][party][u'name'][u'last'])
except KeyError:
pass

except:
pass
except:
pass
try:
casedict['FilingDate'] = str(jscase[u'filingDate'])	
except:
pass
listcasedicts.append(casedict)

allcasedata = pd.DataFrame(listcasedicts)

# Create a Pandas Excel writer using XlsxWriter as the engine.
writer = pd.ExcelWriter('courtdata' + County + '.xlsx', engine='xlsxwriter')

# Convert the dataframe to an XlsxWriter Excel object.
allcasedata.to_excel(writer, sheet_name='Sheet1')

# Close the Pandas Excel writer and output the Excel file.
writer.save()
