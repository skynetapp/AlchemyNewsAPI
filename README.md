#### Date: 08-03-2017
#### Description: This document aims to define the Alchemy News API coding.


#### The Folder Structure is as follows:
   
   
   Root Directory | Sub Directory | Sub Directory 
------------ | ------------- | -------------
index.php | | |
Global | DBmongo(Mongo Connection),DBmysql(MySQL Connection), AlchemyAPI(Alchemy API Connection)  | 
Lib | Smarty,Common functions,AlchemyAPI | |
Modules | AlchemyNews | Alchemy News Controller, Alchemy News Action, Alchemy News MySql, Alchemy News View, Alchemy News DB Mongo|
Views | AlchemyNews | header.tpl, footer.tpl(Common files), alchemyNewsForm.tpl, alchemyNewsmasterList.tpl, alchemyNewsdetailList.tpl|

#### Code Flows as follows:
   * To insert or get data from DB code flows.. index.php -> Controller -> Action -> MySql.
   * To view the data code flows.. index.php -> Controller -> View.
   
 
#### Step 1:
  Add the created Url and Alchemy Key in the config.php under bluemix2.0 folder.we can get the Alchemy API Key by logging into IBM Bluemix. 
	
**_Code:_**
	
```
	
$GLOBALS['alchemyNews_apiKey']='xxxxxxxxxxxxxxxxxxxxxxxxxx';
$GLOBALS['alchemy_url']='https://gateway-a.watsonplatform.net/calls';
	
```
	
  
#### Step 2:
  Create a module name as 'alchemyNews' in index.php and respective actions will be performed accordingly.
  
#### Step 3:
   In the server normally we will be able to see News form by running the url -  http://159.203.239.91/bluemix2.0/index.php?module=alchemyNews&action=AddList with the following fields.
   
- Start Date
- End Date
- Keyword
- Type
- Category
   
#### Step 4:
   When we submit the form, action **InsertList** will be called and function **insertAlchemyNewsResponseIntoDB** will get the data.
   
#### Step 5:
   All the details will be set to function **getAlchemyNewsUrl()** from Lib -> AlchemyNewsAPI  and sends request to Alchemy API response function **entities('text', $text, null)**.
   
#### Step 6:
   On receiving response from API, the JSON response will be stored in Mongo DB by calling function  **insertAlchemyNewsJSONResponseIntoMongo(json_encode($data))**.

#### Step 7:
   The response data, type and the request will be passed to function **insertNewsMasterDataIntoMySQL** from Controller to action.
   
#### Step 8:
   In action, first mysql connection will be set by function **setAlchemyNewsMySQLConnection** which gets the DB connection from DBMySQL.
 
#### Step 9:
  Function **insertNewsMasterDataIntoMySQL** from AlchemyNewsMysql.php, will be called in action and inserts the news in master table by calling function **insertIntoNewsMasterData**.
  
#### Step 10:
   Here based on the master request id, the response will be stored in Child table by function **getNewsParsedDataFromJSONResponse($masterId)**.


#### Step 11:
To get the Master list function **getMasterNewsDataFromMySQL** will be called from index.php to controller.

- Function **getAllNewsMasterDataFromMySQL** will be called from controller to action.
- In action, MySQL connection will be set and based on the query it will get the master list table records.
- The result list will be returned to controller.
   
#### Step 12:
   To view the Master list function **showExtractedMasterListView($list)** will be called from controller to View.
   
**_Code:_**

```
  function showExtractedMasterListView($data_arr){
        $smarty = new Smarty();
        $smarty->assign('base_path',$GLOBALS['base_path']);
		$smarty->assign('cursor',$data_arr);
		
	    $smarty->display(''.$GLOBALS['root_path'].'/Views/AlchemyExtract/allMasterList.tpl');
    }
    
``` 

#### Step 13:
   To view the Child data based on the master id, function **getAlchemyNewsChildDataFromMySQL($post_data)** will be called from index.php to controller.
   
   - Function **getAllNewsChildDataFromMySQL($post_data)** will get the records based on the master id using MySql query. 
   - Function **showDetailNewsListView($alchemy_list_vo)** will be called in view. 
   
#### Assumptions

- DBMongo - Inserts the Alchemy JSON response into mongo. It is included in Global -> DBMongo.

#### Errors

If any erros found, the following may be the reasons.

- Url might not be correct. It should be as below.

**_Url:_**

```
http://159.203.239.91/bluemix2.0/index.php?module=alchemyNews&action=AddList

```
- The root path, base path and the database name should be correct in config.php.
- The alchemy API key should be a valid key from IBM Bluemix. If the key is expired, the results may not be appeared.
- If the day limit is exceeded for the API key, then also the child results will not be inserted. 


#### MySQL Database Details

  
 Database Name: bluemix
 
 Tables | Description | Fields |
------------ | ------------- | ----------
alchemy_news_master | Stores the form submitted data to one record | master_id, start_date, end_date, keyword, sentiment_type, taxonomy, requested_datetime |
alchemy_news_child | Stores the child records based on master id | child_id, master_id, published_date_confident, published_date_date, title, time_stamp,response_datetime |
 
 
#### Mongo Database details
 
Database Name: lytepole
 
 



   
