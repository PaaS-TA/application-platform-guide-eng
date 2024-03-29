### [Index](https://github.com/PaaS-TA/Guide-eng/blob/master/README.md) > [AP User Guide](../README.md) > Go Development

### 1. Outline

#### 1.1. Document Outline

##### 1.1.1. Purpose
This document (Go Application Development Guide) is a document that presents the service packs (Mysql, MongoDB, RabbitMQ, Redis, and ClusterFS) of Open PaaS projects in connection with Go applications to use services and deploy applications.

##### 1.1.2. Range
The range of this document is limited to the Go application development and service pack linkage of Open PaaS projects.

##### 1.1.3. References
-   [***https://code.google.com/p/golang-korea/wiki/EffectiveGo***](https://code.google.com/p/golang-korea/wiki/EffectiveGo)
-   [***https://github.com/brianmario/mysql2***](https://github.com/brianmario/mysql2)
-   [***https://github.com/redis/redis-rb***](https://github.com/redis/redis-rb)
-   [***https://github.com/fog/fog***](https://github.com/fog/fog)
-   [***https://www.jetbrains.com/idea/***](https://www.jetbrains.com/idea/)

### 2. Go Application Development Guide

#### 2.1. Outline
It binds various service packs registered with Open PaaS to applications written in the Go language. Access information for each service is obtained from the bound environmental information (VCAP\_SERVICES) of the application and applied to the application for use and allows to create of Go applications in a Windows environment.

#### 2.2. Development Environment Configuration
Configure the development environment of the Go Application as shown below.
-   OS : Windows 7 64bit
-   Go : 1.5.2
-   IDE : Intellij IDEA  
There are Community and Ultimate versions for Intellij IDEA. Community Version is for Free and the Ultimate version is a 30-day trial version.

##### 2.2.1. Go SDK Installation
1.  Download Go SDK

| <span id="__DdeLink__2953_294360055" class="anchor"></span>https://golang.org/dl/ |
|-----------------------------------------------------------------------------------|

<img src="./images/go/image2.png" width="612" height="151" />

-   Download

Go SDK: go1.5.2.windows-amd64.msi

1.  Install Go SDK

-   Double-click go1.5.2.windows-amd64.msi to execute the installation.

<img src="./images/go/image3.png" width="297" height="170" />

-   Click the “Execute” button

<img src="./images/go/image4.png" width="351" height="275" />

-   Click the “Next” button

<img src="./images/go/image5.png" width="330" height="258" />

-   Click the “Next” button 

<img src="./images/go/image6.png" width="324" height="254" />

-    Click the “Next” button after selecting the directory of the Go SDK to be installed

<img src="./images/go/image7.png" width="330" height="258" />

-   Click the “Install” button

<img src="./images/go/image8.png" width="317" height="247" />

<img src="./images/go/image9.png" width="323" height="252" />

-   Click “Finish”

##### 2.2.2. IntelliJ IDEA Installation
1.  Download IDEA

| http://www.jetbrains.com/idea/ |
|--------------------------------|

<img src="./images/go/image10.png" width="601" height="272" />

<img src="./images/go/image11.png" width="499" height="400" />

<img src="./images/go/image12.png" width="559" height="138" />

-   Download: ideaIC-15.0.2.exe

1.  Install IntelliJ IDEA

-   Double-click ideaIC-15.0.2.exe and execute the installation.

<img src="./images/go/image13.png" width="290" height="225" />

-   Click the “Next” button

<img src="./images/go/image14.png" width="315" height="245" />

-   Set the path to install and click the “Next” button

<img src="./images/go/image15.png" width="315" height="245" />

-   Click the “Next” button

<img src="./images/go/image16.png" width="315" height="245" />

-   Click the “Install” button

<img src="./images/go/image17.png" width="308" height="240" />

<img src="./images/go/image18.png" width="309" height="240" />

-   Select “Run IntelliJ IDEA Community Edition” check

-   Click the “Finish” button

#### 2.3. Development

Data management for sample applications uses either MySQL or MongoDB, so when requesting API, it is determined with the DBType value in the request body.

##### 2.3.1. Connect Sample Application

1.  Connect Go Sample Project

<img src="./images/go/image19.png" width="315" height="219" />

-   Select “Import Project”

<img src="./images/go/image20.png" width="315" height="418" />

-   Set path for the downloaded Go Sample Application

-   Click the “Ok” button

<img src="./images/go/image21.png" width="275" height="269" /> <img src="./images/go/image22.png" width="284" height="278" />

-   Click the “Next” button - Click “Next” the button

<img src="./images/go/image23.png" width="353" height="345" />

-   Click the “Next” button

<img src="./images/go/image24.png" width="339" height="331" />

-   Click the “Finish” button

##### 2.3.2. Sample Application Environment Setting

-   Install Go Plugin and Go Sample Application's environment setting in IntelliJ IDEA Environment 

<img src="./images/go/image25.png" width="535" height="359" />

-   Select File &gt; Settings 

<img src="./images/go/image26.png" width="541" height="366" />

-   Select Plugins &gt; Browse repositories

<img src="./images/go/image27.png" width="549" height="459" />

-   Search for “Go” and click Go from the retrieved list. Click Install button

<img src="./images/go/image28.png" width="556" height="466" />

-   Select “Restart IntelliJ IDEA”

<img src="./images/go/image29.png" width="366" height="135" />

-   Click the “Restart” button

<img src="./images/go/image30.png" width="459" height="308" />

-   Select File &gt; Project Structure...

<img src="./images/go/image31.png" width="462" height="384" />

-   Select “New” from the Project SDK domain

<img src="./images/go/image32.png" width="451" height="375" />

-   Select “Go SDK”

<img src="./images/go/image33.png" width="321" height="368" />

-   Select “Go SDK” from the installed directory and click the “OK” button

<img src="./images/go/image34.png" width="442" height="368" />

-   Select “OK”

<img src="./images/go/image35.png" width="573" height="312" />

-   Select the “main. go” file from the go-sample-app project

-   Select “Configure Go Libraries” from the right top

<img src="./images/go/image36.png" width="266" height="445" /> <img src="./images/go/image37.png" width="264" height="443" />

-   For the Global libraries domain, select Go SDK Directory

-   For the Project libraries domain, select the directory where the Go Sample Application is located

-   Click the “OK” button

<img src="./images/go/image38.png" width="571" height="310" />

-   Select “Change module type to Go and reload project” at the top right

<img src="./images/go/image39.png" width="347" height="126" />

-   Select “Reload project”

<img src="./images/go/image40.png" width="560" height="305" />

##### 2.3.3. Connection through VCAP\_SERVICES Environment Setting Information 

To obtain access information for each service to which an application deployed on an open platform is bound, information may be obtained by reading VCAP\_SERVICES environment Setting information registered for each application.

1.)  Application Environment Information of the Open Platform

-   When the service is bound, environment information is registered by the application in the form of JSON.

~~~
{
 "VCAP_SERVICES": {
  "p-rabbitmq": [
   {
    "credentials": {
     "dashboard_url": "https://pivotal-rabbitmq.10.244.0.34.xip.io/#/login/f8b12fcd-df98-4745-a6b2-61f01d20fe24/5lpn72rufsivfgnf3ft4l1p797",
     "hostname": "10.244.9.50",
     "hostnames": [
      "10.244.9.50"
     ],

…..(Skip)…..
     "ssl": true,
     "uri": "amqps://f8b12fcd-df98-4745-a6b2-61f01d20fe24:5lpn72rufsivfgnf3ft4l1p797@10.244.9.50/a1aec425-d1ec-40b7-865f-4eaba371b9a2",
     "uris": [
      "amqps://f8b12fcd-df98-4745-a6b2-61f01d20fe24:5lpn72rufsivfgnf3ft4l1p797@10.244.9.50/a1aec425-d1ec-40b7-865f-4eaba371b9a2"
     ],
     "username": "f8b12fcd-df98-4745-a6b2-61f01d20fe24",
     "vhost": "a1aec425-d1ec-40b7-865f-4eaba371b9a2"
    },
    "label": "p-rabbitmq",
    "name": "rabbitmq-service-instance",
    "plan": "standard",
    "tags": [
     "rabbitmq",
     "messaging",
     "message-queue",
     "amqp",
     "stomp",
     "mqtt",
     "pivotal"
    ]
   }
  ]
…..(Skip)…..
~~~

-   VCAP\_SERVICES Information Structure

2.)  How to extract VCAP\_SERVICES information

-   Extract and return the “uri” information from VCAP\_SERVICE information.

-   Parent Element (VCAP\_SERVICE)
    -   First Element (p-rabbitmq) Service Name Information
    -   Second Element (credentials) Information
    -   Third Element (uri) Information

~~~
func sample_function_name() string {

   args := os.Getenv("VCAP_SERVICES")     //VCAP_SERVICES Element Information (Parent Element)
   var amqp_uri string
   if args != "" {
      var vcap_env map[string]interface{}
      if err := json.Unmarshal([]byte(args), &vcap_env); err != nil {
         log.Panic(err.Error())
      }

      //Service instance (not name) - for example : p-mysql - type check !!! []interface{}
      if vcap_env["p-rabbitmq"] != nil {    //First Element(Service Name) Information
         sub_env := vcap_env["p-rabbitmq"].([]interface{})
         credentials := (sub_env[0].(map[string]interface{}))["credentials"].(map[string]interface{})
//Second Element “credentials” Information
         amqp_uri = credentials["uri"].(string)    //Third Element “uri” Information
      }
   }
   return amqp_uri
}
~~~

-   Refer to VCAP\_SERVICES Structure

##### 2.3.4. Connecting Database, Redis, RabbitMQ  through config.ini file

1.)  config.ini

-   mysql connection information settings

~~~
#Go Sample Web Server port
server.port = 8080
#Mysql DB Info
mysql.dburl=”mysql\_server\_ip”:”mysql\_server\_port”/”database\_name”
mysql.userid=”user\_id”
mysql.userpwd=”user\_password”
mysql.maxconn=”max\_connection\_number”

MongoB Info
mongodb.dburl=”mongodb\_server\_ip”: ”mongodb\_server\_port”/”collection\_name”
mongodb.userid=”user\_id”
mongodb.userpwd=”user\_password”

# Redis Info
redis.addr=”redis\_server\_ip”: ”redis\_server\_port”

# RabbitMQ Info
rabbitmq.user=”rabbitmq\_id”
rabbitmq.pass=”rabbitmq\_password”
rabbitmq.addr=”rabbitmq\_server\_ip”: ”rabbitmq\_server\_port”
~~~

2.)  Connect Sample

~~~
import (
   "bufio"
   "fmt"
   "io"
   "log"
   "os"
   "strconv"
   "strings"

   "net/http"

   "encoding/json"
   "org/openpaas/sample/Godeps/_workspace/src/gopkg.in/redis.v3"

   "org/openpaas/sample/datasource"
   "org/openpaas/sample/handler"
   "org/openpaas/sample/message"
)

type Config map[string]string

func main() {
   // Processing by Datasource - MySql, Cubrid, MongoDB Type
   var dbconfig *datasource.DBConfig
   var mgodbconfig *datasource.MgoDBConfig
   var handlers http.Handler

   fmt.Println("##### Go Sample Application start!!!")
   //============================================
   // bring the dbtype information from the properties.
   dbtype := os.Getenv("dbtype")
   //============================================

   //============================================
   // Sample VCAP_SERVICE INFO Parsing
   os_args := os.Getenv("VCAP_SERVICES")
   readOSEnvironment(os_args)
   //============================================

   //============================================
   // Read basic property settings information
   config, err := ReadConfig(`config.ini`)
   if err != nil {
      fmt.Println(err)
   }
   //============================================

   // Default Redis Client Create - For Login & Logout
   redis_client := initRedis(config["redis.addr"])
   defer redis_client.Close()

   // default dbytype = "mysql"
   if dbtype == "" {
      dbtype = "mysql"
   }

   // RabbitMQ
   //endpoint := "amqps://929b5a98-6521-4da5-86c1-0fcaa2450a86:55dgq7ocf1gtiamqum0hb9p8be@10.244.9.50:5671/3c14abdc-c922-428c-af70-d61eb91ef818"
   endpoint := readOSEnvironment_mq()
   fmt.Println("########## credentials-uri:", endpoint)
   mq := message.NewRabbitMQ(endpoint)
   if mq != nil {
      defer message.CloseMQ(mq)
   }

   // DB Initialize -
   // Process Connection by DB Type
   if dbtype == "mysql" {
      maxConnection, err := strconv.Atoi(config["mysql.maxconn"])
      if err != nil {
         log.Println(err)
         os.Exit(-1)
      }

      dbconfig = datasource.NewDBConfig(config["mysql.userid"], config["mysql.userpwd"], config["mysql.dburl"], maxConnection)
      defer dbconfig.CloseDb()

      fmt.Println("dbmap:", dbconfig.DBMAP)
      if dbconfig.DBMAP == nil {
         log.Panic("Couldn't create Database Connection properly - MySQL!!!")
         os.Exit(-1)
      }

      // Connect Route Path information with processing services
      handlers = handler.NewHandler(dbconfig.DBMAP, redis_client, mq.Ch)

   } else if dbtype == "mongodb" {
      mgodbconfig = datasource.NewMgoDBConfig(config["mongodb.userid"], config["mongodb.userpwd"], config["mongodb.dburl"])
      defer mgodbconfig.CloseDb()

      fmt.Println("session:", mgodbconfig.SESSION)
      if mgodbconfig.SESSION == nil {
         log.Panic("Couldn't create Database Connection properly - MongoDB!!!")
         os.Exit(-1)
      }

      // Connect Route Path information with processing services
      handlers = handler.NewMgoHandler(mgodbconfig.SESSION, redis_client, mq.Ch)

   } else {
      fmt.Println("No database type found.")
      os.Exit(-1)
   }

   if err := http.ListenAndServe(fmt.Sprintf(":%v", config["server.port"]), handlers); err != nil {
      log.Fatalln(err)
   }
}

/*
   Read Config file
*/
func ReadConfig(filename string) (Config, error) {
   // init with some bogus data
   config := Config{
      "server.ip":     "127.0.0.1",
      "server.port":   "8080",
      "mysql.dburl":   "",
      "mysql.userid":  "",
      "mysql.userpwd": "",
      "mysql.maxconn": "",
   }

   if len(filename) == 0 {
      return config, nil
   }
   file, err := os.Open(filename)
   if err != nil {
      return nil, err
   }
   defer file.Close()

   reader := bufio.NewReader(file)
   for {
      line, err := reader.ReadString('\n')
      // check if the line has = sign
      // and process the line. Ignore the rest.
      if equal := strings.Index(line, "="); equal >= 0 {
         if key := strings.TrimSpace(line[:equal]); len(key) > 0 {
            value := ""
            if len(line) > equal {
               value = strings.TrimSpace(line[equal+1:])
            }
            // assign the config map
            config[key] = value
         }
      }
      if err == io.EOF {
         break
      }
      if err != nil {
         return nil, err
      }
   }
   return config, nil
}


func initRedis(addr string) *redis.Client {
   client := redis.NewClient(&redis.Options{
      Addr: addr,
   })
   return client
}

~~~

##### 2.3.5. Connect GlusterFS

1.)  File Upload

-   Read the file requested from the client and send it to the ClusterFS Server

~~~
func (h *GlusterFSService) Upload(w http.ResponseWriter, r *http.Request) {
   if err := r.ParseMultipartForm(MAX_MEMORY); err != nil {
      log.Println(err)
      http.Error(w, err.Error(), http.StatusForbidden)
   }
   index := 0
   path := make([]string, len(r.MultipartForm.File))
   for _, fileHeaders := range r.MultipartForm.File {
      for _, fileHeader := range fileHeaders {
         file, _ := fileHeader.Open()
         buf, _ := ioutil.ReadAll(file)
         thumb_img_path, err := ObjectPut(buf, fileHeader.Filename)
         path[index] = "{\"thumb_img_path\":\"" + thumb_img_path + "\"}"
         if err != nil {
            log.Fatalln("!!! ObjectPut error:", err)
         }
      }
      index++
   }
   js, err := json.Marshal(path)
   if err != nil {
      log.Fatalln("Error writing JSON:", err)
   }
   w.Header().Set("Content-Type", "application/json")
   w.WriteHeader(http.StatusOK)
   w.Write([]byte(js))
   return
}
~~~

2.)  Connect GlusterFS Server and send file

-   Connect to GlusterFS Server and send file

~~~
func ObjectPut(buf []byte, fileName string) (string, error) {
   userName, password, authUrl, tenantName := readOSEnvironment_gf()
   c := swift.Connection{
      UserName: userName,
      ApiKey:   password,
      AuthUrl:  authUrl,
      Tenant:   tenantName,
   }
   // Authenticate
   err := c.Authenticate()
   if err != nil {
      panic(err)
   }
   // Container
   _, _, err = c.Container(CONTAINER_NAME)
   if err != nil {
      var meta = swift.Headers{"X-Container-Read": ".r:*"}
      err = c.ContainerCreate(CONTAINER_NAME, meta)
      if err != nil {
         panic(err)
      }
   }
   // Put object
   headers := swift.Headers{}
   headers["Content-Length"] = strconv.FormatInt(int64(len(buf)), 10)
   err = c.ObjectPutBytes(CONTAINER_NAME, fileName, buf, "application/octet-stream")
   if err != nil {
      log.Fatalln("!!! ObjectPut error:", err)
   }
   // Fetch object info and compare
   info, _, err := c.Object(CONTAINER_NAME, fileName)
   if err != nil {
      log.Println(err)
   }
   if info.Bytes != int64(len(buf)) {
      log.Println("Bad length")
   }
   thumb_img_path := c.StorageUrl + "/" + CONTAINER_NAME + "/" + fileName
   return thumb_img_path, err
}
~~~

3.)  GlusterFS Connection information

