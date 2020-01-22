

# CUSTOMER AWS: CUSTOM PROGRAMMING AND SCRIPTS
## Windows ARC SERVER

Most jobs are controlled by task scheduler on windows and happen off-peak hours. When making changes, please account for UTC (aws console time) arc server time zone (eastern) and your time zone.




- C:\Python27\ArcDropScripts\SERVER\ArcServerCompress.py

     Compresses the databases nightly, so they remain performant. It also clears out the workspace cache.

    ```
    ############################
    ##### Compress the sde #####
    ############################

    #Compress sde to keep things running smoothly
    start_time = time.time()
    try:
        arcpy.Compress_management(r"C:\Users\Administrator\AppData\Roaming\ESRI\Desktop10.5\ArcCatalog\coop.sde")
        print "Coop Compressed!\n"
    except Exception as err:
        if err.message in arcpy.GetMessages(2):
            print arcpy.GetMessages()
        else:
            print unitcode(err).encode("utf-8:")
    elapsed_time = time.time() - start_time
    print elapsed_time
    print " seconds\n"

    # Clear SDE cache
    print "clearing workspace cache for all server connections that are idle. Active connections will not be affected"
    arcpy.clearWorkspaceCache_management()
    end_time_script = time.time() - start_time_script
    print "Time to compress all .sde databases and dump idle caches:
    print end_time_script / 60
    print "Mins\n"

    ```




- C:\Python27\ArcDropScripts\*COOP*\BackupCleanup.py

    This script makes a nightly backup of the default workspace for each sde in the database. It then uses file properties to determine age and deletes anything older than 7 days. This is in conjunction with backups for the google drive. The server as a whole is backed up daily. Backups (disk image) for our backups (XML export) for our backups (google drive).


    ```
    ###########################
    ##### Backup Cleanup ######
    ###########################

    # Import arcpy module
    import arcpy
    import os
    import time
    import sys
    import datetime


    # put the coop here
    coop = 'some coop name here'

    # time stuff, mess with time carefully you could end up in the past :)
    now = time.time()
    created = datetime.date.today()
    created = created.strftime("*ymd")

    # path where "file management" takes place
    path = "C:\\Backups\\"+coop.upper()+"\\"
    print "Purging old backups in " + path

    # iterating through all the files in the directory and then deleting the ones older than 7 days
    for a in os.listdir(path):
        a = os.path.join(path, a)
        # print a
        if os.stat(a).st_mtime < now - 7 * 86400:
            # print os.stat (a)
            if os.path.isfile(a):
                os.remove(os.path.join(path, a))

    # prep arcpy env
    arcpy.env.overwriteOutput = True
    print " flag set to overwrite existing data!"
    # Local variables:
    sde = "C:\\Users\\Administrator\\AppData\Roaming\\Esril\Desktop10.5\\ArcCatalog\\"+coop.lower()+".sdel\" + coop.upper() + ".DBO.sdm_CrescentLink"
    xml = "C:\\Backups\\"+coop.upper()+"\\"+coop.lower()+"cl" + created + ".xml"
    print "exporting " + xml
    # Process: Export XML Workspace Document
    arcpy.ExportXMLWorkspaceDocument_management(
        sde, xml, "DATA", "BINARY", "METADATA")
    print "Crescentlink Backup Complete"
    # Local variables:
    sde = "C:\\Users\\Administrator\\AppData\Roaming\\Esril\Desktop10.5\\ArcCatalog\\"+coop.lower()+".sdel\" + coop.upper() + ".DBO.sdm_WorkOrder"
    xml = "C:\\Backups\\"+coop.upper()+"\\"+coop.lower() + "WO" + created + ".xml"
    print "exporting " + xml
    # Process: Export XML Workspace Document
    arcpy.ExportXMLWorkspaceDocument management(sde, xml, "DATA", "BINARY", "METADATA")
    print "Crescentlink WO Backup Complete"

    ```


- C:\Python27\ArcDropScripts\\Subscriber.py

    This is a nightly MySQL query to bring the drop website data into the mapping so it can be published on the web service. Other data can be added to the sde if you wish.


    ```
    ###################################################################
    ### Copy Subscriber Data from local webserver to arc gis server ###
    ###################################################################

    import mysql.connector
    import csv
    import arcpy

    # internal only address for dropwebsite
    sql_connect = mysql.connector.connect(user='someuser', password='password', host='xxx.xxx.xxx.xxx', database='joomla')
    print "Connected to coop Dropwebsite Database..."
    cursor = sql_connect.cursor()
    query = (r"SELECT * from subscriber")
    print "opening temp file..."
    c = csv.writer(open("Subscribers.csv", "wb"))
    print "Selecting Subscibers..."
    cursor.execute(query)
    rows = cursor.fetchall()
    print "writing column headers..."
    c.writerow(cursor.column_names)
    print "Parsing subs into file...
    for row in rows:
        c.writerow(row)
    print "Cleaning up query..."
    cursor.close()
    sql_connect.close()

    # Setting Local variables:
    print "Setting ArcServer Tool Parameters..."
    subscibers_csv =  r"C:\Python27\ArcDropScriptsCoopSubscribers.csv"
    coop_sde = r"C:\Users\Administrator\AppData\Roaming\EsriDesktop10.5\ArcCatalog\coop.sde"


    print "Running Table to Geodatabase tool to ingest temp CSV..."
    arcpy.env.workspace = coop_sde
    fc = "coop.DBO.Subscribers..."
    if arcpy.Exists(fc):
        print "Updating Table..."
        arcpy.Delete_management(fc)

    # Process: Table To Geodatabase (multiple)
    arcpy.TableToGeodatabase_conversion(subscibers_csv, coop_sde)
    print "COMPLETE!!!"

    ```
