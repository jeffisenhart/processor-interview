# SignaPay Transaction Application by Jeff I

This project is a simple Node.js application using Express that allows users to upload transaction files for processing.

## Retrospective

Fun assignment, been a while since I had to do all the processing in one flow.:)

The language I am most comfortable in is Java, but I wanted to challend myslef in node. Some of my stack was
   + node.js
   + express
   + handlebars
   + VS Code
   + docker compose
   + postgres

I opted for super classes an factory patterns for CreditCard and Parsing implementations. I always try to lean into polymorphism so new parsers/cards can be easily added to the system later.

I used some card validation regex examples I found on stack overflow, I do not know their rules/checksums off the top of my head

I spent quite a bit of time on this, what should still be knocked out is
   + Unit test
   + Common contract responses from the controller layer. Currently just have a response with a data field and a nextOffset field, perhaps some opportunity there. Would want the contract to be flexible so it would not matter what the database choice was.

In real life:
   + The upload would just push to some kind of blob storage like S3
   + The upload event would get realized and processed.
      + At the conclusion of the event processing, notifications would be sent
         + One example would be: Error stakeholders would get notified of invalid transactions so they have an opportunity to reconsile
   + The file name should probably give more intel (like vendor, data, country, etc)
   + The aggregate reporting should not be "group by" queries on a main operational transaction table. Periodic jobs or listening to the (aformentioned) events may provide an opportunity to aggregate numbers real time


