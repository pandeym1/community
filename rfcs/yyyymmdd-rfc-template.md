# Weather Image Fetch Protocol

| Status        | (Proposed)                                           |
:-------------- |:---------------------------------------------------- |
| **RFC #**     | ---------------------------------------------------- |
| **Author(s)** | Manas Pandey                                         |
| **UFID**      | 39151151                                             |
| **Updated**   | 2023-04-07                                           |

## Objective

The goal of Weather Image Fetch Protocol (WIFP) is to allow users to pull an image from a server that displays information about the weather. This image can be modified based on client input to access information for different regions, change measurement systems, and change image icons. 

## Motivation

While there are many APIs in place to fetch weather data, it is still a challenge to parse, format, and display the information in a meaningful way. Programmers may want to modify weather information by themselves to have a customizable application, but in the event where weather information makes up a small portion of an application it may seem like a waste of time to invest in a custom display for it. 

WIFP is an application layer protocol that solves this problem by allowing users to connect to a server and easily generate a boiler plate display for the weather. 

## User Benefit

The users will benefit by not having to reinvent the wheel in the event they need a physical display of weather in their application. With updates, the high degree of customizability that the protocol offers to users will make it easier to have a boiler plate display early in development that can be swapped out if a more detailed display is needed.

## Design Proposal

The server is responsible for fetching information about weather and interpreting the fetched data (compute time, date, temperature measurement). When a client interacts with the server they can specify what region they want to fetch the weather image of. This step lets the server configure the url it will use to scrape information from the internet using the OpenWeater API. After this information is fetched and the user request for an image to be generated, the server starts processing the information it had retrieved. The processing step consists of converting the temperature into the correct format (Celsius or Fahrenheit) that is decided by the client. The server also processes the time it receives from the internet and generates the date which consists of month, day, year, time zone, and time in UTC. This generated date is used to decide whether it is day or night and change the icons for the image it is about to generate accordingly. In the final step of processing the server creates a 350x350 pixel image and transfers it to the client via a byte stream.

As stated before the client is just in charge of issuing commands and accepting the server response. When the server generates an image the client saves it by converting the byte stream to an image on its end. The client should be able to make some modifications to the image based on its preferences. 

The connection between the client and the server is possible by the use of a TCP socket. The purpose of this protocol being client server based as opposed to an API is so many clients can connect at once (will come in later update) and have the server handle the heavy computing for them. There is also a set of instructions that this protocol follows on the server side that make it different from an API.

### Implementation
The actual implementation will be explained first starting from the server side. The first thing that a server does after setting up its printer writer, buffered reader, data output stream, and data input stream is wait to hear a response from the client using the buffered reader. This is the parameter of a while loop, and the while loop will keep repeating until the buffered reader gets a null value (client disconnects). Based on the client response the server will perform four commands which are configure, change units, generate, and quit. The simplest command is quit, which makes the server use its print writer to tell the client to terminate itself through a message. If configure is selected the server will wait for a response from the client using its buffered reader. This response should be given in the format of a string only and contain the city and the initials (not capitalized) of the country that the user wants to fetch weather data from (EX: London,uk). It saves this value in a string to use when the command generate is called. If the change units command is received from the client, the server multiplies an int value by -1. This int value represents what units of measurement the temperature will be in the generate command. Negative means Celsius and positive means Fahrenheit. The biggest and most complex command here is the generate command. Upon receiving this command the server will create a string to send a request to the Open Weather API. Based on the users input the string value will change. The URL feature of java is used to fetch a response from the API. Upon getting a response a string builder is used and the response from the API is converted from JSON to a single string. This is done because Java does not have any built in commands to convert JSON into map. This string is passed into a function that will generate a Map<String, Object> based on the JSON values. Inside of this function regex is used to parse the string formatted in JSON to a map. From here the server can access all the data given by the API by the map that was generated. The server then takes information like the name, temperature, humidity, date, and date in hours to and prints it out on the server side. This printing is mostly for debugging purposes. From here, the server is responsible for converting data like date and temperature to be readable since the API has it formatted inappropriately. The first conversion that is performed is converting the date that was given in the format of a double from the API to a workable date on the server. This was done by casting the double to a long value, multiplying it by 1000, and finally converting it to a Date using java’s built Date class. The next processing that is performed is to convert the temperature from Celsius or Fahrenheit using the relevant formulas. Conversion for this step works the same way as the previous step. Finally the value of name and humidity it type casted to a string since the API only returns objects with the current server implementation. The last step of the generate command is to generate an image and convert it to a byte stream to send to the client. All the possible icons and backgrounds were imported to the server when it was initiated. Depending on the time and weather description the relevant icon is fetched. Depending on the time the relevant background is fetched (uses hours from date to figure this out). It is placed upon a new buffered image of size 350x350 pixels using java’s graphics library. After all items have been placed onto the buffered image, FileOutputStream and ImageIO converts this to a png image file on the server side. File input stream reads in this image, converts it to bytes, and uses the data output stream to send the bytes to the client. 

