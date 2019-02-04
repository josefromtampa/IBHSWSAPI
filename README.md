# IBHSWSAPI 
IBHS Web Services API for accessing various features of Focus.

## IBHS Web Service API (Rest/JSON)
IBHS Web Services API for accessing various features of Focus.  This web services represents a variety of Focus features, including:
  - Create an evaluation
  - Update a created evaluation
  - Retrieve created evaluations
  - Retrieve all questions
  - Create/Update answers for an evaluation
  - Retrieve answers to an evaluation
  - Create/Update attachments to answers
  - Retrieve attachament to answers
`
Samples of request/response for the web service are available at [PostMan API Docs](https://documenter.getpostman.com/view/9267/RztmsUpJ).

## Getting Started

The IBHS Web Service API service operation definition and reques/response pairs can be found in [PostMan API Docs](https://documenter.getpostman.com/view/9267/RztmsUpJ).


### Initial Access

The IBHS Web Service is Restful (REST vs SOAP) and leverages the more seccint and readable data format of JSON (JSON vs XML). To access any of the service operations, a security token must be retrieved.  To get a security Token, the getSecurityToken(UserID, Pwd) must be called.
The security token is time-sensitive and is initially set to 7 days.  The value that is returned when calling getSecurityToken includes:
  - SecurityToken - Used for calling the service's operations.
  - SecurityExpiration - Date when the security token expires.
Call getSecurityToken to refresh the Security Token and extend the expiration.

### Base Models

The main models around the Focus Platform:
  - Evaluation
    - Application
    - PropertyAddress
    - PropertyDetails
    - HomeOwner
  - Questions
  - Answsers
  - Attachments

### Understanding Focus Collections

The Focus system uses collections to define specific attributes such as HomeProgramType (Hurricane, High Wind), HomeCateogryType (New Home, Existing Home), DesignationTypes (Gold, Silver, Bronze), etc.  
The IBHS Web services allow for the retrieval of all defined collections along with the retrieval of the key/values associated with each collection (view Postman Documention for more).

### Understanding Questions

#### Understanding Question Reponses

#### Questions: When to show
#### Questions: When to use dependentShowData
#### Questions: When to use MultiOccGroup
### Understanding Answers
#### Type: Is a specific response required?
#### Answer Model: Question ID, Answer Response, Answer Sequence
### Understand Answer Attachments

