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
Each evaluation contains a collection of answers to questions.  The list of all questions (for a given program type) can be retrieved via the web service, as well as a single questions (given a progam type and question Id).  The question have serveral characteristics that are important to note.

The service, when all questions are retrieved, will show a complete and full set of all questions.  It's important to note that:
```
   {  "SectionTypeId": 1,
      "SectionDescription": "General/Site Information",
      "QuestionDescription": "Wind Speed (ASCE 7-05)",
      "SortOrder": 1,
      "SortOrderInSection": 13,
      "QuestionId": 17,
      "QuestionCode": "wind705",
      "MinAttachments": 0,
      "MaxAttachments": 0,
      "MemberOfDependentShowGroup": null,
      "MultiOccuranceQuestionGroup": null,
      "DependentShowGroup": null,
      "Include": true,
      "ForDesignationType": "Gold Silver Bronze",
      "MinDesignationType": 1,
      "QuestionTypeId": 4
    }  
```
Questions include attributes (that don't change):

  -"QuestionDescription": Question verbiage shown to user
  -"QuestionId": permenant question ID 
  -"QuestionCode": permenant question code
  -"QuestionTypeId": Type of input for the question (see Collections section for EvalQuestionType [4=Number]).
  
#### Questions: When to show, sort order and attachment requirements
If this flag=true, then the question is included in this Program Type (homeProgramTypeId).  This value does not change for the given program type under any conditions:

  -"Include": true,
  
Question are required for Gold (DesignationTypeId=3) encompass all questions associated with a particular Program type (homeProgamTypeId). Silver includes a subset of Gold questions.  There are no Silver-Only question -- Silver questions do also apply to Gold, so if a question is asked for Silver, it will be asked for Gold.  Bronze is the same way -- Bronze is a subset of Silver.

  -"ForDesignationType": Can be "Bronze", "Silver Bronze", or "Gold Silver Bronze"
  -"MinDesignationType": where 1="Gold Silver Bronze", 2="Gold Silver" 3="Gold"

The order in which IBHS's application "Focus" show questions follows a Sort Order of the Section and Sort order of the question in the section.  Sorting should be "SortOrder, SortOrderInSection":

  -"SortOrder": Sort order for the section (Questions belong to sections, sections own this sort order)
  -"SortOrderInSection": Sorting of the questions in the current section.

Questions could have an image/document requirement.  When both Min and Max are 0, no attachment is required.

  -"MinAttachments": 0,
  -"MaxAttachments": 0,
  
#### Questions: When to use dependentShowData
Question where [MemberOfDependentShowGroup == null] can be shown, while those that have a value in this field are "Conditionally Rendered".  Typically, another question's answer will trigger for all questions with a particular MemberOfDependentShowGroup name to appear.

  -"MemberOfDependentShowGroup": null
  
In order for the dependent question to show, another question's answer must be set to a particular value.  This is configured in the DependentShowGroup field.

  -"DependentShowGroup": Data mapping for "Answer : Dependent Group Name"

For example, question 189's DependendShowData consist of the following:

```
  {
    "depResponse": "Yes",
    "showGroup": "ventmnt",
    "showSection": "",
    "showStopper": "",
    "cntGrp": "",
    "fvaudit": ""
  },
  {
    "depResponse": "mounttemp",
    "showGroup": "RoofVentType",
    "showSection": "",
    "showStopper": "",
    "cntGrp": "",
    "fvaudit": "a1"
  },
  {
    "depResponse": "ridgetemp",
    "showGroup": "rdgevnt",
    "showSection": "",
    "showStopper": "",
    "cntGrp": "",
    "fvaudit": "a1"
  },
  {
    "depResponse": "offridgetemp",
    "showGroup": "ofrgevnt",
    "showSection": "",
    "showStopper": "",
    "cntGrp": "",
    "fvaudit": "a1"
  },
  {
    "depResponse": "bothtemp",
    "showGroup": "bothvnt",
    "showSection": "",
    "showStopper": "",
    "cntGrp": "",
    "fvaudit": "a1"
  }
  ```
  note the ShowGroup determines the group of questions to show for the a given depResponse (which is the answer).
#### Questions: When to use MultiOccGroup
  Another aspect of questions are the ability to collect a particular section several times over.  For example, when asked "How many roof covering", you could capture answers to the same set of questions for each roof covering type.  These type of questions are referred to as "Multi-Occurrance" Questions.  In the "DependentShowData" field, a question might have.
  
  For example, question 203 has:
  
  ```
  [{
      "depResponse":"multiCntTag",
      "showGroup":"","showSection":"",
      "showStopper":"",
      "cntGrp":"OffRidge",
      "fvaudit":"" 
  }]  
  ```

  -"MultiOccuranceQuestionGroup": null,

In this example, show all questions where MutliOccuranceQuestionGroup="OffRidge".

#### Understanding Question Reponses
If a question requires a specific answer from, say, a drop-down or Yes/No, True/False, etc., the possible values for a given question Id are available via two different service calls:
##### All at once (Bulk)
For situations where the bulk of all responses for all questions can be held, then looked up as needed, the bulk service call will save on repetitive "one-at-a-time" calls.

##### One at a time
A separate call is available for situations where it's more feasible to go after the possible responses to a question "on the fly".


### Understanding Answers
#### Type: Is a specific response required?
#### Answer Model: Question ID, Answer Response, Answer Sequence
### Understand Answer Attachments