-   Extract information from VCAP\_SERVICES

~~~
func readOSEnvironment_gf() (string, string, string, string) {
   args := os.Getenv("VCAP_SERVICES")
   var userName, password, authUrl, tenantName string
   if args != "" {
      var vcap_env map[string]interface{}
      if err := json.Unmarshal([]byte(args), &vcap_env); err != nil {
         log.Panic(err.Error())
      }
      //Service instance (not name) - for example : p-mysql - type check !!! []interface{}
      if vcap_env["glusterfs"] != nil {
         sub_env := vcap_env["glusterfs"].([]interface{})
         credentials := (sub_env[0].(map[string]interface{}))["credentials"].(map[string]interface{})
         userName = credentials["username"].(string)
         password = credentials["password"].(string)
         authUrl = credentials["auth_url"].(string)
         tenantName = credentials["tenantname"].(string)
      }
   }
   return userName, password, authUrl, tenantName
}
~~~

#### 2.4. Deployment

-   Deploy Go Sample Application using the cf cli command.

-   Proceed under the assumption that cf cli is installed and cf login is already done.

~~~
### Go Sample Application Deployment
$ cd “Go Sample Application” directory
$ source .envrc                 # To set GO_PATH 

$ cd src/org/openpaas/sample

$ cf push –f manifest.yml        # Deploy Go Sample Application
Using manifest file manifest.yml
Creating app go-sample in org org / space space as admin...
OK
Using route go-sample.os.paasxpert.com
Binding go-sample.os.paasxpert.com to go-sample...
OK
Uploading go-sample...
Uploading app files from: /home/ihocho/git/OpenPaaSSample/GoSample/go-sample-app/src/org/openpaas/sample
Uploading 854.2K, 185 files
Done uploading               
OK
Binding service rabbitmq-service-instance to app go-sample in org org / space space as admin...
OK
Starting app go-sample in org org / space space as admin...
Creating container
Successfully created container
Downloading app package...
Downloaded app package (321.1K)
Downloading buildpacks (https://github.com/cloudfoundry/go-buildpack.git)...
Downloaded buildpacks
Staging...
-------> Buildpack version 1.7.0
https://pivotal-buildpacks.s3.amazonaws.com/concourse-binaries/godep/godep-v17-linux-x64.tgz
-----> Checking Godeps/Godeps.json file.
-----> Installing go1.4.2... done
Downloaded [https://storage.googleapis.com/golang/go1.4.2.linux-amd64.tar.gz]
-----> Running: godep go install -tags cloudfoundry ./...
Exit status 0
Staging complete
Uploading droplet, build artifacts cache...
Uploading droplet...
Uploading build artifacts cache...
Uploaded build artifacts cache (60M)
Uploaded droplet (2.8M)
Uploading complete
1 of 1 instances running
App started
OK
App go-sample was started using this command `sample`
Showing health and status for app go-sample in org org / space space as admin...
OK
requested state: started
instances: 1/1
usage: 256M x 1 instances
urls: go-sample.os.paasxpert.com
last uploaded: Mon Dec 14 04:39:52 UTC 2015
stack: cflinuxfs2
buildpack: https://github.com/cloudfoundry/go-buildpack.git
     state     since                    cpu    memory         disk      details   
#0   running   2015-12-14 01:41:20 PM   0.0%   2.1M of 256M   0 of 1G      
~~~

Since the data management of the Go Sample Application is done by either MySQL or MongoDB. Set database name(mysql or mongodb) at the environment setting information(dbtype) when deploying.

#### 2.5. Test

The test can be done directly at the command window using the curl command.

~~~
### Check all Organizations information
curl -v http://”depolyed_sample_app_name”/orgs

### Check the specified Organization's information
curl -v http:// ”depolyed_sample_app_name”/orgs/{org_id}

### Create Organization
curl -X POST -v 
–d '{"id":”org_id”,"label":"org_lable_info","desc":"org_description_info","url":"org_uri_info"}' 
http:// ”depolyed_sample_app_name”/orgs

### Modify Organization
curl -X PUT -v
-d '{"label":" org_lable_info ","desc":"org_description_info","url":"org_url_info"}'
http:// ”depolyed_sample_app_name”/orgs/{org_id}

### Delete Organization
curl -X DELETE -v http:// ”depolyed_sample_app_name”/orgs/{org_id}
~~~


### [Index](https://github.com/PaaS-TA/Guide-eng/blob/master/README.md) > [AP User Guide](../README.md) > Go Development
