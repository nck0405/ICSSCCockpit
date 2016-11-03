#BLU Acceleration and Node.js

For issues that you encounter with this service, go to **Get help** in the <a href="https://www.ibmdw.net/bluemix/get-help/">BlueMix development community</a>.

You can bind your Node.js application to BLUAcceleration database and work with your data. To enable this connection you need to use a custom buildpack to download driver dependencies to connect to the BLUAcceleration database from the Node.js runtime. This topic explains how to get your code running using this method. The sample illustrated here uses express and jade node modules.

###Required components

The following components are required to connect BLUAcceleration service from a Node.js application. The are all described in further detail in this topic.

- package.json
- Node.js program
- Buildpack to install dependencies 

####package.json
The package.json contains information about your app and its dependencies. It is used by npm tool to install, update, remove and manage the node modules you need. Add the following ibm_db dependency to your package.json:
```
  {
    "engines": { "node": "0.10.21" },
    "name": "bluaccnodesample",
    "version": "0.0.1",
    "description": "A sample node app",
    "dependencies" : {
      "ibm_db" : "~0.0.1",
	  "express": "3.x",
	  "jade": "x"
    },
    "scripts": {
       "start": "node app.js"
	}
}
```
###Connecting to BLU Acceleration from Node.js code

In your Node.js code, parse the VCAP_Services environment variable to retrieve the database connection information and connect to the database as shown in the following example.

For more information on the structure of the VCAP_SERVICES environment variable see Getting started with the BLU Acceleration service
```
var app = express();

// all environments
app.set('port', process.env.PORT || 3000);
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');
app.use(express.favicon());
app.use(express.logger('dev'));
app.use(express.json());
app.use(express.urlencoded());
app.use(express.methodOverride());
app.use(express.cookieParser('your secret here'));
app.use(express.session());
app.use(app.router);
app.use(express.static(path.join(__dirname, 'public')));
var db2;

var env = JSON.parse(process.env.VCAP_SERVICES);
db2 = env['BLUAcceleration-10.5.3 Rev. A Build 0.1'][0].credentials;

var connString = "DRIVER={DB2};DATABASE=" + db2.db + ";UID=" + db2.username + ";PWD=" + db2.password + ";HOSTNAME=" + db2.hostname + ";port=" + db2.port;

app.get('/', routes.listSysTables(ibmdb,connString));
```
In the routes/index.js file, open the connection using the connection string and query the database as follows.
```
exports.listSysTables = function(ibmdb,connString) {
	return function(req, res) {
		   
	   ibmdb.open(connString, function(err, conn) {
			if (err ) {
			 res.send("error occurred " + err.message);
			}
			else {
				conn.query("SELECT TABNAME,TABSCHEMA FROM SYSCAT.TABLES FETCH FIRST 10 ROWS ONLY", function(err, tables, moreResultSets) {

							
				if ( !err ) { 
					res.render('tablelist', {
						"tablelist" : tables
						
					 });

					
				} else {
				   res.send("error occurred " + err.message);
				}

				/*
					Close the connection to the database
					param 1: The callback function to execute on completion of close function.
				*/
				conn.close(function(){
					console.log("Connection Closed");
					});
				});
			}
		} );
	   
	}
}
```
Please download and examine the attached sample for the rest of the code.
###Uploading your application

Use the -b option of the 'cf push' command to specify the db2nodejsbuildpack when pushing your application to Bluemix

`cf push <app-name> -b https://github.com/ibmdb/db2nodejsbuildpack`

###Related Links
- <a href="http://pic.dhe.ibm.com/infocenter/db2luw/v10r5/index.jsp" target=_blank>IBM DB2 v10.5 Information Center</a>
- <a href="http://www.ibm.com/developerworks/data/products/db2/" target=_blank>IBM DB2 developerWorks</a>
- <a href="https://www.ibm.com/developerworks/community/blogs/pd/entry/using_ibm_db2_from_node_js4?lang=en" target=_blank>Using IBM DB2 from node.js</a>

