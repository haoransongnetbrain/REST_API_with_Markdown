
# Return the devices path results
During current use case, the final goals are present the One-Ip Table of devices in current domain and path result between two random devices in current domain.
There are totally 10 REST APIs we going to concerned to be a part of this use case, as shown in following:<br> 

**[Step 1: Use case preparation](Step-1:-Use-case-preparation)**
>> 1a. import all useful modules and create global variables<br>
>> 1b. call login API<br>
>> 1c. call get_all_accessible_tenants API<br>
>> 1d. call get_all_accessible_domains API<br>
>> 1e. call specify_a_working_domain API<br>

**[Step 2: Get the devices path results](Step-2:-Get-the-devices-path-results)**
>> 2a. call get_all_devices API<br>
>> 2b. call resolve_device_gateway API<br>
>> 2c. call calculate_path API<br>
>> 2d. call get_path_result API<br>
>> 2e. call logout API

The sequencial of provided APIs is also the sequence of our workflow steps.<br>

***Note:*** if users want to find the path results of devices, then the step sequence must be followed in Step 2. If users call these APIs with a different sequential then there would be no results or some errors would be occured. 

## Step 1: Use case preparation.
***1a. import all useful modules and create global variables.***<br>
> Note: If users try to use this code. please remember to change the "nb_url" to users' own working url.

***1b. call login API.***<br>
>Same with use case 2, we calling the login API with "username" and "password" as inputs in the first step. As response we can get the authentication token as one fixed input in following APIs calling. If users get errors when calling this API please check the API documentation on [Github_login](https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/STANDARD_formate_TEST1_LOGIN_API.ipynb).

***1c. call get_all_accessible_tenants API***
>After we got the token from previous section, we need to use this token as a key to find all tenants which we have the access authentication. During this step, the most important feature is to get the tenant id of the corresponding tenant which we decide to work inside. After running this API successfully, we will get the tenantId of the willing tenant which will be set as another input for next step API calling. If users want to get more details about this API or get errors when calling this API please check the API documentation on [Github_tenant](https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/STANDARD_formate_TEST1_Get_All_Asseccible_Tenants_API_Test1%20.ipynb) 

***1d. call get_all_accessible_domains API.***
>In this section, we are going to find all accessible domains in the corressponding tenant which we have got the tenantId from previous step. Similar with step 2, during current API call, we have to decide which domain we are going to work inside and get the domainId at meanwhile to prepare for next API calling. If users get errors when calling this API please check the API documentation on [Github_domain](https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/STANDARD_formate_Get_all_accessible_domains_of_a_tenant%20_Test1%20.ipynb) 

