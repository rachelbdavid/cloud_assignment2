# cloud_assignment2
### URK18CS226
## _Design a vital sign monitor and visualise your data in the cloud_
### Cloud servce used: ```Google Cloud```
1. Create a project
2. Goto _Navigation menu -> APIs_ & Services and check whether the following APIs are enabled. If not enabled, enable the APIs by searching the API in the console and click _Enable_.
   * Cloud IoT API
   * Cloud Pub/Sub API
   * Dataflow API
3. In the navigation menu, select _Pub/Sub -> Topics_ and create a new topic is created with name ```iotlab``` by clicking _Create Topic_.
4. A new principal ```cloud-iot@system.gserviceaccount.com``` is added by clicking _+Add Principal_ under _Permissions_ tab on the right hand side with ```Pub/Sub Publisher```.
5. Navigate to _BigQuery_ and create a new dataset with name ```iotlabdataset``` under the current project name.
6. Under -iotlabdataset_, create a new empty table by clicking on the three dots next to dataset name and give the following details
    * Table name: ```sensordata```
    * Under _Schema_ give the following field details by clicking _+_ indicating _Add field_.
        * ```timestamp```, field type is ```TIMESTAMP```
        * ```device```, field type is ```STRING```
        * ```temperature``` field type is ```FLOAT```
 7. A cloud bucket is created by navigating to _Cloud Storage_ and naming the bucket as ```cs226_assignment2``` with _Location_ set to ```Multi-region```.
 8. Navigate to _Dataflow_ and click _+Create job from template_. Name the job as ```iotlabflow```.
 9. For Dataflow template, choose _Pub/Sub Topic to BigQuery_.
 10. For _Input Pub/Sub topic_, enter ```projects/<project-id>/topics/iotlab```.
 11. For _BigQuery output table_, enter ```<project-id>::iotlabdataset.sensordata```.
 12. For tempory location, enter ```gs://<bucket-name>/tmp/```.
 13. Click on _Show optional parameters_ and set _Max workers_ as ```2``` and _Machine Type_ as ```n1-standard-1```. At last, click _Run Job_.
 14. At this step, 2 virtual machines will be created. Click ```SSH -> Open in browser window``` for virtual machine named ```iot-device-simulator```.
 15. A _virtual environment_ is created and _gcloud SDK_ is initialized in the SSH window opened.
 16. In SSH, _log in with new account_ is selected and the current project is picked for usage.
 17. Required software packages and python components ar einstalled in virtual environment.
 18. The ```http://github.com/GoogleCloudPlatform/training-data-analyst``` Git is cloned in SSH for data analysis.
 19. An _IoT registry_ is created with name ```iotlab-registry``` and a _cryptographic keypair_ is created using the below command.
```
cd $HOME/training-data-analyst/quests/iotlab/
openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem \
    -nodes -out rsa_cert.pem -subj "/CN=unused"
```
20. Two devices named ```temp-sensor-buenos-aires``` and ```temp-sensor-istanbul``` are created under _iotlab-registry_ and linking with the RSA256 certificates.
21. These devices are started in the SSH using the following commands. The results will be stored in BigQuery ```iotlabdataset.sensordata```.
```cd $HOME/training-data-analyst/quests/iotlab/
curl -o roots.pem -s -m 10 --retry 0 "https://pki.goog/roots.pem"
```
```python cloudiot_mqtt_example_json.py \
   --project_id=$PROJECT_ID \
   --cloud_region=$MY_REGION \
   --registry_id=iotlab-registry \
   --device_id=temp-sensor-buenos-aires \
   --private_key_file=rsa_private.pem \
   --message_type=event \
   --algorithm=RS256 > buenos-aires-log.txt 2>&1 &
   ```
   ```
   python cloudiot_mqtt_example_json.py \
   --project_id=$PROJECT_ID \
   --cloud_region=$MY_REGION \
   --registry_id=iotlab-registry \
   --device_id=temp-sensor-istanbul \
   --private_key_file=rsa_private.pem \
   --message_type=event \
   --algorithm=RS256
   ```
22. Navigate to _BigQuery_ and run the following queries in two different query tabs to get the required results.
```SELECT timestamp, temperature from iotlabdataset.sensordata
WHERE device = 'temp-sensor-buenos-aires'
ORDER BY timestamp DESC
LIMIT 100
```
```SELECT timestamp, temperature from iotlabdataset.sensordata
WHERE device = 'temp-sensor-istanbul'
ORDER BY timestamp DESC
LIMIT 100
```
23. After the results are generated, the data table is explored in _Data Studio_ by linking the resultant tables.
24. _Time Series_ charts are created for each tables having _timestamp_ in x-axis and _temperature_ in y-axis.
25. Chart from [temp-sensor-buenos-aires](https://github.com/rachelbdavid/cloud_assignment2/blob/main/IMG/sensor1_visualization.png)
26. Chart from [temp-sensor-istanbul](https://github.com/rachelbdavid/cloud_assignment2/blob/main/IMG/sensor2_visualization.png)
