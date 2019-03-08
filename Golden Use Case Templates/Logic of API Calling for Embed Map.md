
# Logic of API Calling for Embed Map
Many service providers and large enterprises benefit from their powerful web portals which offer seamless integrations with various well-established network management systems. NetBrain Embedded Map is a technology offered by NetBrain, which integrates a highly dynamic and easily deployable network map into client web portal by providing them with a set of pre-defined iframes and API endpoints.

User can choose either of the following ways to consume NetBrain Embedded Map technology:<br>
>▪ View site maps, device group maps, and maps in the Public folder.<br>
▪ View path maps by calculating paths.

**Workflow**
><br>1. Modify the web.config file of your NetBrain Web Server to allow the iframe to render NetBrain web page.<br>2. Reference the Web Server script library in the portal.<br>3. Initialize NetBrain instance.<br>4. Construct the drop-down menu for Tenant.<br>5. Construct the drop-down menu for Domain.<br>6. Construct the drop-down menu for Site/Device Group/Public Map File.<br>7. Browse and open a specific map.<br>8. Construct the text box and drop-down menu for path options.<br>9. Calculate a path and view the path result.

## Web server configuration and UI preparation 
Please check the detail information in [reference](https://github.com/Gongdai/REST_API_with_Markdown/blob/master/Golden%20Use%20Case%20Templates/v3_NetBrain_Embedded_Map_Quick_Start_Guide.pdf)

## Sequence of REST API callings
a. Get Authentication Token and Specify Working Domain.
>1. Login <br>
>2. Get all accessible tenants <br>
>3. Get all accessible domains of a tenant

b. Get Specified Device Information and Calculate Path. 
>4. Get child sites of a specific site <br>
>5. Get device group list 
>6. Calculate a Path <br>
>7. Get the gateway information of a device <br>
>8. Get path calculation status <br>
>9. Get path calculation result 
>10. Get file list

c. Finish All Callings and Logout Netbrain 
>11. Stop a path 
>12. Logout


```python
# import python modules 
import requests
import time
import urllib3
import pprint
#urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
import json

# input all fixed global variables. 
nb_url = "http://192.168.28.79"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'} 
TenantName = "Initial Tenant"
DomainName = "GE Test"
username = "gdluserTest"
password = "123456"
groupName = "testG1"
sitePath = "My Network"
folderId = ""
fileTypes = [0, 11 ,21]
```


```python
# Step 1, prepare for get path feature.

body = {
    "username" : username,      
    "password" : password  
}

# provide all required REST API URLs.
login_URL = nb_url + "/ServicesAPI/API/V1/Session"
Accessible_tenants_url = nb_url + "/ServicesAPI/API/V1/CMDB/Tenants"
Accessible_domains_url = nb_url + "/ServicesAPI/API/V1/CMDB/Domains"
Specify_a_working_domain_url = nb_url + "/ServicesAPI/API/V1/Session/CurrentDomain"


# login to get authentication tokan.
def login(login_URL, body, headers):
    try:
        # Do the HTTP request
        response = requests.post(login_URL, headers=headers, data = json.dumps(body), verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js["token"])
        else:
            return ("Get token failed! - " + str(response.text))
    except Exception as e:
        return (str(e))

# return all accessible tenants of this Netbrain user account.
def get_all_accessible_tenants(Accessible_tenants_url, token, headers):
    headers["Token"] = token
    try:
        # Do the HTTP request
        response = requests.get(Accessible_tenants_url, headers=headers, verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            result = response.json()
            tenants = result["tenants"]
            tenant =  [x for x in tenants if x["tenantName"] == TenantName] # Name of the tenant which we are going to work inside
            #tj = tenant.json()
            tenantId = tenant[0]["tenantId"]
            #ten_j = json.dumps(tenant)
            return tenantId
        else:
            return ("Get tenants failed! - " + str(response.text))
    except Exception as e: return (e)

# return all accessible domains in corresponding tenant.
def get_all_accessible_domains(tenantId, token, headers):
    headers["Token"] = token
    data = {"tenantId": tenantId}
    try:
        # Do the HTTP request
        response = requests.get(Accessible_domains_url, params=data, headers=headers, verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            result = response.json()
            domains = result["domains"]
            domain =  [x for x in domains if x["domainName"] == DomainName]# Name of the domain which we are going to work inside
            domainId = domain[0]["domainId"]
            return domainId
        else:
            return ("Get domains failed! - " + str(response.text))

    except Exception as e: print (str(e))
        
# call specify_a_working_domain API
def specify_a_working_domain(tenantId, domainId, Specify_a_working_domain_url, headers, token):
    headers["Token"] = token
    body = {
        "tenantId": tenantId,
        "domainId": domainId
    }
    
    try:
        # Do the HTTP request
        response = requests.put(Specify_a_working_domain_url, data=json.dumps(body), headers=headers, verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            result = response.json()
            return ("Working Domain Specified Successfully, with domainId: " + domainId)
            
        elif response.status_code != 200:
            return ("Login failed! - " + str(response.text))

    except Exception as e: print (str(e))
```


```python
token = login(login_URL, body, headers)
tenantId = get_all_accessible_tenants(Accessible_tenants_url, token, headers)
domainId = get_all_accessible_domains(tenantId, token, headers)
res =  specify_a_working_domain(tenantId, domainId, Specify_a_working_domain_url, headers, token)

print("Token: "+token) # print out the authentication token.
print("Tenant ID: "+tenantId) # print out the specified tenant id.
print("Domain ID: "+domainId) # Print out the specified domain Id.
print (res) # confirm user get in the correct domain

```
API Response:

    Token: 6c93fb12-afd0-41de-b557-5491c4fc0a1d
    Tenant ID: fb24f3f0-81a7-1929-4b8f-99106c23fa5b
    Domain ID: 0201adc4-ae96-46f0-ae3d-01cdba9e41d6
    Working Domain Specified Successfully, with domainId: 0201adc4-ae96-46f0-ae3d-01cdba9e41d6
    


```python
# Step 2, calling all path API and save result to specified folder.

# All required REST API URLs
Get_child_sites_of_a_specific_site_url = nb_url + "/ServicesAPI/API/V1/CMDB/Sites/ChildSites"
Get_device_group_list_url = nb_url + "/ServicesAPI/API/V1/CMDB/DeviceGroups"
Get_group_devices_url = nb_url + "/ServicesAPI/API/V1/CMDB/Devices/GroupDevices/" + str(groupName)
Get_the_gateway_information_of_a_device_url = nb_url + "/ServicesAPI/API/V1/CMDB/Path/Gateways"
Calculate_a_Path_url = nb_url + "/ServicesAPI/API/V1/CMDB/Path/Calculation"
Get_file_list_url = nb_url + "/ServicesAPI/API/V1/CMDB/Files/"

data = {
           "sitePath" : sitePath
        } 
# Calling get child list API.
def Get_child_sites_of_a_specific_site(Get_child_sites_of_a_specific_site_url, data, headers, token):
    headers["Token"] = token
    try:
        response = requests.get(Get_child_sites_of_a_specific_site_url, params = data, headers = headers, verify = False)
        #response = requests.delete(full_url, headers = headers, verify = False)
        if response.status_code == 200:
            result = response.json()["sites"]
            return (result)
        else:
            return ("Site Created Failed! - " + str(response.text))

    except Exception as e:
        return (str(e)) 

# Calling get device group list API.
def Get_device_group_list(Get_device_group_list_url, headers, token):
    headers["Token"] = token
    try:
        response = requests.get(Get_device_group_list_url, headers = headers, verify = False)
        #response = requests.delete(full_url, headers = headers, verify = False)
        if response.status_code == 200:
            result = response.json()["deviceGroups"]
            return (result)
        else:
            return ("Site Created Failed! - " + str(response.text))

    except Exception as e:
        return (str(e)) 
    
# Calling get group devices API
def Get_group_devices(Get_group_devices_url, headers, token):
    headers["Token"] = token
    try:
        response = requests.get(Get_group_devices_url, headers=headers, verify=False)
        if response.status_code == 200:
            devices = response.json()["devices"]
            #result = devices[]
            return (devices)
        else:
            return ("Get Group Devices Failed! - " + str(response.text))
    except Exception as e:
        return (str(e))
    
def helper(devicesList):
    ips = []

    for i in devicesList:
        ip = i["hostname"]
        if ip != "":
            ips.append(ip)

    return(ips)
    

# Calling calculate a path API.
def Get_the_gateway_information_of_a_device(Get_the_gateway_information_of_a_device_url, ipOrHost, headers, token):
    headers["Token"] = token
    data = {"ipOrHost":ipOrHost}
    try:
        response = requests.get(Get_the_gateway_information_of_a_device_url, params = data, headers = headers, verify = False)
        if response.status_code == 200:
            result = response.json()["gatewayList"]
            return (result)
        else:
            return ("Create module attribute failed! - " + str(response.text))

    except Exception as e:
        return (str(e)) 
    
# Calling get file list API
def Get_file_list_url(Get_file_list_url, folderId, fileTypes, headers, token):
    headers["Token"] = token
    body = {
        "folderId": folderId, 
        "fileTypes": fileTypes
    }
    try:
        response = requests.get(Get_file_list_url, data = json.dumps(body), headers = headers, verify = False)
        if response.status_code == 200:
            result = response.json()
            return (result)
        else:
            return ("Retrieve folder list failed! - " + str(response.text))

    except Exception as e:
        return (str(e)) 
```


```python
siteInformation = Get_child_sites_of_a_specific_site(Get_child_sites_of_a_specific_site_url, data, headers, token)
siteInformation
```


API Response:



    [{'siteId': '37965f93-377c-46b9-852c-193870bb5933',
      'sitePath': 'My Network/Americas',
      'isContainer': True,
      'children': ['e109a735-752a-41bf-ae22-ddf9c6a2a09f'],
      'siteType': 1},
     {'siteId': 'e109a735-752a-41bf-ae22-ddf9c6a2a09f',
      'sitePath': 'My Network/Americas/Argentina',
      'isContainer': True,
      'children': ['bd85e5ba-453c-4a91-9204-9b0a75a923d9'],
      'siteType': 1},
     {'siteId': 'bd85e5ba-453c-4a91-9204-9b0a75a923d9',
      'sitePath': 'My Network/Americas/Argentina/Provincia de Buenos Aires',
      'isContainer': True,
      'children': ['ae1a0cd5-c9cb-4821-a40e-f3441ad71a23'],
      'siteType': 1},
     {'siteId': 'ae1a0cd5-c9cb-4821-a40e-f3441ad71a23',
      'sitePath': 'My Network/Americas/Argentina/Provincia de Buenos Aires/Benavidez',
      'isContainer': True,
      'children': ['e762eaa7-507f-4c02-9d40-c616f6d64702'],
      'siteType': 1},
     {'siteId': 'e762eaa7-507f-4c02-9d40-c616f6d64702',
      'sitePath': 'My Network/Americas/Argentina/Provincia de Buenos Aires/Benavidez/AM-ARG-BA-BEN-1621-KM375RAM1618',
      'isContainer': False,
      'siteType': 2},
     {'siteId': 'cc91d598-b404-4053-944f-3db7b2537b46',
      'sitePath': 'My Network/testSite1',
      'isContainer': True,
      'children': ['6c1f12b6-90c6-4e02-aa6f-f1f7128c2f5b'],
      'siteType': 1},
     {'siteId': '6c1f12b6-90c6-4e02-aa6f-f1f7128c2f5b',
      'sitePath': 'My Network/testSite1/testSiteLeaf',
      'isContainer': False,
      'siteType': 2},
     {'siteId': '732e8ab6-6b69-417d-ad03-2cc447100166',
      'sitePath': 'My Network',
      'isContainer': True,
      'children': ['37965f93-377c-46b9-852c-193870bb5933',
       'cc91d598-b404-4053-944f-3db7b2537b46'],
      'siteType': 0}]




```python
groupList = Get_device_group_list(Get_device_group_list_url, headers, token)
groupList
```


API Response:



    [{'id': 'a2576703-08f6-4a1e-bd38-c03f36e962d1',
      'name': '#BGP 12345',
      'type': 2},
     {'id': '76c464a9-7fee-4ce6-ba05-c9962f206ed9',
      'name': '#BGP 34567',
      'type': 2},
     {'id': '95efd71c-635b-44a0-ac6c-76d99f0f63fb',
      'name': '#BGP 45678',
      'type': 2},
     {'id': 'fff9a03b-052c-4ece-b5db-8fb38ac4eada',
      'name': '#BGP 65111',
      'type': 2},
     {'id': 'fa2273d4-6600-4870-8286-a258a3bb2b9f',
      'name': '#BGP 65112',
      'type': 2},
     {'id': '3174bdc3-93a3-4fcb-b503-5d266c26bf3f',
      'name': '#BGP 65222',
      'type': 2},
     {'id': '8395ef0c-2b99-4c83-84e2-a2e0b9e9c22f', 'name': '#EIGRP', 'type': 2},
     {'id': 'd22885ac-e930-4ada-9c4c-f59b1611c3fa',
      'name': '#EIGRP 34567',
      'type': 2},
     {'id': '649bc165-fe2d-485d-96a0-c4fa9ecf6282',
      'name': '#EIGRP 45678',
      'type': 2},
     {'id': '40e37021-f94d-4a95-9aa2-216875d6b013',
      'name': '#OSPF 12345',
      'type': 2},
     {'id': '08722b14-d70c-4293-b451-721b2c2a8e12', 'name': 'testG1', 'type': 1}]




```python
devicesList = Get_group_devices(Get_group_devices_url, headers, token)
devicesList
```

API Response:




    [{'id': '4b814621-05d0-4dd2-98e0-59d79b1ec410',
      'mgmtIP': '123.11.11.11',
      'hostname': 'R11'},
     {'id': '5074459d-1435-4f65-a323-94c7dffcd3a9',
      'mgmtIP': '10.1.13.1',
      'hostname': 'R13'},
     {'id': '575fd214-acdf-427c-914a-2acd2aedb6af',
      'mgmtIP': '123.12.12.12',
      'hostname': 'R12'},
     {'id': '64a80717-49a3-4f61-829b-926d1dabde79',
      'mgmtIP': '123.10.1.2',
      'hostname': 'R1'},
     {'id': '98c00148-016f-43fc-9c1d-926fc728551e',
      'mgmtIP': '123.20.1.1',
      'hostname': 'R15'},
     {'id': 'bbfbc73f-3425-4286-9402-fda3bc4e7661',
      'mgmtIP': '10.1.14.1',
      'hostname': 'R14'},
     {'id': 'f51f6e8e-d4ef-47af-9139-74a18691c052',
      'mgmtIP': '123.20.1.2',
      'hostname': 'R16'},
     {'id': 'f7eec066-c9b0-4e08-8ada-aad8f5e35a16',
      'mgmtIP': '123.10.10.10',
      'hostname': 'R10'}]




```python
IPList =  helper(devicesList)
IPList
```

API Response:




    ['R11', 'R13', 'R12', 'R1', 'R15', 'R14', 'R16', 'R10']




```python
results = []
for i in IPList:
    result = Get_the_gateway_information_of_a_device(Get_the_gateway_information_of_a_device_url, i, headers, token)
    results.append(result)
results
```

API Response:




    [[{'ip': '123.11.11.11', 'devName': 'R11', 'intfName': 'Loopback0'}],
     [{'ip': '10.1.13.1', 'devName': 'R13', 'intfName': 'Ethernet0/0'}],
     [{'ip': '123.12.12.12', 'devName': 'R12', 'intfName': 'Loopback0'}],
     [{'ip': '123.10.1.2', 'devName': 'R1', 'intfName': 'Ethernet0/2'}],
     [{'ip': '123.20.1.1', 'devName': 'R15', 'intfName': 'Ethernet0/1'}],
     [{'ip': '10.1.14.1', 'devName': 'R14', 'intfName': 'Ethernet0/0'}],
     [{'ip': '123.20.1.2', 'devName': 'R16', 'intfName': 'Ethernet0/1'}],
     [{'ip': '123.10.10.10', 'devName': 'R10', 'intfName': 'Loopback0'}]]




```python
# Calling Calculate a path API
from random import randint

def Calculate_a_Path(Calculate_a_Path_url, body, headers, token):
    headers["Token"] = token
    
    try:
        response = requests.post(Calculate_a_Path_url, data = json.dumps(body), headers = headers, verify = False)
        if response.status_code == 200:
            result = response.json()
            return (result["taskID"])
        else:
            return ("Create module attribute failed! - " + str(response.text))

    except Exception as e:
        return (str(e)) 
    
ress = []    
for i in range(len(results)):
    gw = results[i][0]

    j = randint(0, len(results))
    gw1 = results[j][0]

    if i != j:
        sourceIP = gw["ip"]
        sourcePort = 0
        sourceGwIP = gw["ip"]
        sourceGwDev = gw["devName"]
        sourceGwIntf = gw["intfName"]
        destIP = gw1["ip"]
        destPort = 0
        pathAnalysisSet = 2
        protocol = 4
        isLive = False

        body = {
                        "sourceIP" : sourceIP,                # IP address of the source device.
                        "sourcePort" : sourcePort,
                        "sourceGwDev" : sourceGwDev,          # Hostname of the gateway device.
                        "sourceGwIP" : sourceGwIP,            # Ip address of the gateway device.
                        "sourceGwIntf" : sourceGwIntf,        # Name of the gateway interface.
                        "destIP" : destIP,                    # IP address of the destination device.
                        "destPort" : destPort,
                        "pathAnalysisSet" : pathAnalysisSet,  # 1:L3 Path; 2:L2 Path; 3:L3 Active Path
                        "protocol" : protocol,                # Specify the application protocol, check online help, such as 4 for IPv4.
                        "isLive" : isLive                     # False: Current Baseline; True: Live access
            } 

        res = Calculate_a_Path(Calculate_a_Path_url, body, headers, token)
        ress.append(res)
ress
```

API Response:




    ['2252233e-7e6f-48b9-aa3e-a35b41f8ef14',
     'a33bcdad-d31d-4bbb-98cc-4201574d36c5',
     '3c816dee-acbe-4722-bca4-3ffbd8c4ea79',
     '91f078b9-941e-4ac9-bd41-cd6f0bb6a8b8',
     '1d665bea-0b22-481a-a362-7b98eb529ff0',
     'e3e9d59b-fd0a-4fb0-8d9d-ee53c8391349',
     'b312ec85-e75e-42af-bd72-f47ff7da6bed',
     'f21d2327-394c-4b81-ae32-77231e75cd2f']




```python
# Calling get path calculation result API
import time

def Get_path_calculation_result(Get_path_calculation_result_url, headers, token):
    headers["Token"] = token
    try:
        response = requests.get(Get_path_calculation_result_url, headers = headers, verify = False)
        if response.status_code == 200:
            result = response.json()
            return (result.json())
        else:
            return (response.json())

    except Exception as e:
        return (str(e)) 

Final = []
for i in range(len(ress)):
    if len(ress[i]) <= 36: # length of "taskId"
        time.sleep(3)   # Delays for 5 seconds. You can also use a float value.
        Get_path_calculation_result_url = nb_url + "/ServicesAPI/API/V1/CMDB/Path/Calculation/" + str(ress[i]) + "/Result"
        final = Get_path_calculation_result(Get_path_calculation_result_url, headers, token)
        d = json.dumps(final)
        j = json.loads(d)
        Final.append(final)
    
Final
```

API Response:




    [{'hopList': [{'hopId': '98d641bc-8f09-4581-b7c9-74f9418a5c19',
        'srcDeviceName': 'R11',
        'inboundInterface': 'Ethernet0/1',
        'mediaName': '',
        'dstDeviceName': 'SW3',
        'outboundInterface': 'Ethernet0/3',
        'nextHopIdList': ['ed26ae18-fdcd-4bb1-a761-263fcb13cb10']},
       {'hopId': 'ed26ae18-fdcd-4bb1-a761-263fcb13cb10',
        'srcDeviceName': 'SW3',
        'inboundInterface': 'Ethernet2/0',
        'mediaName': '',
        'dstDeviceName': 'SW4',
        'outboundInterface': 'Ethernet2/0',
        'nextHopIdList': ['9eb0f4b2-4df6-4589-bd30-e524b65f8caa']},
       {'hopId': '9eb0f4b2-4df6-4589-bd30-e524b65f8caa',
        'srcDeviceName': 'SW4',
        'inboundInterface': 'Ethernet2/0',
        'mediaName': '',
        'dstDeviceName': 'SW3',
        'outboundInterface': 'Ethernet2/0',
        'nextHopIdList': ['aeb5645c-0e24-48c7-b249-c503158acd7b']},
       {'hopId': 'aeb5645c-0e24-48c7-b249-c503158acd7b',
        'srcDeviceName': 'SW3',
        'inboundInterface': 'Ethernet0/1',
        'mediaName': '',
        'dstDeviceName': 'R9',
        'outboundInterface': 'Ethernet0/1',
        'nextHopIdList': ['2cbf5d75-f978-4a1b-ba88-0a17646dc514']},
       {'hopId': '2cbf5d75-f978-4a1b-ba88-0a17646dc514',
        'srcDeviceName': 'R9',
        'inboundInterface': 'Ethernet1/0',
        'mediaName': '',
        'dstDeviceName': '33.34.4.1',
        'outboundInterface': 'Ethernet0',
        'nextHopIdList': []}],
      'statusCode': 794008,
      'statusDescription': "Task '2252233e-7e6f-48b9-aa3e-a35b41f8ef14' is failed"},
     {'hopList': [],
      'statusCode': 794008,
      'statusDescription': "Task 'a33bcdad-d31d-4bbb-98cc-4201574d36c5' is failed"},
     {'hopList': [{'hopId': 'eaaec884-ec70-41ff-8e8d-d64cf9159a97',
        'srcDeviceName': 'R12',
        'inboundInterface': 'Ethernet0/1',
        'mediaName': '',
        'dstDeviceName': '201.1.12.1',
        'outboundInterface': 'Ethernet0',
        'nextHopIdList': []}],
      'statusCode': 794008,
      'statusDescription': "Task '3c816dee-acbe-4722-bca4-3ffbd8c4ea79' is failed"},
     {'hopList': [{'hopId': '3e9e6020-9274-4bfa-8247-a710ea9c196e',
        'srcDeviceName': 'R1',
        'inboundInterface': 'Ethernet0/2',
        'mediaName': '123.10.1.0/30',
        'dstDeviceName': 'R4',
        'outboundInterface': 'Ethernet0/1',
        'nextHopIdList': ['40d8f300-9c5b-4714-9aad-ea563a59a196']},
       {'hopId': '40d8f300-9c5b-4714-9aad-ea563a59a196',
        'srcDeviceName': 'R4',
        'inboundInterface': 'Ethernet0/0',
        'mediaName': '123.10.1.16/30',
        'dstDeviceName': 'R2',
        'outboundInterface': 'Ethernet0/2',
        'nextHopIdList': ['f7d763de-e0c5-4008-9f0e-c7529153b461']},
       {'hopId': 'f7d763de-e0c5-4008-9f0e-c7529153b461',
        'srcDeviceName': 'R2',
        'inboundInterface': 'Ethernet0/3.100',
        'mediaName': '10.120.100.0/30',
        'dstDeviceName': 'R20',
        'outboundInterface': 'Ethernet2/0.100',
        'nextHopIdList': ['b2f061bb-7ab2-4adb-a01a-8e2b47a011a6']},
       {'hopId': 'b2f061bb-7ab2-4adb-a01a-8e2b47a011a6',
        'srcDeviceName': 'R20',
        'inboundInterface': 'Ethernet2/0',
        'mediaName': '',
        'dstDeviceName': 'R2',
        'outboundInterface': 'Ethernet0/3',
        'nextHopIdList': ['2e91bb4e-7c3a-4c4a-bd49-31f1957a9280']},
       {'hopId': '2e91bb4e-7c3a-4c4a-bd49-31f1957a9280',
        'srcDeviceName': 'R2',
        'inboundInterface': 'Ethernet0/2',
        'mediaName': '123.10.1.16/30',
        'dstDeviceName': 'R4',
        'outboundInterface': 'Ethernet0/0',
        'nextHopIdList': ['40d8f300-9c5b-4714-9aad-ea563a59a196']}],
      'statusCode': 794008,
      'statusDescription': "Task '91f078b9-941e-4ac9-bd41-cd6f0bb6a8b8' is failed"},
     {'hopList': [{'hopId': '3b08db2e-f491-4e0e-80df-6c9a82a4a8c0',
        'srcDeviceName': 'R15',
        'inboundInterface': 'Ethernet0/0',
        'mediaName': '',
        'dstDeviceName': '103.2.45.1',
        'outboundInterface': 'Ethernet0',
        'nextHopIdList': []}],
      'statusCode': 794008,
      'statusDescription': "Task '1d665bea-0b22-481a-a362-7b98eb529ff0' is failed"},
     {'hopList': [{'hopId': 'b78fb225-4641-44e4-8efd-78d6791e89e4',
        'srcDeviceName': 'R14',
        'inboundInterface': 'Ethernet0/1',
        'mediaName': '',
        'dstDeviceName': '202.2.14.1',
        'outboundInterface': 'Ethernet0',
        'nextHopIdList': []}],
      'statusCode': 794008,
      'statusDescription': "Task 'e3e9d59b-fd0a-4fb0-8d9d-ee53c8391349' is failed"},
     {'hopList': [{'hopId': 'd2e307c1-064f-4e63-acdd-a87fbd66dfcb',
        'srcDeviceName': 'R16',
        'inboundInterface': 'Ethernet0/1',
        'mediaName': '',
        'dstDeviceName': 'SW5',
        'outboundInterface': 'Ethernet0/1',
        'nextHopIdList': ['5eca016a-b9cb-4688-95ab-15bc356011ac']},
       {'hopId': '5eca016a-b9cb-4688-95ab-15bc356011ac',
        'srcDeviceName': 'SW5',
        'inboundInterface': 'Ethernet0/0',
        'mediaName': '',
        'dstDeviceName': 'R15',
        'outboundInterface': 'Ethernet0/1',
        'nextHopIdList': ['adcbd14a-34f7-463a-ab6d-f935eb09c297']},
       {'hopId': 'adcbd14a-34f7-463a-ab6d-f935eb09c297',
        'srcDeviceName': 'R15',
        'inboundInterface': 'Ethernet0/0',
        'mediaName': '',
        'dstDeviceName': '103.2.45.1',
        'outboundInterface': 'Ethernet0',
        'nextHopIdList': []}],
      'statusCode': 794008,
      'statusDescription': "Task 'b312ec85-e75e-42af-bd72-f47ff7da6bed' is failed"},
     {'hopList': [{'hopId': '3d4a23ea-681d-467a-95f2-0def8accdbc7',
        'srcDeviceName': 'R10',
        'inboundInterface': 'Ethernet0/2',
        'mediaName': '',
        'dstDeviceName': 'SW4',
        'outboundInterface': 'Ethernet0/2',
        'nextHopIdList': ['3ba3019e-88f9-49f1-b46e-ef9a808492b1']},
       {'hopId': '3ba3019e-88f9-49f1-b46e-ef9a808492b1',
        'srcDeviceName': 'SW4',
        'inboundInterface': 'Ethernet2/0',
        'mediaName': '',
        'dstDeviceName': 'SW3',
        'outboundInterface': 'Ethernet2/0',
        'nextHopIdList': ['3a61fb95-28f8-49ef-a18d-8ef987434843',
         '16383782-6a0a-46a7-a152-9800b70360a3']},
       {'hopId': '3a61fb95-28f8-49ef-a18d-8ef987434843',
        'srcDeviceName': 'SW3',
        'inboundInterface': 'Ethernet2/0',
        'mediaName': '',
        'dstDeviceName': 'SW4',
        'outboundInterface': 'Ethernet2/0',
        'nextHopIdList': ['3ba3019e-88f9-49f1-b46e-ef9a808492b1']},
       {'hopId': '16383782-6a0a-46a7-a152-9800b70360a3',
        'srcDeviceName': 'SW3',
        'inboundInterface': 'Ethernet0/1',
        'mediaName': '',
        'dstDeviceName': 'R9',
        'outboundInterface': 'Ethernet0/1',
        'nextHopIdList': ['b44d45c9-8997-4710-b6d0-6fcc53942fd3']},
       {'hopId': 'b44d45c9-8997-4710-b6d0-6fcc53942fd3',
        'srcDeviceName': 'R9',
        'inboundInterface': 'Ethernet1/0',
        'mediaName': '',
        'dstDeviceName': '33.34.4.1',
        'outboundInterface': 'Ethernet0',
        'nextHopIdList': []}],
      'statusCode': 794008,
      'statusDescription': "Task 'f21d2327-394c-4b81-ae32-77231e75cd2f' is failed"}]




```python
folderInfo = Get_file_list_url(Get_file_list_url, folderId, fileTypes, headers, token)
folderInfo
```


API Response:


    "Invalid URL '<function Get_file_list_url at 0x000002A7781EF488>': No schema supplied. Perhaps you meant http://<function Get_file_list_url at 0x000002A7781EF488>?"




```python
# call logout API

Logout_url = nb_url + "/ServicesAPI/API/V1/Session"

def logout(Logout_url, token, headers):
    headers["token"] = token
    
    try:
        # Do the HTTP request
        response = requests.delete(Logout_url, headers=headers, verify=False)
        # Check for HTTP codes other than 200
        if response.status_code == 200:
            # Decode the JSON response into a dictionary and use the data
            js = response.json()
            return (js)
        else:
            return ("Session logout failed! - " + str(response.text))

    except Exception as e:
        return (str(e))

logout = logout(Logout_url, token, headers)
logout
```

API Response:


    {'statusCode': 790200, 'statusDescription': 'Success.'}



### More further details required, please check the detail information in [reference](https://github.com/Gongdai/REST_API_with_Markdown/blob/master/Golden%20Use%20Case%20Templates/v3_NetBrain_Embedded_Map_Quick_Start_Guide.pdf)


```python

```
