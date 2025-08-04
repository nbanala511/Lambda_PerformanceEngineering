## Load testing our serverless application  
### Postman Setup  
1. Make sure you have the latest Postman app https://www.postman.com/downloads/  
2. Open the postman desktop app and click on "New" in the top left
3. Select "HTTP" from the options.  
4. Change the method to "POST"  
5. Click on the "Body" and the  select the "raw" option
6. copy and paste the below json into the body 
```json
{
    "operation": "list",
    "tableName": "lambda-apigateway",
    "payload": {
    }
}
```
7. Get the url from API Gateway stages we created previously  
<img width="1123" height="564" alt="APIGW_URL" src="https://github.com/user-attachments/assets/0df6745a-ee90-4acf-a700-ee569cae1973" />  

8. past the url into the url section in the postman  
9. Click on "Save" in the top right.  
10. Click on "Send".  
You should see the response. 

### Load Test  
1. Right click on the new request you just created.  
<img width="1127" height="488" alt="LoadTest_setup" src="https://github.com/user-attachments/assets/9c3a1c69-94ef-4133-abd1-863c3d1dd4e8" />  
2. Click on "Performance", then select "Ramp up" under load profile  
3. select “10” in Virtual users, and Test duration as 2 mins. Click Run!
<img width="508" height="696" alt="Ramup_loadtest" src="https://github.com/user-attachments/assets/7a7e7443-8db0-414f-8163-66443f9460c4" />
4. Atfer the load test is run you can see the result. (YOu can also export the result as pdf).  


#### Now that you have the results from the load test lets change the lambda default memory (128 MB) to some increased memory and run the load test again and see the difference in the performance.