***1e. call specify_a_working_domain API.***<br>
>After we running this step successfully, we directly complete the full login processes which means we totally join in Netbrain System by calling APIs(because we have record our tenantId and domainId，if users don't know the ID of corresponding tenant and domain please fully follow step 1 to step 4 in use case 1). Next step, we will start to use Netbrain functions formally. If users want to get more details about this API or get errors when calling this API please check the API documentation on [Github_domain](https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/STANDARD_formate_Specify_a_domain_to_work_on_API_Test1%20.ipynb).



```python
# import python modules 
import requests
import time
import urllib3
import pprint
#urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
import json

nb_url = "http://192.168.28.79"
headers = {'Content-Type': 'application/json', 'Accept': 'application/json'} 
TenantName = "Initial Tenant"
DomainName = "Support and Service"
username = "gdluserTest"
password = "123456"
```


```python
# call login API

body = {
    "username" : username,      
    "password" : password  
}

login_URL = nb_url + "/ServicesAPI/API/V1/Session"

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
    
token = login(login_URL, body, headers)
print(token) # print out the authentication token.
```

    cc25cbea-d244-494d-b6f9-32937bdadc69
    


```python
# call get_all_accessible_tenants API

Accessible_tenants_url = nb_url + "/ServicesAPI/API/V1/CMDB/Tenants"

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

tenantId = get_all_accessible_tenants(Accessible_tenants_url, token, headers)
print(tenantId) # print out the specified tenant id.
```

    fb24f3f0-81a7-1929-4b8f-99106c23fa5b
    


```python
# call get_all_accessible_domains API

Accessible_domains_url = nb_url + "/ServicesAPI/API/V1/CMDB/Domains"

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

domainId = get_all_accessible_domains(tenantId, token, headers)
print(domainId) # Print out the specified domain Id.
```

    850ff5e9-c639-404d-85a3-d920dbee509c
    


```python
# call specify_a_working_domain API

Specify_a_working_domain_url = nb_url + "/ServicesAPI/API/V1/Session/CurrentDomain"

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

res =  specify_a_working_domain(tenantId, domainId, Specify_a_working_domain_url, headers, token)
print (res)
```

## Step 2: Get the devices path results
***2a. call get_all_devices API***
>As we have mention at deginning, in this use case, we are going to get devices path results. So the precondition of calling path APIs is the information of devices. In current section, after we calling this Api successfully, we will get information of all devices in current domain. As following, I provide a sub-section as a helper to filt information, because after calling get all devices API we will get a json file from API response, it 's include device id, device management ip, device hostname and some other information. But we only need the managment Ip as input for next section. Thus, we provide a small funtion to filt out the "mgmIp" from json file.
If users get errors when calling this API please check the API documentation on [Github_devices](https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/Device%20API%20Design/STANDARD_formate_Get_Devices_API_Test.ipynb) 

***2b. call resolve_device_gateway API***
>After we got the pure Ips list we can start from calling resolve devices gateway API. Mention again, if users want to get path result by calling APIs then users must follow the sequencial of Step 2. In current section, we will input the Ips list we have got from previous section. After the API running successfully, we will get a gateway list with some device informations which is the required input for next section. If users want to get more details about this API or get errors when calling this API please check the API documentation on [Github_Gateway](https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/Path%20API%20Design/STANDARD_formate_Resolve_Device_Gateway_API_Test.ipynb)

***2c. call calculate_path API***
>During this section, we are going to calling the Calculate Path API and set the gateway information list as one input(other inputs are shown in following code cell). When calling this API, users must input the required parameters correctly and follow the format of each inputs examples([Github_calPath](https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/Path%20API%20Design/STANDARD_formate_Calculate_Path_API_Test.ipynb)). After calling this API successfully, we will get the corresponding taskId of gateway information which has been put in. And the taskId is the only one required input for next section. If users want to get more details about this API or get errors when calling this API please check the API documentation on [Github_calPath](https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/Path%20API%20Design/STANDARD_formate_Calculate_Path_API_Test.ipynb).

***2d. call get_path_result API***
>So far we attemp to the final functional step of this use case: to get the calculation result of the task which we have got the taskId in section 2c. After we running the following sample code successfully, we will finally get the path result in a json file. If users want to get more details about this API or get errors when calling this API please check the API documentation on [Github_pathRes](https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/Path%20API%20Design/STANDARD_formate_Get_Path_Calculation_Result_API_Test.ipynb). 

***2e. call logout API***
>After we got all informations from this case, we have to logout from the Netbrain System.
If users want to get more details about this API or get errors when calling this API please check the API documentation on [Github_logout](https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/STANDARD_formate_Logout_Test1%20.ipynb). 


```python
# call get_all_devices API

Get_all_devices_in_domain_url = nb_url + "/ServicesAPI/API/V1/CMDB/Devices"

def get_all_devices(Get_all_devices_in_domain_url, headers, token):
    try:
        response = requests.get(Get_all_devices_in_domain_url, headers=headers, verify=False)
        if response.status_code == 200:
            result = response.json()
            devices = result["devices"]
            return devices
        else:
            return("Get Devices failed! - " + str(response.text))
    except Exception as e:
        return (str(e)) 
    
devices = get_all_devices(Get_all_devices_in_domain_url, headers, token)

print("Total Devices Number: " + str(len(devices)))

devices[0:10]
```

    Total Devices Number: 93
    




    [{'id': '1266a178-b829-43c8-9c24-c34154a15d30',
      'mgmtIP': '192.168.28.204',
      'hostname': 'R20',
      'deviceTypeName': 'Cisco Router',
      'firstDiscoverTime': '0001-01-01T00:00:00',
      'lastDiscoverTime': '0001-01-01T00:00:00'},
     {'id': '497b25bd-1f8c-4bfa-80be-49ab692ce4d4',
      'mgmtIP': '123.10.1.10',
      'hostname': 'R3',
      'deviceTypeName': 'Cisco Router',
      'firstDiscoverTime': '0001-01-01T00:00:00',
      'lastDiscoverTime': '0001-01-01T00:00:00'},
     {'id': 'f190b385-676f-4579-ad6d-700122a21caf',
      'mgmtIP': '123.10.1.17',
      'hostname': 'R2',
      'deviceTypeName': 'Cisco Router',
      'firstDiscoverTime': '0001-01-01T00:00:00',
      'lastDiscoverTime': '0001-01-01T00:00:00'},
     {'id': '1d48d218-06cf-4657-af2c-39796946122b',
      'mgmtIP': '123.10.1.1',
      'hostname': 'R4',
      'deviceTypeName': 'Cisco Router',
      'firstDiscoverTime': '0001-01-01T00:00:00',
      'lastDiscoverTime': '0001-01-01T00:00:00'},
     {'id': '81229708-571a-419a-a10d-9481661718a4',
      'mgmtIP': '123.10.1.2',
      'hostname': 'R1',
      'deviceTypeName': 'Cisco Router',
      'firstDiscoverTime': '0001-01-01T00:00:00',
      'lastDiscoverTime': '0001-01-01T00:00:00'},
     {'id': '6d62e420-af59-4ee3-948d-54df60fe05ca',
      'mgmtIP': '123.10.1.6',
      'hostname': 'R5',
      'deviceTypeName': 'Cisco Router',
      'firstDiscoverTime': '0001-01-01T00:00:00',
      'lastDiscoverTime': '0001-01-01T00:00:00'},
     {'id': '5c3d72d6-d0f2-41f4-8b1e-5762dff6e55a',
      'mgmtIP': '123.10.1.22',
      'hostname': 'R6',
      'deviceTypeName': 'Cisco Router',
      'firstDiscoverTime': '0001-01-01T00:00:00',
      'lastDiscoverTime': '0001-01-01T00:00:00'},
     {'id': 'b98f107a-622e-4985-8f95-f5b541f699f3',
      'mgmtIP': '123.7.7.7',
      'hostname': 'R7',
      'deviceTypeName': 'Cisco Router',
      'firstDiscoverTime': '0001-01-01T00:00:00',
      'lastDiscoverTime': '0001-01-01T00:00:00'},
     {'id': '9b60fcfc-a405-478d-83a3-99b0ce6c9b64',
      'mgmtIP': '123.8.8.8',
      'hostname': 'R8',
      'deviceTypeName': 'Cisco Router',
      'firstDiscoverTime': '0001-01-01T00:00:00',
      'lastDiscoverTime': '0001-01-01T00:00:00'},
     {'id': '087814b3-f372-4878-bdcd-c31ba061c864',
      'mgmtIP': '123.10.10.10',
      'hostname': 'R10',
      'deviceTypeName': 'Cisco Router',
      'firstDiscoverTime': '0001-01-01T00:00:00',
      'lastDiscoverTime': '0001-01-01T00:00:00'}]




```python
# A sub-section for filt out the "mgmIp" from json file.
ips = []

for i in range(len(devices)):
    ip = devices[i]["mgmtIP"]
    if ip != "":
        ips.append(ip)
  
print(str(len(ips)))
ips[0:10]
```

    88
    




    ['192.168.28.204',
     '123.10.1.10',
     '123.10.1.17',
     '123.10.1.1',
     '123.10.1.2',
     '123.10.1.6',
     '123.10.1.22',
     '123.7.7.7',
     '123.8.8.8',
     '123.10.10.10']




```python
# call resolve_device_gateway API

Resolve_Device_Gateway_url = nb_url + "/ServicesAPI/API/V1/CMDB/Path/Gateways"
results = []
def resolve_device_gateway(Resolve_Device_Gateway_url, token, ipOrHost, headers):
    headers["Token"] = token
    data = {"ipOrHost":ipOrHost}
    try:
        response = requests.get(Resolve_Device_Gateway_url, params = data, headers = headers, verify = False)
        if response.status_code == 200:
            result = response.json()
            return (result)
        else:
            return ("Create module attribute failed! - " + str(response.text))

    except Exception as e:
        print (str(e)) 
        
for i in range(len(ips)):
    result = resolve_device_gateway(Resolve_Device_Gateway_url, token, ips[i], headers)
    #results.append(result["gatewayList"])
    #if result["gatewayList"] != ""
    res = result["gatewayList"]
    results.append(res)
    #print (res)
#result = resolve_device_gateway(Resolve_Device_Gateway_url, token, ips[0], headers)
#result["gatewayList"]
print(str(len(results)))
results[0:10]
```

    88
    




    [[{'ip': '192.168.28.204', 'devName': 'R20', 'intfName': 'Ethernet0/1'}],
     [{'ip': '123.10.1.10', 'devName': 'R3', 'intfName': 'Ethernet0/1'}],
     [{'ip': '123.10.1.17', 'devName': 'R2', 'intfName': 'Ethernet0/2'}],
     [{'ip': '123.10.1.1', 'devName': 'R4', 'intfName': 'Ethernet0/1'}],
     [{'ip': '123.10.1.2', 'devName': 'R1', 'intfName': 'Ethernet0/2'}],
     [{'ip': '123.10.1.6', 'devName': 'R5', 'intfName': 'Ethernet0/1'}],
     [{'ip': '123.10.1.22', 'devName': 'R6', 'intfName': 'Ethernet0/2'}],
     [{'ip': '123.7.7.7', 'devName': 'R7', 'intfName': 'Loopback0'}],
     [{'ip': '123.8.8.8', 'devName': 'R8', 'intfName': 'Loopback0'}],
     [{'ip': '123.10.10.10', 'devName': 'R10', 'intfName': 'Loopback0'}]]




```python
# call calculate_path API

from random import randint

Calculate_Path_url = nb_url + "/ServicesAPI/API/V1/CMDB/Path/Calculation"

def calculate_path(Calculate_Path_url, body, headers, token):
    headers["Token"] = token
    
    try:
        response = requests.post(Calculate_Path_url, data = json.dumps(body), headers = headers, verify = False)
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

        res = calculate_path(Calculate_Path_url, body, headers, token)
        ress.append(res)

print(str(len(ress)))

ress

```

    88
    




    ['72102f34-7265-41ed-aa26-c5558fa1cc60',
     '294d3f2a-4393-482c-9d20-98adbf0ece1b',
     '47ca12d1-0291-4920-9b0d-a567e58d5594',
     'bfd9b6f4-cb34-4c82-b366-a2e088dc0e1b',
     'fb6b646d-ae49-4d31-92c2-5f373f535cec',
     '220a3564-70f2-4dfb-90f0-ef6f48eb8951',
     'a570dd42-0ad6-407d-acb5-42cc09b7902b',
     'e0eb3fbf-272c-43f5-9d2c-ce7a4a98555a',
     '5088adf4-9333-4eec-aae0-74cf5fea23d4',
     '84a62b19-992e-4600-b4a8-e933e44247c4',
     'a9e758ec-efb4-4bd5-815e-7b0692f49d8c',
     '9f8794bf-f19b-4fef-b1be-423c1bbb8da4',
     'aecd8186-8e9a-493c-a19b-3d06db7fca88',
     '9056b019-8739-4c78-85a4-dde5bd765fe6',
     '60a9d225-7690-4db9-b1c4-4a83e12df3fc',
     '66c31a41-da86-44c6-81de-a9a3a14c6b58',
     '6ac4f9f9-f49b-4ba6-b0e0-78294319dc5a',
     '047a4edd-7ec4-4126-bc63-f33574f5e77a',
     'b32562d7-75d6-47ad-8898-135e311244c8',
     '7d6c3ad3-b99f-43e5-925d-34c02a1a0304',
     '8d15f6c4-1124-49ce-b40d-b163634b331b',
     '24c07b5d-e452-4c2c-b958-f0bc2af782a6',
     '859f15ff-7e66-4738-ad40-f3b6ebf7e383',
     '9ac27c4f-2431-4d33-9de8-1894e0e5bdee',
     'f7d28a65-433a-436d-a8e0-d3e1ff9d34d8',
     '2ba66022-4f7e-4ab5-94ef-11c2790b2d36',
     'fc12f1b8-4182-4eec-a8ac-96cfe2493fae',
     '8c627e88-3f0c-4b13-9bc8-73dbeac5fc6b',
     '538cb957-e77d-433d-af77-1d2bbc39721d',
     'e9c4d4b6-ebb6-4523-bb7b-780e8a18751e',
     '0b90e7f8-7afa-4d7e-af0e-e4a62bd6ade7',
     '65429420-9c74-40e9-b2b5-1609513a2d9a',
     '01023db2-cc92-4a45-9e8d-19d0fed53504',
     '46c9c790-68a3-41f7-bfbe-336f5f3a2f9f',
     'c3f85b53-e0ca-440f-a822-f57e4f8736ac',
     '19f65585-ea47-400f-a586-80dfde8ee5bf',
     'e618ff7b-3859-45ed-bd32-9397ea4f2a9a',
     'e1a969bc-90cb-4bec-9081-f4110c92565f',
     'ce871565-1aa3-46a8-990d-0dd78aac78ba',
     'a4b4c693-29ba-45fa-bf26-b8d2c0e9d9f0',
     '60d95dae-b2b7-4e05-b29f-13a1bd32bddc',
     'a6d5b89e-53a6-4c90-a405-a8e0d1a30eee',
     'aa446f53-0e8f-45ec-a611-a380ef107364',
     'ef62485a-50d0-4c90-8a42-8a99f3195620',
     'a8046960-bdbe-45f4-b68f-dcfb4cce854b',
     '5b971053-2af8-4b2a-b33b-0976ef43e1e0',
     'bee65c16-fe7c-4bf0-9829-5df77a61e5e4',
     '36ba4fb2-b2da-4127-96d6-f6904811624e',
     '666fc964-2c18-465e-9340-e93a3524a6ee',
     '2c1780a1-a36e-4757-b5b3-9dc43fd7ad74',
     'd6c0aadc-7210-4bfd-abaf-15a8b28462ff',
     '3bac2ac5-abcb-409c-9666-e21fa5d3564d',
     'e00f88b5-55d6-40ba-86bd-29450c5fcf0f',
     '9d0cefb4-5156-4839-b20b-4355df23fdbf',
     '2c88e42d-05f6-457b-96e0-c1be3386e753',
     '097bd2d4-8013-4e1a-a3b9-e54ad0754719',
     '6d62afd5-9a65-429f-9610-9c75316212b0',
     '7fef4e25-494a-41bf-8d91-bb51ba5c2122',
     '97ff60ed-b89e-495f-8642-ec6f1bda87a4',
     'c1529076-1af2-43c1-9ade-add6f407142d',
     '49e0e25b-6896-4054-a86f-d6ccf6834c50',
     'f55c19dc-59af-46cd-a3cd-9d325836bef8',
     '29b1ec7f-eba7-45e7-946e-28662ab7fb0b',
     '10c77730-409f-44c8-8ab0-30f8b8549a14',
     'b3b74030-124b-42ea-bd04-6032b7299c20',
     '16092d4c-ca2c-4c01-9846-ee6b72b78d8b',
     '7e1bb61d-db2a-4b80-afac-7319b48fee66',
     'a5a1beb3-d517-4ffa-803e-daa47f968b9d',
     '7d95caa1-a2d9-4211-93fa-bb9a019f863b',
     '5a45b2ec-60b2-45fa-9177-f3bbd1b90c76',
     '8bf9b56f-d0b1-415d-a9ae-b0441b3c6b49',
     'a14864e0-cd5c-4e25-88ab-cfb6d1028892',
     '136c7564-0de4-4d14-8c47-e8a2fd694b6a',
     '5748da3b-b8de-4222-9707-eea5f9888568',
     '99532176-c411-4123-a8cf-2e5d14eeba56',
     'f3d25529-badd-41e5-b360-f0d0c50ec663',
     '81c1f9c8-78c3-4939-80a6-ad50008c72ab',
     '7800959c-396c-40c7-8df4-c9888a905f6b',
     '6cfd2868-75cb-4828-bb11-d4d83d495d40',
     'd1fa12f4-3038-4a89-95a5-cc0912139693',
     '71588fa2-1607-43fa-9395-feef314c1e5f',
     'e6081d9c-0d28-4bc1-b0a5-1b7eb5f9f78e',
     '494bced7-b87a-4dcc-9dd0-c4e4a8cec810',
     'f262cae0-671c-4053-8805-2b3b73d9afa9',
     '4313cd67-6980-4def-9001-044290dda6ab',
     '31d2a2bf-9e10-4f40-a68e-8513f97079c4',
     '69416e0f-a7f3-4229-afef-05a16ce963f5',
     '7537a319-d89f-4483-b846-de3b1e2003fb']




```python
# call get_path_result API

import time
#time.sleep(30)   # Delays for 5 seconds. You can also use a float value.
def get_patth_result(Get_Path_Calulation_Result_url, headers, token):
    headers["Token"] = token
    try:
        response = requests.get(Get_Path_Calulation_Result_url, headers = headers, verify = False)
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
        Get_Path_Calulation_Result_url = nb_url + "/ServicesAPI/API/V1/CMDB/Path/Calculation/" + str(ress[i]) + "/Result"
        final = get_patth_result(Get_Path_Calulation_Result_url, headers, token)
        d = json.dumps(final)
        j = json.loads(d)
        Final.append(final)
    
Final[0:10]
```




    [{'hopList': [{'hopId': '4999cdf5-e828-4812-b0e6-8974b5771d47',
        'srcDeviceName': 'R20',
        'inboundInterface': 'Ethernet3/0',
        'mediaName': '',
        'dstDeviceName': 'R3',
        'outboundInterface': 'Ethernet0/3',
        'nextHopIdList': ['a685d6c4-460d-46cb-b34b-1696692e1443']},
       {'hopId': 'a685d6c4-460d-46cb-b34b-1696692e1443',
        'srcDeviceName': 'R3',
        'inboundInterface': 'Ethernet0/0.99',
        'mediaName': '102.2.123.0/30\r\nVRF:INET',
        'dstDeviceName': 'ISP',
        'outboundInterface': 'Ethernet0/0.99-102.2.123.1(INET)',
        'nextHopIdList': []}],
      'statusCode': 794008,
      'statusDescription': "Task '72102f34-7265-41ed-aa26-c5558fa1cc60' is failed"},
     {'hopList': [{'hopId': '9c2f0f36-fa72-4f86-9676-facb3aaf7921',
        'srcDeviceName': 'R3',
        'inboundInterface': 'Ethernet0/1',
        'mediaName': '123.10.1.8/30',
        'dstDeviceName': 'R2',
        'outboundInterface': 'Ethernet0/1',
        'nextHopIdList': ['3cb3ad1b-bc3f-4833-bf04-a6ae905ab2da']},
       {'hopId': '3cb3ad1b-bc3f-4833-bf04-a6ae905ab2da',
        'srcDeviceName': 'R2',
        'inboundInterface': 'Ethernet0/3',
        'mediaName': '',
        'dstDeviceName': 'R20',
        'outboundInterface': 'Ethernet2/0',
        'nextHopIdList': ['a84900ed-e0b5-4e31-8698-86244aefc46e']},
       {'hopId': 'a84900ed-e0b5-4e31-8698-86244aefc46e',
        'srcDeviceName': 'R20',
        'inboundInterface': 'Ethernet3/0',
        'mediaName': '',
        'dstDeviceName': 'R3',
        'outboundInterface': 'Ethernet0/3',
        'nextHopIdList': ['e5c35c60-3faa-430c-a0a5-aba2994c3899']},
       {'hopId': 'e5c35c60-3faa-430c-a0a5-aba2994c3899',
        'srcDeviceName': 'R3',
        'inboundInterface': 'Ethernet0/0.99',
        'mediaName': '102.2.123.0/30\r\nVRF:INET',
        'dstDeviceName': 'ISP',
        'outboundInterface': 'Ethernet0/0.99-102.2.123.1(INET)',
        'nextHopIdList': []}],
      'statusCode': 794008,
      'statusDescription': "Task '294d3f2a-4393-482c-9d20-98adbf0ece1b' is failed"},
     "'dict' object has no attribute 'json'",
     {'hopList': [{'hopId': 'f688e220-9b2c-4915-ac8c-415e60c483d3',
        'srcDeviceName': 'R4',
        'inboundInterface': 'Ethernet0/0',
        'mediaName': '123.10.1.16/30',
        'dstDeviceName': 'R2',
        'outboundInterface': 'Ethernet0/2',
        'nextHopIdList': ['5f7a6f7f-87ae-4fe3-a8ad-47168af6466b']},
       {'hopId': '5f7a6f7f-87ae-4fe3-a8ad-47168af6466b',
        'srcDeviceName': 'R2',
        'inboundInterface': 'Ethernet0/3',
        'mediaName': '',
        'dstDeviceName': 'R20',
        'outboundInterface': 'Ethernet2/0',
        'nextHopIdList': ['26a99a7b-5112-48d6-9dbd-4a19276d37c4']},
       {'hopId': '26a99a7b-5112-48d6-9dbd-4a19276d37c4',
        'srcDeviceName': 'R20',
        'inboundInterface': 'Ethernet3/0',
        'mediaName': '',
        'dstDeviceName': 'R3',
        'outboundInterface': 'Ethernet0/3',
        'nextHopIdList': ['152ea66b-32c1-4690-8d8b-1b8bcca214ce']},
       {'hopId': '152ea66b-32c1-4690-8d8b-1b8bcca214ce',
        'srcDeviceName': 'R3',
        'inboundInterface': 'Ethernet0/0.99',
        'mediaName': '102.2.123.0/30\r\nVRF:INET',
        'dstDeviceName': 'ISP',
        'outboundInterface': 'Ethernet0/0.99-102.2.123.1(INET)',
        'nextHopIdList': []}],
      'statusCode': 794008,
      'statusDescription': "Task 'bfd9b6f4-cb34-4c82-b366-a2e088dc0e1b' is failed"},
     {'hopList': [{'hopId': '30fb1a7e-aba5-4f81-861b-f1adaaa4f2c3',
        'srcDeviceName': 'R1',
        'inboundInterface': 'Ethernet0/2',
        'mediaName': '123.10.1.0/30',
        'dstDeviceName': 'R4',
        'outboundInterface': 'Ethernet0/1',
        'nextHopIdList': ['ca888d42-2378-49d5-bd1d-5f0aadb774ef']},
       {'hopId': 'ca888d42-2378-49d5-bd1d-5f0aadb774ef',
        'srcDeviceName': 'R4',
        'inboundInterface': 'Ethernet0/0',
        'mediaName': '123.10.1.16/30',
        'dstDeviceName': 'R2',
        'outboundInterface': 'Ethernet0/2',
        'nextHopIdList': ['927ff0a3-a103-40d5-8a18-b04a06164f41']},
       {'hopId': '927ff0a3-a103-40d5-8a18-b04a06164f41',
        'srcDeviceName': 'R2',
        'inboundInterface': 'Ethernet0/3',
        'mediaName': '',
        'dstDeviceName': 'R20',
        'outboundInterface': 'Ethernet2/0',
        'nextHopIdList': ['f35315e5-faac-49b6-b6ee-7bc1a4cfa647']},
       {'hopId': 'f35315e5-faac-49b6-b6ee-7bc1a4cfa647',
        'srcDeviceName': 'R20',
        'inboundInterface': 'Ethernet3/0',
        'mediaName': '',
        'dstDeviceName': 'R3',
        'outboundInterface': 'Ethernet0/3',
        'nextHopIdList': ['11dc27e4-3cef-4ada-8d02-b2e7ee01d7a1']},
       {'hopId': '11dc27e4-3cef-4ada-8d02-b2e7ee01d7a1',
        'srcDeviceName': 'R3',
        'inboundInterface': 'Ethernet0/0.99',
        'mediaName': '102.2.123.0/30\r\nVRF:INET',
        'dstDeviceName': 'ISP',
        'outboundInterface': 'Ethernet0/0.99-102.2.123.1(INET)',
        'nextHopIdList': []}],
      'statusCode': 794008,
      'statusDescription': "Task 'fb6b646d-ae49-4d31-92c2-5f373f535cec' is failed"},
     "'dict' object has no attribute 'json'",
     "'dict' object has no attribute 'json'",
     {'hopList': [{'hopId': '2c9e8cab-2ed2-41fd-b567-539635be08b0',
        'srcDeviceName': 'R7',
        'inboundInterface': 'Ethernet0/1',
        'mediaName': '123.10.1.24/30',
        'dstDeviceName': 'R6',
        'outboundInterface': 'Ethernet0/1',
        'nextHopIdList': ['1da498a8-7f2f-46dd-ae67-30925a623366']},
       {'hopId': '1da498a8-7f2f-46dd-ae67-30925a623366',
        'srcDeviceName': 'R6',
        'inboundInterface': 'Ethernet0/2',
        'mediaName': '123.10.1.20/30',
        'dstDeviceName': 'R4',
        'outboundInterface': 'Ethernet0/2',
        'nextHopIdList': ['74d6f440-5bf2-4f1a-ba64-c0cef8505683']},
       {'hopId': '74d6f440-5bf2-4f1a-ba64-c0cef8505683',
        'srcDeviceName': 'R4',
        'inboundInterface': 'Ethernet0/0',
        'mediaName': '123.10.1.16/30',
        'dstDeviceName': 'R2',
        'outboundInterface': 'Ethernet0/2',
        'nextHopIdList': ['8b14660d-606b-44cb-bcca-85ed53683c05']},
       {'hopId': '8b14660d-606b-44cb-bcca-85ed53683c05',
        'srcDeviceName': 'R2',
        'inboundInterface': 'Ethernet0/3',
        'mediaName': '',
        'dstDeviceName': 'R20',
        'outboundInterface': 'Ethernet2/0',
        'nextHopIdList': ['49c61972-ffe2-4629-8f43-cbc22ffaf841']},
       {'hopId': '49c61972-ffe2-4629-8f43-cbc22ffaf841',
        'srcDeviceName': 'R20',
        'inboundInterface': 'Ethernet3/0',
        'mediaName': '',
        'dstDeviceName': 'R3',
        'outboundInterface': 'Ethernet0/3',
        'nextHopIdList': ['25f820ea-1a52-4727-ba58-bee89da6764e']},
       {'hopId': '25f820ea-1a52-4727-ba58-bee89da6764e',
        'srcDeviceName': 'R3',
        'inboundInterface': 'Ethernet0/0.99',
        'mediaName': '102.2.123.0/30\r\nVRF:INET',
        'dstDeviceName': 'ISP',
        'outboundInterface': 'Ethernet0/0.99-102.2.123.1(INET)',
        'nextHopIdList': []},
       {'hopId': '8ec48dc8-efe2-4336-ac56-892049eb0037',
        'srcDeviceName': 'R7',
        'inboundInterface': 'Ethernet0/2',
        'mediaName': '123.10.1.28/30',
        'dstDeviceName': 'R5',
        'outboundInterface': 'Ethernet0/2',
        'nextHopIdList': ['ea036835-ac4a-4280-9d83-371f723df8eb']},
       {'hopId': 'ea036835-ac4a-4280-9d83-371f723df8eb',
        'srcDeviceName': 'R5',
        'inboundInterface': 'Ethernet0/0',
        'mediaName': '123.10.1.12/30',
        'dstDeviceName': 'R3',
        'outboundInterface': 'Ethernet0/2',
        'nextHopIdList': ['fc3d708c-de44-4a36-855e-b7fa3d23a8f2']},
       {'hopId': 'fc3d708c-de44-4a36-855e-b7fa3d23a8f2',
        'srcDeviceName': 'R3',
        'inboundInterface': 'Ethernet0/1',
        'mediaName': '123.10.1.8/30',
        'dstDeviceName': 'R2',
        'outboundInterface': 'Ethernet0/1',
        'nextHopIdList': ['8b14660d-606b-44cb-bcca-85ed53683c05']}],
      'statusCode': 794008,
      'statusDescription': "Task 'e0eb3fbf-272c-43f5-9d2c-ce7a4a98555a' is failed"},
     "'dict' object has no attribute 'json'",
     {'hopList': [{'hopId': 'f13f131c-1e2d-45b2-9bf8-16a3beb216a4',
        'srcDeviceName': 'R10',
        'inboundInterface': 'Ethernet0/2',
        'mediaName': '',
        'dstDeviceName': 'SW4',
        'outboundInterface': 'Ethernet0/2',
        'nextHopIdList': ['a003e592-2d95-419a-a454-77ae7c94c5d5']},
       {'hopId': 'a003e592-2d95-419a-a454-77ae7c94c5d5',
        'srcDeviceName': 'SW4',
        'inboundInterface': 'Ethernet2/0',
        'mediaName': '',
        'dstDeviceName': 'SW3',
        'outboundInterface': 'Ethernet2/0',
        'nextHopIdList': ['f1818e17-a530-4548-b679-44681f9df295',
         '557f6695-8fde-45c4-8474-79e7acc22e47']},
       {'hopId': 'f1818e17-a530-4548-b679-44681f9df295',
        'srcDeviceName': 'SW3',
        'inboundInterface': 'Ethernet2/0',
        'mediaName': '',
        'dstDeviceName': 'SW4',
        'outboundInterface': 'Ethernet2/0',
        'nextHopIdList': ['a003e592-2d95-419a-a454-77ae7c94c5d5']},
       {'hopId': '557f6695-8fde-45c4-8474-79e7acc22e47',
        'srcDeviceName': 'SW3',
        'inboundInterface': 'Ethernet0/1',
        'mediaName': '',
        'dstDeviceName': 'R9',
        'outboundInterface': 'Ethernet0/1',
        'nextHopIdList': ['eb5922d5-fa4c-4e5b-b8b3-d0d12f790d8e']},
       {'hopId': 'eb5922d5-fa4c-4e5b-b8b3-d0d12f790d8e',
        'srcDeviceName': 'R9',
        'inboundInterface': 'Ethernet1/0',
        'mediaName': '33.34.4.0/30',
        'dstDeviceName': 'AS30000',
        'outboundInterface': 'Ethernet1/0-33.34.4.1',
        'nextHopIdList': []}],
      'statusCode': 794008,
      'statusDescription': "Task '84a62b19-992e-4600-b4a8-e933e44247c4' is failed"}]



From the final result, we can see all tasks are failed, that is because in Step 8 we couple the gateway addresses randomly, which means that is possible for each pair of the gateways, there is no through path between them. And as a clarification, if users get the taskId from Step 8 successfully, that is not means the path is exist during gateway pairs. 


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

## References:
> 1) login API:

