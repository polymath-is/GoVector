GoVector
========

### Overview
This is a small library that you can add to your Go project to
generate a [ShiViz](http://bestchai.bitbucket.org/shiviz/)-compatible
vector-clock timestamped log of events in your distributed system.

PLEASE NOTE: GoVec is compatible with Go 1.4 + 

* govec/: Contains the Library and all its dependencies
* test.go: A small program to test the library
* example-Log.txt: An example log generated by the library

### Index

type GoLog
   * func Initialize(string ProcessName, string LogName) *GoLog
   * func InitializeMutipleExecutions(string ProcessName, string LogName) *GoLog
   * func PrepareSend(string LogMessage, byte[] b) (byte[] output)
   * func UnpackReceive(string LogMessage, byte[] b) (byte[] output)
   * func LogLocalEvent(string LogMessage)
   * func DisableLogging()
   

### Usage

Add the govec folder to your project and import it with:

	"import ./govec"


####   type GoLog

	type GoLog struct{
		// contains filtered or unexported fields
	}

 The GoLog struct provides an interface to creating and maintaining vector timestamp entries in the generated log file
 
#####   func Initialize

	func Initialize(string ProcessName, string LogName) *GoLog

Returns a Go Log Struct taking in two arguments and truncates previous logs:
* MyProcessName (string): local process name; must be unique in your distributed system.
* LogFileName (string) : name of the log file that will store info. Any old log with the same name will be truncated


#####   func InitializeMutlipleExecutions
	
	func Initialize(string ProcessName, string LogName) *GoLog

Returns a Go Log Struct taking in two arguments without truncating previous log entry:
* MyProcessName (string): local process name; must be unique in your distributed system.
* LogFileName (string) : name of the log file that will store info. Each run will append to log file seperated 
by "=== Execution # ==="

#####   func PrepareSend
	
	func PrepareSend(string LogMessage, byte[] b) (byte[] output)

Logs LogMessage and vector timestamp into log file, and packs incoming byte[] b and vector timestamp as a GOB. Returns a byte[] output that can be sent on wire.

#####   func UnpackReceive
	
	func UnpackReceive(string LogMessage, byte[] b) (byte[] output)
	
Unpacks incoming byte[] b from GOB and logs LogMessage with received vector timestamp. Returns byte[] output which can be used for further processing by the program.

#####   func LogLocalMessage

	func LogLocalEvent(string LogMessage)
	
Increments current vector timestamp and logs it into Log File. 

#####   func DisableLogging

	func DisableLogging()
	
Disables Logging. Log messages will not appear in Log file any longer.
Note: For the moment, the vector clocks are going to continue being updated.

###   Examples

The following is a basic example of how this library can be used 

	package main

	import "./govec"

	func main() {
		Logger := govec.Initialize("MyProcess", "LogFile")
		
		//In Sending Process
		
		//Prepare a Message
		messagepayload := []byte("samplepayload")
		finalsend := Logger.PrepareSend("Sending Message", messagepayload)
		
		//send message
		//connection.Write(finalsend)

		//In Receiving Process
		
		//receive message
		recbuf := Logger.UnpackReceive("Receiving Message", finalsend)

		//Can be called at any point 
		Logger.LogLocalEvent("Example Complete")
		
		Logger.DisableLogging()
		//No further events will be written to log file
	}

This produces the log "LogFile.txt" :

	MyProcess {"MyProcess":1}
	Initialization Complete
	MyProcess {"MyProcess":2}
	Sending Message
	MyProcess {"MyProcess":3}
	Receiving Message
	MyProcess {"MyProcess":4}
	Example Complete

### VectorBroker

type VectorBroker
   * func Init(logfilename string, pubport string, subport string)

### Usage

    A simple standalone program can be found in server/broker/runbroker.go 
    which will setup a broker with command line parameters.
   	Usage is: 
    "go run ./runbroker (-logpath logpath) -pubport pubport -subport subport"

    Tests can be run via GoVector/test/broker_test.go and "go test" with the 
    Go-Check package (https://labix.org/gocheck). To get this package use 
    "go get gopkg.in/check.v1".
    
Detailed Setup:
Step 1:
    Create a Global Variable of type brokervec.VectorBroker and Initialize 
    it like this =

    broker.Init(logpath, pubport, subport)
    
    Where:
    - the logpath is the path and name of the log file you want created, or 
    "" if no log file is wanted. E.g. "C:/temp/test" will result in the file 
    "C:/temp/test-log.txt" being created.
    - the pubport is the port you want to be open for publishers to send
    messages to the broker.
    - the subport is the port you want to be open for subscribers to receive 
    messages from the broker.

Step 2:
    Setup your GoVec so that the realtime boolean is set to true and the correct
    brokeraddr and brokerpubport values are set in the Initialize method you
    intend to use.

Step 3 (optional):
    Setup a Subscriber to connect to the broker via a WebSocket over the correct
    subport. For example, setup a web browser running JavaScript to connect and
    display messages as they are received. Make RPC calls by sending a JSON 
    object of the form:
            var msg = {
            method: "SubManager.AddFilter", 
            params: [{"Nonce":nonce, "Regex":regex}], 
            id: 0
            }
            var text = JSON.stringify(msg)

####   RPC Calls

    Publisher RPC calls are made automatically from the GoVec library if the 
    broker is enabled.
    
    Subscriber RPC calls:
    * AddNetworkFilter(nonce string, reply *string)
        Filters messages so that only network messages are sent to the 
        subscriber.      
    * RemoveNetworkFilter(nonce string, reply *string)
        Filters messages so that both network and local messages are sent to the 
        subscriber.
    * SendOldMessages(nonce string, reply *string)
        Sends any messages received before the requesting subscriber subscribed.
 
