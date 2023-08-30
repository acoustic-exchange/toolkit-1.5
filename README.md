# Exchange Toolkit 1.5

## NEW FEATURES ADDED TO VERSION 1.5

1) Fields marked as "is_original=true" will be now contained in its own field inside the tsv file, i.e:
	          identifier_email	                         identifier_email_original
	ubxtestuser@ubx.com,anothertestuser@ubx.com	            ubxtestuser@ubx.com

	While identifier_email will contain all values found, identifier_email_original will contain the "is_original" value.
	
	New config variable ubx.toolkit.separator created (Default=,) separator for joining values for multiple identifiers having same name.
	
	You will need to Modify the corresponding mapping so it points to the new "is_original" field. Find next some examples:
			
    a) Map the new "is_original" field to a currently-existing database field. This is an easy way to set the new field since it does not need changes at Database level.
   ```
      <Column>
         <Name>Email</Name>
         <EventField>identifier_email_original</EventField>
      </Column>
   ```
      
    b) Map both "normal" and "is_original" fields.
   ```
      <Column>
         <Name>Email</Name>
         <EventField>identifier_email</EventField>
      </Column>
      <Column>
         <Name>Email_Original</Name>
         <EventField>identifier_email_original</EventField>
      </Column>
   ```
      
     For this approach you will need to perform the next extra steps at Database level:
     * Add the missing "Email_Original" field to the table
     * Increase the size of the "Email" field to your convenience since it can now contain more than one value. Take in count that
       this column can contain comma-separated values so you will need to perform the corresponding changes to your implementation
       so it can support this kind of input.
				
    Note: You should not need to update / change any mapping or DB unless you need the "is_original" value in DB in a specific column of it's own, ensuring backward compatibility in events import. 

2) On errors caused by the TSV validation process, identified as HeaderException, such as "empty file" , "no event code" and "no tsv extension" errors, customer will be able to:

    * Define if the tsv file should be moved to dataProceed / dataError folders. A new config variable tsv.onheadererror.movefile has been created (Default=false)
    * Define if the toolkit should continue executing or not. A new config variable tsv.onheadererror.stopExecution has been created (Default=true)

3) Database-generated error messages now feature a more descriptive explanation by adding the information of the table's and field's name. This is since some database engines such as some configurations for MS SQL Server shows very generic and non-descriptive error messages (no field nor table name included)

4) Issue related to toolkit showing an incorrect error count in the console´s summary fixed

5) An explicit exit code has been added once the script finished its execution


## INSTALLATION AND CONFIGURATION

1. Install and configure the environment files

    Depending on your operating system, copy and rename <CU_HOME>\bin\example_setenv.bat or example_setenv.sh to setenv.bat or setenv.sh
    Modify the following variables in the setenv file:

    JAVA_HOME: Enter the path to the JRE.
    CU_HOME: Enter the path to the directory where you extracted the Exchange Toolkit files.
    JDBC_CLASSPATH: Enter the path to the JDBC driver for the database importer.

2. Install the configuration properties and logback files.

    Copy and rename the default example files that are provided in the conf directory.
    
    Rename example_logback.xml to be logback.xml.
    Rename example_config.properties to be config.properties.
    Rename example_jdbc.properties to jdbc.properties.
    Note: Do not rename config.properties or jdbc.properties after you create them. The Exchange Toolkit requires these default configuration files in the CU_HOME directory. If you need to create alternative configuration or data source files, you can create multiple other copies with different names and different values so that you can override the default settings, if necessary.

