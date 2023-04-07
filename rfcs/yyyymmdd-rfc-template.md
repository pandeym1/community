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
