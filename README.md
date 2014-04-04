# app_stub  
## Self updating JS application stub

 

Nowadays, it is easy to drop a web app into Phonegap or Node Webkit to create deployable applications. What is annoyingly hard is updating those apps without redeploying the entire package.

Since the application is Javascript anyway, it should be possible to have a launcher stub which fetches versioned updates in the background from a database. Since CouchDB/PouchDB makes database synchronizing totally painless, it should then be possible to deliver application updates as a single JSON record. The launcher stub would be responsible for copying those files to the local filesystem and booting the application to the most recent app folder. 

### Details: 

Data structure: We will have two databases, an application database and a files database. The application database is very simple, it stores only one type of record -- applications. Each application record has a "deploy_type" which is either "production" or "pre-production", an "app_name" field and also a list of files (not attachments) each with CRC, type, size and URL. A second database holds records for each file and acts as a simple filestore. The client stub will fetch all changed files into a new versioned application folder.

### Launcher boot process: 

 When the launcher starts up, it fetches the current "application_ver", calculates the correct application path (base path + app ver) and launches the app's index.html page. Then, it asynchronously checks for updates, downloads update files if available, creates a new application folder and, when finished, updates the "application_ver" variable. Then it removes older application folders (keeping only the two most recent to allow for simple rollbacks.)

 The important thing here is that 1) the initial launch is done immediately so that user does not see a launch delay. Also 2) downloading the new version is done entirely asynchronously and in the background so the user does not see performance degradation. 

The stub pseudo code might look something like this:

### On instantiate:

```

// launch application
current_ver = stub.getVersion();
app_filebase = stub.getFileBase();
stub.launchApp(app_filebase + '/' + current_ver);

// pseudocode is synchronous but this should all be done asynchronously
stub.syncAppDB();
new_ver = stub.getLatestAppVersion();
if (app_ver != new_ver) {
   files = stub.getNewAppFilesList();
   files.foreach(file) {
     old_filepath = app_filebase + '/' + current_ver +'/' + file.path;
     new_filepath = app_filebase + '/' + new_ver +'/' + file.path; 
     old_crc = CRC32(old_filepath);
     if (old_crc = file.crc) copyFile(old_filepath, new_filepath);
       else saveFromURL(file.URL, new_filepath, file.CRC); 
   }
  stub.setVersion(new_ver);
});

```

The final part of the project would be a simple Grunt script with two steps: "test" and "deploy". "Test" would move the contents of the app folder into the database but mark the new app record "deploy_type" as "pre-production". "Deploy" would do the same thing but mark the app record as "production". The Files database would experience no differences (and files uploaded in the "test" build would not be re-uploaded in the "deploy" build unless changed."). There would also have to be some way of setting a launcher stub to subscribe to the "pre-production" updates. Versioning numbers would have to be configurable in the grunt script.
