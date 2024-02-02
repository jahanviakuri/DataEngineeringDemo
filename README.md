# DataEngineeringProject01
## Scope : Migrating Onpremise database (portfolioproject) to Azure datafactory then loading on to Datalake Gen2 then transforming using Databricks and to perform analysis using synapse analytics finally visualize data using powerbi dashboard

Resource group 
 0. keyvault -keydemo1
 1. data factory -dfdemo1
 2. data lake gen2 (raw,transform)-dldemo1
 3. azure databricks -dbdemo1
 4. synapse analytics -synapdemo1

    create user janu // SQL
0)  in SSMS create username : janu and password : **********
store these values in key vault 
    
1) ## SqlDB(SSMS) ---connection----> Azure data factory 
  Integration runtime :  to perform operations we need compute power and infrastaruture to execute operation 
    AIR ( Azure integeration runtime ) : if you want to connect cloud services ( data factory to data lake)
     SHIR (self hosted integration runtime ) : if you want to connect onpremise db to any cloud service liek datafactory 
   df-demo1 >> manage>> create SHIR >> express setup (to install SHIR application automatically)
check: search for integration runtime in your local machine

  3) ## Ingestion ( from onpremise to data factory)
    
     df-demo1>>Author>>new pipeline >> drag copy data
     ### source :SQL
      new source >> SQL server >> linked service >> name: onpremisesqlserver >> SHIR>> get server & db name from SSMS >> sql authentication >>test connection >> create
     #### create link to keyvault for username & password authentication
     link to keyvault >> system assigned >> create      ( before establishing keyvault linked services check the access policies add df-demo01 to secret access)
     give sceret permisions >> select dfdemo01 >> create ( giving grants to datafactory to read all secrets from keyvault) 
     ### sink:ADL GEN 2 Bronze layer
     new sink >> ADL GEN2 >> Paraquet (data is stored in column-based format) >>linked service to adl >> AHIR ( since df to dl) >> acc key authentication>> storage acc name >>specify location (/raw-data or bronze)

     debug the pipeline >> refresh the /raw-data to list the  files 
     
5) ## Pipeline Automation using Lookup ,for each 
      now lets create a pipe line to automate the above ingestion process ( means all the files are automatically copy to /raw-data in datalake from onpremise sql server)
    In SSMS run a SQL query to get the list of tables from sys.tables  (select s.schemaname as schemaname, t.tablename as tablename from ...)
     drag lookup activity >> settings >> SQL server >> linked service >> onpremisesqlserver >> SHIR>>
     in settings tab >> query ( sql query to list all the tables in db)>> debug
   the o/p of above lookup acticivity after debug is json   values[ { schemaname : "dbo", tablename: "coviddeaths" } , {schemaname : "dbo", tablename: "covidvaccines"}]
    drag for each loop (dynamic action ) --run for each value in json  ( ().values.o/p) add dynamic contnet in seetings>>item>> item= @action(lookup for all table).output.values
   go to activities to go inside the for each loop >> drag copy data >>
   ### source (repeeate above steps)>> query >> add dynamic content >> @concat(select * from item().schemaname,.,item().tablename)
   ### sink ( folder structure : bronze/schema/tablename/tablename.paraquet
   open>> parameters>> schemaname :item().schemaname , tablenbame : item().schemaname
   connection >> file path >> bronze / directory >> dynamic content :@concat(item().schemaname,/,item().tablename) >> fileneme : @concat(item().tablename,.,paraquet)
    publish >> debug or add tigger  then check the bronze layer in ADL

   6) ### data transformation : databricks
   
     