>https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/STANDARD_formate_TEST1_LOGIN_API.ipynb<br> 

> 2) get_all_accessible_tenants API:

>https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/STANDARD_formate_TEST1_Get_All_Asseccible_Tenants_API_Test1%20.ipynb<br>

> 3) get_all_accessible_domains API: 

>https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/STANDARD_formate_Get_all_accessible_domains_of_a_tenant%20_Test1%20.ipynb<br>

> 4) specify_a_working_domain API: 

>https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/STANDARD_formate_Specify_a_domain_to_work_on_API_Test1%20.ipynb<br>

> 5) get_all_devices API: 

>https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/Device%20API%20Design/STANDARD_formate_Get_Devices_API_Test.ipynb<br>

> 6) resolve_device_gateway API: 

>https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/Path%20API%20Design/STANDARD_formate_Resolve_Device_Gateway_API_Test.ipynb<br>

> 7) calculate_path API: 

>https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/Path%20API%20Design/STANDARD_formate_Calculate_Path_API_Test.ipynb<br>

> 8) get_patth_result API:

>https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/Path%20API%20Design/STANDARD_formate_Get_Path_Calculation_Result_API_Test.ipynb

> 9) logout API: 

>https://github.com/Gongdai/Netbrain_REST_API_First_Regularization/blob/master/Netbrain_REST_API/API_test/STANDARD_formate_Logout_Test1%20.ipynb


```python

```
