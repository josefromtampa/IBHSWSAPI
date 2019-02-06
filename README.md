# IBHS Web Services API
IBHS Web Services API for accessing various features of Focus.  Access to this service requires a security token only accessible with a valid User ID and password.

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
  
Note: Any evaluation can be accessed by it's unique Fortified Id number (string).

All responses include a **expiration date** of the Security Token along with an array of any **errors[]** encountered.

### Creating an Evaluation
The web service has a CreateEvaluation operation that can be called with the appropriate JSON data. 
  This operation does require a unique street/city/state along with Home Owner inforamtion.  
  In order for the evaluation to be seen by a specific evaluator, an evaluator Id must be used in the **request.application.EvaluatorId** field. If an invalid evaluator Id is used, the evaluation will be parked in a temporary area and can still be claimed by any evaluator -- but must have the Fortified Id in order to claim.  (The process of claiming in currently only available in the IBHS Focus application).

This operation, when successful, will return the key identifier the evaluation -- the Fortified Id.  

See Postman [Create an Evaluation](https://documenter.getpostman.com/view/9267/RztmsUpJ#523391be-00fa-4787-aa7b-fb7e47252d5a) for examples.

### Updating an Evaluation
The web service can perform an update to an evaluation.  Much like the CreateEvaluation, it uses the same data model, but includes a necessary FortifiedId field for identifing the evaluation. ( 

Not all sections need to be included in the update -- only section that have changed.  Any answer that is null or whose answer is 0 (and should be, like DesignationTypeId) will not be replaced.

See Postman [Updating an Evaluation](https://documenter.getpostman.com/view/9267/RztmsUpJ#f132d3fa-eac6-479a-97e8-3b4e19cec4fb) for examples.

### Retrieving an Evaluation
The web service can retrieve the information associated with a given FID and will tyically include additional data in the response:
Example:
```
{
  "propertyDetails": {
    "Latitude": "-40.498",
    "Longitude": "82.2323",
    "BuilderName": "Mr. Builder",
    "HomeCategoryId": 1,
    "Remodeled": true,
    "RemodeledDescription": "New Attached Structure",
    "YearBuilt": 2014,
    "RoofAge": 4,
    "NewRoof": true
  },
  "propertyAddress": {
    "Address1": "12101 Beach Dr.",
    "Address2": "",
    "City": "Tampa",
    "StateAbbr": "FL",
    "ZipCode": "33611"
  },
  "mailingAddress": {
    "Address1": "3210 First St.",
    "Address2": "",
    "City": "Tampa",
    "StateAbbr": "GA",
    "ZipCode": "33612"
  },
  "homeOwner": {
    "Salutation": "Mr.",
    "Email": "sample@email.com",
    "FirstName": "John",
    "LastName": "Smith",
    "Phone": "1212324343"
  },
  "application": {
    "DesignationTypeId": 1,
    "HomeProgramTypeId": 1,
    "ProjectLabel": "NCIUA",
    "EvaluationDate": "2019-01-14T23:43:24.427",
    "EvaluatorId": 14
  },
  "EvaluationId": 33099,
  "FortifiedId": "FEH3361120190D2DE44",
  "HomeCategoryId": 1,
  "HomeCategory": "New Construction",
  "HomeProgramTypeId": 1,
  "HomeProgram": "FORTIFIED Home:Hurricane",
  "DesignationTypeId": 1,
  "Designation": "FORTIFIED Bronze",
  "Status": "Evaluation In Progress",
  "StatusId": 8,
  "EvaluatorId": 14,
  "Evaluator_Firstname": "Jose",
  "Evaluator_LastName": "Jimenez",
  "EvaluationExpireDate": null,
  "CreatedDate": "2019-02-03T21:14:19.207",
  "ModifiedDate": null,
  "DateSubmit_Started": null,
  "DateSubmit_Ended_AuditBegins": null,
  "Date_Designated": null,
  "Errors": [],
  "SecurityTokenInvalidAfter": "2019-02-09T00:00:00"
}
```
See Postman [Retrieving an Evaluation](https://documenter.getpostman.com/view/9267/RztmsUpJ#794497f9-a594-4e81-a3fa-e218ec1a6323) for examples.

### Understanding Focus Collections

The Focus system uses collections to define specific attributes such as HomeProgramType (Hurricane, High Wind), HomeCateogryType (New Home, Existing Home), DesignationTypes (Gold, Silver, Bronze), etc.  
The IBHS Web services allow for the retrieval of all defined collections along with the retrieval of the key/values associated with each collection (view Postman [Collections](https://documenter.getpostman.com/view/9267/RztmsUpJ#fc0295f5-8473-46d7-a267-d1bd304a646d) for more).

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
NOTE: When answering Multi-Occurance questions, use the answer's "AnswerSequence" to distinguish betweeneach set of question/answers.  The AnswerSequence always start with 1 for all questions (never 0), and goes up 1 for each occurance of a group of question/answers.

#### Understanding Question Reponses
If a question requires a specific answer from, say, a drop-down or Yes/No, True/False, etc., the possible values for a given question Id are available via two different service calls:
##### All at once (Bulk)
For situations where the bulk of all responses for all questions can be held, then looked up as needed, the bulk service call will save on repetitive "one-at-a-time" calls.

##### One at a time
A separate call is available for situations where it's more feasible to go after the possible responses to a question "on the fly".


### Understanding Answers
Answer structures from the service are based on important linking information.  These include the Question Id, Answer and Answer Sequence.

It is not necessary to know whether an answer already exists for a given evaluation.  The web service performs an UPSERT (Insert/Update) operation for answers: if the answer doesn't exist, it is created and value set. If it does exist, the values is simply updated.

#### Answer Model: Question ID, Answer Response, Answer Sequence
The web service returns all answers for a given evaluation (in the for of the Fortified Id).

Example:
```
"Answers": [
    {
      "QuestionId": 2,
      "AnswerSequence": 1,
      "Question": "Dwelling Type",
      "Answer": "Single"
    },
    {
      "QuestionId": 3,
      "AnswerSequence": 1,
      "Question": "Exposure Category",
      "Answer": "C"
    },
    {
      "QuestionId": 4,
      "AnswerSequence": 1,
      "Question": "Year Built",
      "Answer": "2018"
    },
    {
      "QuestionId": 5,
      "AnswerSequence": 1,
      "Question": "How was year built determined?",
      "Answer": "Construction Observation by Evaluator"
    },
    {
      "QuestionId": 11,
      "AnswerSequence": 1,
      "Question": "Does home have gables?",
      "Answer": "No"
    },
    {
      "QuestionId": 12,
      "AnswerSequence": 1,
      "Question": "Does home have garage doors?",
      "Answer": "Yes"
    },
    {
      "QuestionId": 13,
      "AnswerSequence": 1,
      "Question": "Does home have a chimney?",
      "Answer": "No"
    },
    {
      "QuestionId": 14,
      "AnswerSequence": 1,
      "Question": "Is structure supported by dry stack or unreinforced foundation?",
      "Answer": "No"
    },
```


### Understand Answer Attachments
[This section is under design considerations and could change at any time]
Much like Answers, attachments are associated to a Fortified Id and can be retrieved, upserted and deleted.  But, unlike Answers, Attachments require resources to be available within the IBHS system indpendently.
To accomplish this, the Attachment structure contains an ImportUrl used for the web services to pull the resource into the IBHS system.  Once the system imports the URL, the attachment's name will be updated:

```
  "AnswerAttachments": [
    {
        "QuestionId": 97,
        "Question": "If access is Partial or None, describe what restricts access? (Type n/a, if not applicable)",
        "AnswerSequence": 1,
        "AttachmentSequence": 2,
        "AttachmentName": "",
        "AttachmentNameUrl": "https://secure.fortifiedhome-ibhs.com/ibhs_uploads/FEH336112019036CE51/images/",
        "AttachmentComment": null,
        "ImportAttachmentUrl": "https://upload.wikimedia.org/wikipedia/commons/9/9a/Tin-roof.jpg",
        "ImportAttachmentFailed": null,
        "ImportAttachmentFailedCount": null,
        "ImportAttachmentDateImported": null
    }
```
Attachments require a Question Id, AnswerSequence and Attachment Sequence.  The AttachmentUrl is used for importing an image.

#### Images in the payload:
It is possible to support transporting base64 byte arrays as a field whereas the web service would transform the byte array into an image or document.  It's important to know that there is a limit to the amount of data transfered in HTTP Post requests and, because of this, would be advised to only provide individual attachment uploads and avoid bulk attachment uploads when the images are in the request's payload.