The client side is much less process heavy than the server side. After setting up the same types of streams as the server and connecting to it the client enter a while loop that will continue until it is told to stop by a certain command. From here the client waits for input from the user and after getting it sends it to the server. The commands the client can perform are configure, change units, generate, and quit. If the quit command is send to the server the client will receive a quit response from the server, which will make the client exit the while loop and close its socket. If the client decided to change units, the request is sent to the server to do so. If the client decides to generate a weather image a request is sent to the image to do so. After sending that specific request the client reads in the bytes from the server and uses a FileOutputStream to generate an image on its own side. If the configure command is selects the client is prompted by to accept another input from the print writer. This input is sent to the server.   


### Performance Implications
* Do you expect any (speed / memory)? How will you confirm?
* There should be microbenchmarks. Are there?
* There should be end-to-end tests and benchmarks. If there are not (since this is still a design), how will you track that these will be created?

### Dependencies
* Dependencies: does this proposal add any new dependencies to TensorFlow?
* Dependent projects: are there other areas of TensorFlow or things that use TensorFlow (TFX/pipelines, TensorBoard, etc.) that this affects? How have you identified these dependencies and are you sure they are complete? If there are dependencies, how are you managing those changes?

### Engineering Impact
* Do you expect changes to binary size / startup time / build time / test times?
* Who will maintain this code? Is this code in its own buildable unit? Can this code be tested in its own? Is visibility suitably restricted to only a small API surface for others to use?

### Platforms and Environments
* Platforms: does this work on all platforms supported by TensorFlow? If not, why is that ok? Will it work on embedded/mobile? Does it impact automatic code generation or mobile stripping tooling? Will it work with transformation tools?
* Execution environments (Cloud services, accelerator hardware): what impact do you expect and how will you confirm?

### Best Practices
* Does this proposal change best practices for some aspect of using/developing TensorFlow? How will these changes be communicated/enforced?

### Tutorials and Examples
* If design changes existing API or creates new ones, the design owner should create end-to-end examples (ideally, a tutorial) which reflects how new feature will be used. Some things to consider related to the tutorial:
    - The minimum requirements for this are to consider how this would be used in a Keras-based workflow, as well as a non-Keras (low-level) workflow. If either isn’t applicable, explain why.
    - It should show the usage of the new feature in an end to end example (from data reading to serving, if applicable). Many new features have unexpected effects in parts far away from the place of change that can be found by running through an end-to-end example. TFX [Examples](https://github.com/tensorflow/tfx/tree/master/tfx/examples) have historically been good in identifying such unexpected side-effects and are as such one recommended path for testing things end-to-end.
    - This should be written as if it is documentation of the new feature, i.e., consumable by a user, not a TensorFlow developer. 
    - The code does not need to work (since the feature is not implemented yet) but the expectation is that the code does work before the feature can be merged. 

### Compatibility
* Does the design conform to the backwards & forwards compatibility [requirements](https://www.tensorflow.org/programmers_guide/version_compat)?
* How will this proposal interact with other parts of the TensorFlow Ecosystem?
    - How will it work with TFLite?
    - How will it work with distribution strategies?
    - How will it interact with tf.function?
    - Will this work on GPU/TPU?
    - How will it serialize to a SavedModel?

### User Impact
* What are the user-facing changes? How will this feature be rolled out?

## Detailed Design

This section is optional. Elaborate on details if they’re important to
understanding the design, but would make it hard to read the proposal section
above.

## Questions and Discussion Topics

Seed this with open questions you require feedback on from the RFC process.