## Functional Requirements

   * The input files will be either a csv,xml or json file
      + no larger than 2MB
      + file name has no special characters (only alphanumeric, dash, underscore and period are accepted)
      + file (with same name) can only be uploaded once (otherwise it will be rejected)
      + the datatypes in the input files are:
         + card_number - string
         + timstamp - timestamp
         + amount - float
      * amount must be a float
      * amount must not be 0
      * card_number must be 16 characters
      * card_number must match card vendors specifications
   *  The csv files will have a header, (cardNumber,timestamp,amount), with comma (,) as the delimiter and quote (") as the escape character.
      + example:
      ```
      cardNumber,timestamp,amount
      3589249568754350,2025-01-20T21:06:45.771911,-503.57
      3171948530096903,2025-03-01T13:30:25.771911,619.51
      ```
   *  The json files will be an array of json object where each json elememet will the required fields cardNumber,timestamp,amount(for example):
      ```
         [
            {
               "cardNumber": "3589249568754350",
               "timestamp": "2025-01-20T21:06:45.771911",
               "amount": -503.57
            },
            ...
         ]
      ```
   *  The xml files will be an formatted as such:
      + root element is transactions
      + transactions has one to many transaction child nodes
      + the transaction child node will have elements cardNumber,timestamp and amount
      + For example
      ```
      <?xml version="1.0" ?>
      <transactions>
         <transaction>
            <cardNumber>3589249568754350</cardNumber>
            <timestamp>2025-01-20T21:06:45.771911</timestamp>
            <amount>-503.57</amount>
         </transaction>
         ...
         ...
      </transactions>
      ```     

.

## Project Structure

```
signapay
├── src
│   ├── app.js                      # Entry point of the application
│   ├── controllers
│   │   └── transacionController.js # Handles file upload logic
│   │   └── adminController.js      # Useful global queries/use cases
│   ├── model
│   │   └── abstractModel.js        # Base class for model classes
│   │   └── transactionModel.js     # CRUD for transactions
│   │   └── bootstrapModel.js       # Init of tables in case docker init issue
│   │   └── errorModel.js           # CRUD for errors
│   │   └── fileModel.js            # CRUD for files
│   ├── domain
│   │   └── dbResponse.js           # Model response of an insert/delete
│   │   └── util.js                 # Common static functions
│   │   └── uploadProcessStats.js   # Encapsulating upload counts
│   │   └── cards
│   │       └── abstractCards.js    # Base class for credit cards
│   │       └── amexCard.js         # Amex card handling
│   │       └── discoverCard.js     # Discover card handling
│   │       └── visaCard.js         # Visa card handling
│   │       └── masterCard.js       # Master card handling
│   ├── views
│   │   ├── index.hbs               # Main page handlebars file
│   │   └── upload.hbs              # Upload handlebars file
│   │   └── data.hbs                # Transactions handlebars file
│   │   └── errors.hbs              # Errors handlebars file
│   │   └── aggregates.hbs          # Simple reporting file
│   │   └── partials
│   │       └── base.hbs            # Common Header
│   └── public
│       └── styles.css              # CSS styles for the 
│       └── util.js                 # js utility functions
├── test
│   ├── test.csv                    # Provided input file
│   └── test.xml                    # Provided input file
│   └── test.json                   # Provided input file
├── package.json                    # npm configuration file
├── .env                            # Environment variables
├── .gitignore                      # gitignore file
└── README.md                       # Project documentation
```

## Installation

1. Clone the repository:
   ```
   git clone https://github.com/jeffisenhart/processor-interview.git
   cd to the processor-interview dir
   ```

2. Install the dependencies:
   ```
   npm install
   ```


## DB creation
   Based on this: https://levelup.gitconnected.com/creating-and-filling-a-postgres-db-with-docker-compose-e1607f6f882f

1. Navigate to the docker directory and run:
   ```
   //This will create the db and the needed tables
   //to run in background
   docker compose up -d
   //otherwise
   docker compose up

   //note, to tear down db environment
   docker compose down
   ```


## Usage

1. Start the DB
   ```
   docker compose up -d
   ```
2. Start the application:
   ```
   npm start
   ```
   or in VS Code, 
   ```
   debug -> Run Script: dev
   ```

2. Open your browser and navigate to http://localhost:3000 to access the main page.

3. This will bring up the Transaction Portal
   * Upload Transaction - brings up the page to upload the CSV data file
   * View Transactions - brings up the page to see successfully uploaded transaction by the given upload file
   * View Errors - brings up the page to see data that did not create a transaction as there were errors
   * Clear All Data - A confirmation button to clear all the database data

# Card Processor

Thank you for taking a the time to complete our interview code project. We realize that there are many ways to conduct the "technical part" of the interview process from L33T code tests to whiteboards, and each has its own respective pros / cons. We intentionally chose the take-home project approach because we believe it gives you the best chance to demonstrate your skills and knowledge in a "normal environment" - i.e. your computer, keyboard, and IDE.

This short exercise is designed to give us a sense of how you approach full stack development. We’re looking for clarity of thought, communication, and code organization — not perfection or a complete product.

Please treat this as something you’d spend **3-5 hours** on. If anything is unclear or you'd make a different decision in a real-world scenario, feel free to call that out.

We encourage you to have fun with the project, while producing a solution that you believe accurately represents how you would bring your skillset to the team.

We have attempted to make this repo as clear as possible, but if you have any questions, we encourage you to reach out.

---

## 📟 Overview

You’ll build a small system that mimics a simplified credit card transaction processor. Your submission should include:

- A **web-based user interface**
  - including a **reporting view** to summarize processed data
- A **server component** with appropriate business logic
- **Data persistence** (in-memory, file-based, or database - your choice)

---

## 🧹 Requirements

You can implement this however you like, as long as the following functionality is covered:

### 1. User Interface

- A minimal web interface for interacting with the system
- Includes a way to view and/or submit transaction data
- Can be built using any approach / tech stack you prefer

### 2. Logic

- You need to accept transaction records
  - These records will come from the provided files
  - The files in the test directory are smaller and meant to ease development
  - The files in the data directory are larger and mean to be the "real transactions"
- Each record includes a card number, timestamp, and amount
- Each directory contains 3 files that need to be processed to capture all transactions
- Card type is determined by the leading character of the card as follows:
  - Amex (3)
  - Visa (4)
  - MasterCard (5)
  - Discover (6)
- Invalid or unrecognized card numbers should be rejected

### 3. Persistence

- Transactions must be stored and retrievable after creation
- Choose any persistence mechanism

### 4. Reporting

- Provide summaries of total processed volume:
  - By Card
  - By Card Type
  - By Day (based on timestamp)
- Provide a list of "rejected" transactions

---

## ✅ What We’re Looking For

- Clear, maintainable code
- A working implementation of the core requirements
- Reasonable structure and organization
- Good judgment in scoping and tradeoffs

Bonus points (not required) for:

- Tests
- Clear commit history
- Clean and responsive UI

## 🧠 Final Thoughts and Hints

- In this scenario, you are the initial architect creating the first pass at this project. You can consider our review the same as a Senior level engineer coming on to the project. Make sure that when we "pick up" the repo, it is clear how to stand up the project, run the solution, and potentially contribute code
- Since you are tackling this specific project, our expectation is that you are at a senior engineer level. While we 100% want your code to represent your preferred style, there are some things we consider "basic" that should be in your submission. These include ideas like the following list. This list is not exhaustive, it is meant to point in a direction:
  - Clear, consistent, readable code
  - Proper use of your selected stack
    - for example, if you choose C#, we would expect to see IOC/DI appropriately implemented
  - DRY
  - Low cyclomatic complexity
  - Low Coupling / High Cohesion
  - Clear thought and patterns for maintainability and expansion
    - This scenario is obviously simplified from reality, that said you should consider future requests like other transaction types, different file formats, etc. - this will at minimum, be a topic in the conversation
- While it should be obvious, this scenario involves "money". This means numerical accuracy is required and at least minimal security should be considered in your submission (we aren't going to "hack your solution", but there shouldn't be open API endpoints either).
- We do NOT expect you to be a designer, we do expect you to consider your user and make the experience intuitive and easy to use

---

## 📦 Submitting

- Fork this repository and push your implementation to your fork
- Submit a pr to this repository when you are ready for us to review your code
  - We will close the PR then review the code on your fork
- Include / Update `README.md` to explain:
  - How to run your code
  - Any decisions or tradeoffs you made
  - Any assumptions or known limitations

