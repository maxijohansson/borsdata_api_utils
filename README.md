# CCM database API Python interface
This package can aid in retrieving data from CCM's data sources into a Python environment. Currently, data can be retrieved from:  
* Börsdata database
* CCM's Guldgruvan database
## Installation
#### pip
```pip install git```  
```pip install git+https://github.com/maxijohansson/borsdata_api_utils.git```
#### conda
```conda install pip```	(probably unnecessary)

If using a conda environment, navigate to the environment's `Scripts` dir (with cd). The `envs` dir is located in the main conda dir.

```...\envs\env_name\Scripts\>```

Install the git package and this package.

```pip install git```  
```pip install git+https://github.com/maxijohansson/ccm-db-api.git```

#### updating
Add  `--upgrade` flag:

```pip install --upgrade git+https://github.com/maxijohansson/ccm-db-api.git```

## Börsdata database
```import ccm_db_api as api_utils```
```borsdata = api_utils.Borsdata_db(key)```

The `Borsdata_db` class contains functions for retrieving data from Börsdata's database. This requires an api key.

## Guldgruvan
```import ccm_db_api as api_utils```
```guldgruvan = api_utils.Guldgruvan(key)```

The `Guldgruvan` class contains functions for retrieving data from CCM's database. This requires an api key.  

### API Overview
* A Python function in this package makes a request to an HTTP endpoint.
* An endpoint in API Gateway calls a Lambda function.
* The Lambda function executes an SQL query to the Guldgruvan database.
* API Gateway returns the output of the Lambda function.
* The Python function translates the JSON output to a Pandas dataframe.

### Available functions:
* ```guldgruvan.instruments()```: Returns a dataframe with all available instruments
* ```guldgruvan.dailyprices(instrument, first, last)```: Returns a dataframe with daily price data for ```instrument``` between dates ```first``` and ```last```


### Creating an endpoint
1. Enter the Lambda service in the AWS Console and click **Create function**.
	* **Function name**: Should refer to the query this endpoint is associated to.
	* **Runtime**: Python 3.8
	* **Execution role**: Use an existing role  -> *RoleForLambda*
	* **VPC**: Choose the one that the database is located in.
	* **Subnets**: Choose 2+
	* **Security groups**: Choose the one that allows incoming access to the database. Probably *launch-wizard-2*.

2. In the **Function code** window / **Actions** / **Upload a file from Amazon S3**, enter the S3 link to the resource with the necessary Python packages for the lambda function: `s3://ccmlambdabucket/pymysql_archive.zip` and save.

3. Create a file *lambda_function.py* in the Lambda function's top directory. Copy the contents from the file *lambda_function.py* in this repo to the *lambda_function.py* file in the **Function code** window.

4. Set the **Environment variables**:
	* ```DATABASE``` := ```guldgruvan```
	* ```DBUSER``` := ```ccm_user```
	* ```ENDPOINT```:= ```guldgruvan.cociqzmze0qh.eu-west-3.rds.amazonaws.com```
	* ```PASSWORD``` := ```***************``` (Password to the RDS database)
	* ```PORT``` := ```3306```  

	Note: A better way of storing the database credentials is with Secrets Manager, so this is subject to change.

5. Implement the database query in *lambda_function.py*.

6. Enter the API Gateway service in the AWS Console and go to **Guldgruvan** / **Actions** / **Create resource**. Choose name and **Create resource**.
7. **Actions** / **Create method** and choose the type of method. (GET if retrieving data)
	* **Integration type**: Lambda function
	* **Lambda Function**: Enter the name of the Lambda function
8. Configure the **Method Request**:

	* **Settings** / **API Key  Required**: True
	* **URL Query String Parameters**: Add parameters for the parameters you want.
9. Add a mapping template to the **Integration request**:
	* **Mapping templates** / **Request body passthrough**: *When there are no...*
	* Add this mapping template:
		* **Content-Type**: application/json
		* Copy the code from the mapping-template file in this repo to the input window and save.