3. Registering an endpoint using the Toolkit

    To share event and audience data between Exchange endpoints, users must register each endpoint with Exchange. After initiating the endpoint registration process in the Exchange UI, Exchange users can acquire their authentication key in the Exchange Endpoint details menu and then provide the key to the endpoint provider.
    Each time an endpoint communicates with Exchange, the endpoint must submit the authentication key as the authorization bearer or URL parameter in calls to Exchange APIs. Each authentication key is unique to a specific Exchange account and for use by a single endpoint.
    
    Depending on your corporate data and network security policies, you can perform either or both of the following configurations.
    
    * Connect to Exchange through a web proxy server
    * Encrypt the Exchange Toolkit configurations
    * To use the Exchange Toolkit to register an endpoint with Exchange, you provide an endpoint-level authentication key as a value in the Exchange Toolkit properties file.
    
    Running the Exchange Toolkit endpoint registration script makes an API call that includes the authentication key so that Acoustic Exchange can authenticate with the endpoint provider.
    
    Note: To register cloud-based endpoints that are not supported by the Exchange Toolkit, you can register the endpoint directly through the endpoint registration wizard.
  
    * Log into Exchange with your Exchange account login credentials
    * On the Endpoints tab, click Register new endpoint to display the endpoint registration wizard.
    * From the list of provided endpoints, select Exchange Toolkit and select Next.
    * The endpoint appears on the Endpoints tab as Pending.
    * On the Endpoints tab, open the endpoint details menu of your endpoint and copy and save the Endpoint Authentication Key.
    * In the config.properties file, enter the Endpoint Authentication Key in the ubx.application.endpoint.authentication.key field.
    * From the $CU_HOME/bin folder, run registerEndpoint.bat or registerEndpoint.sh. script.
    * To confirm the successful endpoint registration, log in to Acoustic Exchange to find your endpoint listed as Active.
    
    Note: If you are upgrading from an earlier version of the Acoustic Exchange Toolkit, you will not need to complete this step as your current endpoint will still function. However, notice that the property names have changed. Update the property names and event mapping file in your Acoustic Exchange Toolkit installation to match the names that are provided in the latest version.
  
    After you have completed the Acoustic Exchange Toolkit and configuration process, return to the UI to select the events and audiences to which you want to subscribe. Configure an event subscription between the event source endpoint and the event destination endpoint.

4. Modify config.properties

    Modify the content of config.properties to suit your installation. Replace the default values with values that are appropriate for your business application and requirements.

5. Modify jdbc.properties
    
    If you plan to import event data into a SQL database, modify the content of jdbc.properties to suit your installation. Configure the required parameters in this file:
    
    jdbc.driver: Specify the JDBC driver class that is used by the database where you import event data. Example, if you are using a DB2 database, jdbc.driver=com.ibm.db2.jcc.DB2Driver;
    jdbc.url: Specify the database type, database driver, host, port, database name, and schema name, if applicable. Consult your database documentation for specific instructions.
    
    Note: If you are using a DB2 database, you must specify the schema name, and it must be specified in uppercase characters.
    
    For example, if you are using a DB2 database, jdbc.url=jdbc:db2://exampleServer:<1234>/DBname:currentSchema=<SCHEMA_NAME>
    
    jdbc.user: Specify the name of a database user that can automatically access, and write to, the database where you import event data.
    
    Note: Database users are restricted to the schemas that they have permission to write to. Moreover, the database user you specify in jdbc.user must have permission to write to the schema you specified in jdbc.url.
    
    For example, jdbc.user=databaseUserName
    
    jdbc.password: Specify the password that is defined for a system user that can automatically access, and write to, the database where you import event data.
    Note:The password must correlate with the system user who is specified in jdbc.user.
    Example, jdbc.password=databaseUserPassword
    
    After you have configured the schema, and are using recognized Exchange events, run the DDL the corresponds to the SQL database you use. The Toolkit includes DDL for three database types. They are:
    
    camp_ubx_tab_db2.sql
    camp_ubx_tab_ora.sql
    camp_ubx_tab_sqlsvr.sql
    
    Note: If you are upgrading from a previous version of the Acoustic Exchange Toolkit, you must upgrade in order of succession. For example, if you are upgrading from Toolkit 1.0 then you first must upgrade to 1.2 before you can upgrade to 1.3. This is required if you intend to use recognized Exchange events.

6. Update audience producer and audience consumer endpoint ID

    In the config.properties file you must update both the ubx.audience.consumer.endpoint.id and the ubx.audience.producer.endpoint.id with your endpoint ID. The endpoint ID is returned registerEndpoint script's log file.
    
    The following is an example of a log generated by the registerEndpoint script:
    ```
    2017-06-23 09:56:18,432 INFO Registered Endpoint ID: 60871
    2017-06-23 09:56:18,433 INFO Please update the configuration file with the endpoint ID for:
    Audience Producer endpoint ID=60871
    Audience Consumer endpoint ID=60871
    2017-06-23 09:56:18,433 INFO Register Endpoint script succeeded.
    2017-06-23 09:56:18,434 INFO * Register Endpoint script completed.
    ```
