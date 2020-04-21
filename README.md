# Jobcast Insight
Classifier using Accord Framework: http://accord-framework.net/

## Classification

### Train

Create folder with endpoint name as the folder name.

```
mkdir job_title
```

Inside the new folder, create `text.txt` with content that you want to classify, with one group of text on each line. Example (4 lines):

```
As a Software Developer at ABC Company, you will be responsible for the full development cycle of ABC Company products. You are given the opportunity to participate in the development cycle from coding, bug fixing, troubleshooting and testing, enabling you to familiarize yourself with Fortinet products and have direct involvement with complex, innovative technology as well as the opportunity to work with the experienced developers that enables you to fast track your career growth.
The position We offer the position for a passionate software developer with strong software development experience. The Role  Research, design, implement and manage various software programs - Provide detailed development and execution plan of software - Develop, test and debug software - Develop regular maintenance and upgrades for existing applications - Assist in the collection and documentation of user requirements - Identify and report technical problems and propose solutions - Prepare manuals on software operation and guides on end-users - Degree in Computer Science, IT, Engineering or related field - 3+ years of experience in a similar role - Knowledge and interest in software development and the latest technologies - An analytical mind - The ability to communicate complex procedures to other colleagues - Commercial and business awareness
Reporting into the Controller, you will be responsible for the following: - Project Accounting - Reporting costs to date with project forecasting. - Invoicing - Budgetting - Assist with Payroll and A/P - Assist with month end - Full cycle accounting for two small companies - G/L account recs - Monthly tax remittances
RESPONSIBILITIES: Full-cycle Accounting including: •Inventory Reporting - maintain and analyze inventory levels and update periodic summary and journal entries for financial reporting •Fixed Asset Reporting - reconcile general ledger, adjusting journal entries and provide financial performance analysis •Long Term Debt and Capital Lease Reporting - report on liability position and debt classification •Update, prepare and analyze periodic working paper, variance analysis, and reports •Prepare any required adjusting entries as required after an investigation •Ensure all period end, quarter-end, year-end requirements in the assigned areas are completed in a timely manner •Prepare assigned balance sheet account reconciliation and ensure accuracy
```

Next to `text.txt`, create as many classification files as you'd like.

E.g. `title.txt` with the job title classification for each line:

```
Software Developer
Software Developer
Accountant
Accountant
```

E.g. `industry.txt` with the industry classification for each line:

```
Technology
Technology
Non-Tech
Non-Tech
```

Run `train/bin/Debug/train.exe` on the folder:
```
.\train.exe "{path to folder}"
```

It will generate a `foldername`.cl file. Rename the extension to `zip` to see the contents of the file.

Copy the *.cl file to Insight.Web/DB/foo. Now try running the classifier:

### Classify

```
curl -H "Content-Type: text/plain" -X POST -d "Looking for software developer" http://localhost:21407/parsing/foo
```

Result:

```
[{
    "industry": [{
        "text": "Technology",
        "score": 0.99500742460394365
    }, {
        "text": "Non-Tech",
        "score": 0.0049925753960563535
    }],
    "title": [{
        "text": "Software Developer",
        "score": 0.991214816133925
    }, {
        "text": "Accountant",
        "score": 0.0087851838660749637
    }]
}]
```

Or you can do multi-lines:

```
POST http://localhost:21407/parsing/foo HTTP/1.1
User-Agent: Fiddler
Content-Type: text/plain
Host: localhost:21407
Content-Length: 92

Looking for a JavaScript developer
Work with numbers, fixing budgets, filing annual taxes
```

Result:

```
[{
    "industry": [{
        "text": "Technology",
        "score": 0.991622049562957
    }, {
        "text": "Non-Tech",
        "score": 0.0083779504370430358
    }],
    "title": [{
        "text": "Software Developer",
        "score": 0.99101188660475181
    }, {
        "text": "Accountant",
        "score": 0.0089881133952481873
    }]
}, {
    "industry": [{
        "text": "Non-Tech",
        "score": 0.6385068761727275
    }, {
        "text": "Technology",
        "score": 0.36149312382727244
    }],
    "title": [{
        "text": "Accountant",
        "score": 0.52784115874744608
    }, {
        "text": "Software Developer",
        "score": 0.47215884125255392
    }]
}]
```

If you run into `roslyn\csc.exe` exception, try this in Package Manager Console:
```
Update-Package Microsoft.CodeDom.Providers.DotNetCompilerPlatform -r
```
Source: https://stackoverflow.com/a/34391473/188740

## Other uses
From README.TXT found in production
```
*.pm files format example:

"Label","Pattern"
"Administrator","'admin' | administrator"
"Software Developer","(software developer | programmer) & !{Administrator}"

*.cl is a zip file generated with train.exe utility - see train.zip content.

   train.exe <folder path>

Folder should contain text files with the same amount of text lines:

   text.txt - text to train on, an instance per line.
   *.txt - assigned category files, an assignment per line

*.svm file name in *.cl zip acrhive is changable to apply filters:

  isSpam [1].svm  -> means top 1 label
  isSpam [5].svm  -> means top 5 label
  isSpam [90%].svm  -> means allowing labels with probability >= 90%
  isSpam [90%][5].svm  -> means allowing labels with probability >= 90%
  isSpam [90%][5].svm  -> takes 5 labels with probablity >= 90%
```