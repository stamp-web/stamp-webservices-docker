# stamp-webservices-docker
Docker composer container files for host the application in docker

Managing Configuration files to deploy to the server can be done by placing files in the config folder.  This is copied to /tmp/config on the container and after
extracting the archive, will copy the structure over in-place to the application root. 

In Jenkins I may need to approve the following in the in-process Script Approvals

* method hudson.model.Job getLastBuild
* method jenkins.model.HistoricalBuild getNumber
* method jenkins.model.Jenkins getItemByFullName java.lang.String
* staticMethod jenkins.model.Jenkins get