# DataEngineeringProject01
# scope : Migrating Onpremise database (Adventure Works) to Azure datafactory then loading on to Datalake Gen2 then transforming using Databricks and to perform analysis using synapse analytics finally visualize data using powerbi dashboard

Resource group 
 0. keyvault -keydemo1
 1. data factory -dfdemo1
 2. data lake gen2 (raw,transform)-dldemo1
 3. azure databricks -dbdemo1
 4. synapse analytics -synapdemo1

    create user janu // SQL
0)  in SSMS create username : janu and password : **********
store these values in key vault 
    
1) SqlDB(SSMS) -------> Azure data factory
   df-demo1 >> manage>> create SHIR >> express setup (to install SHIR application automatically)
(SHIR)self hosted integration  runtime ---->  to connect onpremise data resources to cloud
check: search for integration runtime in your local machine

  2) Ingestion
     df-demo1>>Author>>new pipeline >> drag copy data
     # source :SQL
      new source >> SQL server >> linked service >> onpremisesqlserver >> SHIR>> get server & db name from SSMS >> sql authentication
     create link to onpremise sql for username & password authentication ( before establishing keyvault linked services check the acess policies add df-demo01 to secret access)
     # sink:ADl
     new sink >> ADL GEN2 >> Paraquet (data is stored in column-based format) >>linked service to adl >> AHIR ( since df to dl) >> storage acc>>specify location (/raw-data or bronze)

     debug the pipeline >> refresh the /raw-data to list the tableor files finally publish

     # now lets create a pipe line to automate the above ingestion process ( means all the files are automatically copy to /raw-data in datalake from onpremise sql server)

     create new pipeline

     drag lookup activity >> settings >> SQL server >> linked service >> onpremisesqlserver >> SHIR>>
     in settings tab >> query ( sql query to list all the tables in db)>> debug

     after debug <>
     
