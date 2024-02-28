# Microservice: Celsius to Fahrenheit 
This is a short guide on how to request and receive temperature conversion data from the microservice implemented for the weather app. 

## How to programmatically REQUEST data from the microservice.

**Step 1: Connect to the microservice**
To begin, import the ZeroMQ library with `import zmq`, then initialize a variable using the `zmq.Context()` method to establish a context for the socket. After that is created, use `.socket(zmq.REQ)` method on that newly defined context variable to create a socket variable, using `zmq.REQ` as the parameter  to define this socket as a **REQUEST** socket. Finally, use that socket variable and use the connect method  `.connect("tcp://localhost:4444")` with the microservice host location `tcp://localhost:4444`, which is the port that the reply server is bound to

**Step 2: Format the data request**
Despite being a temperature conversion, the reply server is set up to receive the data as a string, which is promptly typecasts it to an integer as a string. If any data sent over in the request is non-numeric it will not be able to properly convert it.  Simply send the request containing *only* the number of the temperature. e.g `10`. If the microservice recieves something like `10°C` or `10C` or `10°` , it may cause the microservice to crash due to typecasting the string to an int for conversion.

**Step 3: Request the data**
Using your socket variable, use the `.send_string()` method to submit the data request to the microservice, using your data as the parameter.

**Sample request**

    import zmq
    
    #Connect to microservice socket
    context = zmq.Context()
    socket = context.socket(zmq.REQ)
    socket.connect("tcp://localhost:4444")
	
	#Format data
	temperature = 10
	
	#Send the request to the microservice
	socket.send_string(f"{temperature}")
	
    
## How to programmatically RECIEVE data from the microservice

**Step 1: Prepare for the response**
Once a request is made, the server recieve the request, process the data, and send its reply through the established socket. Set up a new variable, and use the .recv_string() method on your socket variable. This will have the program wait and listen for the reply

**Step 2: Recieve and process the data**
The server will send data back in the same manner it recieved it. The response will be a string, containing only the number. So for example, if a request was sent for 10°C, the response  will be recieved as a string `50`, indicating the conversion for 10°C is 50°F. From there it will be simple to typecast the response to an int (or float) for the program to use


**Sample receive**

    fahrenheit = socket.recv_string()
    print(f"{fahrenheit}")



## UML diagram

```mermaid
sequenceDiagram
participant WeatherApp
participant ZeroMQ Socket
participant Microservice
activate WeatherApp
Note over WeatherApp: Format data correctly for microservice
WeatherApp->>ZeroMQ Socket: send_string(data)
activate ZeroMQ Socket
ZeroMQ Socket->>Microservice: socket.rcv_string()
activate Microservice
Note over Microservice: Typecast data for conversion
Microservice->>Microservice: celsius_to_fahrenheit()
Microservice->>ZeroMQ Socket: socket.send_string(data)
deactivate Microservice
ZeroMQ Socket->>WeatherApp: rcv_string()
deactivate ZeroMQ Socket
Note over WeatherApp: Process recieved data
deactivate WeatherApp
