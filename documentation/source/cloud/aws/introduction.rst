AWS Integration
=======================

Flask-Philo supports basic `Amazon Web Service (AWS) <https://aws.amazon.com/>`_ integration
using Amazon's `boto3 <https://pypi.python.org/pypi/boto3>`_ AWS SDK.

More specifically, Flask-Philo provides methods for integration with two main services within Amazon's AWS family:

* **S3 Storage** - Integration with Amazon's S3 file storage buckets, providing a number of useful methods for storing and retrieving data
* **SQS Queuing** - Integration with Amazon's Simple Queuing Service (SQS), with a number of methods for creating and managing queuing systems


Settings AWS Credentials
-----------------------------------

Flask-philo supports two means of authentication for AWS. Firstly, via environment variables:

::

    $ export AWS_SECRET_ACCESS_KEY=your_secret_key
    $ export AWS_ACCESS_KEY_ID=your_access_key


...or via configuration in the settings file:


``<your_app>/config/development.py``
::

    AWS = {
        'credentials': {
            'aws_access_key_id': 'your_access_key',
            'aws_secret_access_key': 'your_secret_key'
        }
    }


Amazon S3 Bucket
-----------------

Flask-Philo supports the use of Amazon's S3 file storage buckets, and provides a number of useful methods for storing and retrieving data.

Each S3 method may be called in two ways:

* Imported and called as a standard Python function
* Invoked from the command line using ``manage.py``

...examples for both are included for every method documented below


Retrieving available Bucket contents
############################

To list all available items within a specified S3 Bucket, we use the *list_objects_v2* method

``list_objects_v2(bucket_name, region_name)``

* **bucket_name** : Name of Amazon S3 Bucket
* **region_name** : Name of Amazon S3 Region

Example Python calling code :

::

    from flask_philo.cloud.aws.s3 import list_objects_v2

    # Retrieve bucket content
    bucket_name = 'my_data_bucket'
    region_name = 'us-west-2'
    bucket_content = list_objects_v2(bucket_name, bucket_region)['Contents']

    # Print all bucket items
    print("Bucket contents : ")
    for bucket_item in bucket_content:
        print(bucket_item['Key'])

    ##########################
    This code yields the following printed output :
    Bucket contents :
    readme.txt
    13167621.mp3
    18776371.mp3


This method may also be directly invoked from the command line as follows:

::

    $ python3 manage.py aws s3 list_objects_v2 --bucket my_data_bucket --region us-west-2
    +----+--------------+---------------------------+---------------+---------------+
    |    |     Key      |       Last Modified       | Size (bytes)  | Storage Class |
    +----+--------------+---------------------------+---------------+---------------+
    | 1  | readme.txt   | 2018-04-11 15:56:52+00:00 |    1238522    |    STANDARD   |
    | 2  | 13167621.mp3 | 2018-04-11 15:56:52+00:00 |    7110629    |    STANDARD   |
    | 3  | 18776371.mp3 | 2018-04-11 15:57:07+00:00 |    8877935    |    STANDARD   |
    +----+--------------+---------------------------+---------------+---------------+



Downloading a file from an S3 Bucket
###################################

Individual S3 data items may be retrieved using the *download_file* method

``download_file(destination_filename, bucket_name, source_key, bucket_region)``

* **destination_filename** : Local, writable file location for downloaded file
* **bucket_name** : Name of Amazon S3 Bucket
* **source_key** : Amazon S3 key for the desired bucket item
* **region_name** : Name of Amazon S3 Region

Example Python calling code :

::

    from flask_philo.cloud.aws.s3 import download_file, list_objects_v2

    # Retrieve first bucket item
    bucket_name = 'my_data_bucket'
    region_name = 'us-west-2'
    bucket_item = list_objects_v2(bucket_name, region_name)['Contents'][0]

    # Download bucket item to new file location "dest/my_new_local_file.txt"
    download_file('dest/my_new_local_file.txt', bucket_name, bucket_item['Key'], region_name)


This method may also be directly invoked from the command line as follows:

::

    $ python3 manage.py aws s3 download_file --bucket my_data_bucket --region us-west-2 --fname my_new_local_file.txt --key my_bucket_file.txt


Uploading a file to an S3 Bucket
###############################

Individual files may be uploaded to an S3 bucket using the *upload_file* method

``upload_file(source_filename, bucket_name, destination_key, bucket_region)``

* **source_filename** : Local, readable file location as source of upload
* **bucket_name** : Name of Amazon S3 Bucket
* **destination_key** : New Amazon S3 key for the uploaded bucket item
* **region_name** : Name of Amazon S3 Region

Example Python calling code :

::

    from flask_philo.cloud.aws.s3 import upload_file, list_objects_v2

    bucket_name = 'my_data_bucket'
    region_name = 'us-west-2'

    # Upload new file to S3 Bucket using Key 'My_New_File_Key'
    upload_file('dest/my_new_local_file.txt', bucket_name, 'My_New_File_key', region_name)


This method may also be directly invoked from the command line as follows:

::

    $ python3 manage.py aws s3 upload_file --bucket my_data_bucket --region us-west-2 --fname my_local_file.txt --key My_New_File_key.txt



Uploading a folder to an S3 Bucket
#################################

Bulk uploads of an entire directory's contents is possible using the *upload_dir* method

``upload_dir(source_dir, bucket_name, region_name)``

* **source_dir** : Local, readable directory containing all files for upload
* **bucket_name** : Name of Amazon S3 Bucket
* **region_name** : Name of Amazon S3 Region

Example Python code :

::

    from flask_philo.cloud.aws.s3 import upload_dir

    bucket_name = 'my_data_bucket'
    region_name = 'us-west-2'
    source_dir = './my_files/for_upload'
    upload_dir(source_dir, bucket_name, region_name)

This method may also be directly invoked from the command line as follows:

::

    $ python3 manage.py aws s3 upload_dir --bucket my_data_bucket --region us-west-2 --root_folder ./my_files/for_upload


------------



Amazon Simple Queuing Service (SQS)
------------------------------

To facilitate task queueing between software components (e.g. between multiple decoupled microservices), Flask-Philo Integrates with Amazon's Simple Queuing Service (SQS), with a number of methods for creating and managing message queuing systems.

For more detail on SQS message queuing, visit the `SQS Introduction <https://aws.amazon.com/sqs/>`_

Each SQS methods may be called in two ways:

* Imported and called as a standard Python function
* Invoked from the command line using ``manage.py``

...examples for both are included for every method documented below

Sending a Message
#################

Send a single message to a queue using the *send_message* method

``send_message(queue_url, message_body, region_name)``

* **queue_url** : URL for SQS queue
* **message_body** : Body of queue message
* **region** : Name of Amazon S3 Region

Example Python code :

::

    from flask_philo.cloud.aws.sqs import send_message

    queue_url = 'https://us-west-2.queue.amazonaws.com/523525905522/new_test_queue'
    message_body = 'My new test message'
    region = 'us-west-2'
    data = send_message(queue_url, message_body, region)

This method may also be directly invoked from the command line as follows:

::

    $ python3 manage.py aws sqs send_message --queue_url https://us-west-2.queue.amazonaws.com/523525905522/new_test_queue --message_body "New Queue Message" --region us-west-2

    execute_command :  aws
    {
      'MD5OfMessageBody': '0ff14a859b7243d32b52126afe82eb68',
      'MessageId': '83dcc0b9-81bf-488d-a8a8-0941228dcec6',
      'ResponseMetadata': {
        'HTTPStatusCode': 200,
        'RetryAttempts': 0,
        'RequestId': '5a78a04b-e4b1-51e1-afa4-f3bc7dd7ac9e',
        'HTTPHeaders': {
          'x-amzn-requestid': '5a78a04b-e4b1-51e1-afa4-f3bc7dd7ac9e',
          'server': 'Server',
          'connection': 'keep-alive',
          'content-type': 'text/xml',
          'date': 'Wed, 22 Aug 2018 17:02:30 GMT',
          'content-length': '378'
        }
      }
    }

Sending a Message Batch
#######################

Send multiple messages to a queue using the *send_message_batch* method

``send_message_batch(queue_url, entries, region)``

* **queue_url** : URL for SQS queue
* **entries** : List of message objects, in dictionary form
* **region** : Name of Amazon S3 Region

Example Python code :

::

    from flask_philo.cloud.aws.sqs import send_message_batch

    url = 'https://us-west-2.queue.amazonaws.com/523525905522/new_test_queue'
    region = 'us-west-2'
    message_batch = [
        {"Id": "1", "MessageBody": "Test Message One"},
        {"Id": "2", "MessageBody": "Test Message Two"}
    ]

    data = send_message_batch(queue_url=url, entries=message_batch, region=region)

This method may also be directly invoked from the command line as follows:

::

    $ python3 manage.py aws sqs send_message_batch --queue_url https://us-west-2.queue.amazonaws.com/523525905522/new_test_queue --region us-west-2 --entries "[{\"Id\":\"1\",\"MessageBody\":\"[message one]\"},{\"Id\":\"2\",\"MessageBody\":\"[message two]\"}]"

    execute_command :  aws
    {
      'ResponseMetadata': {
        'RetryAttempts': 0,
        'RequestId': '9066cfe7-4504-5e4a-ac68-9c10dfa212cb',
        'HTTPHeaders': {
          'connection': 'keep-alive',
          'server': 'Server',
          'x-amzn-requestid': '9066cfe7-4504-5e4a-ac68-9c10dfa212cb',
          'date': 'Wed, 22 Aug 2018 17:07:50 GMT',
          'content-type': 'text/xml',
          'content-length': '734'
        },
        'HTTPStatusCode': 200
      },
      'Successful': [
        {
          'Id': '30b7b2c9-ac68-4541-af5b-64b24935c4c4',
          'MD5OfMessageBody': 'abd51c84e005843b6bffeb7a2d6526c7',
          'MessageId': 'a26c01a4-39d3-4381-91e4-17b4e3f04177'
        },
        {
          'Id': '1caac440-f795-4294-979b-ea9759ae6a47',
          'MD5OfMessageBody':
          'f0ca119551cf7532da54a23477fd39ff',
          'MessageId': '8c5cd9d4-6fae-4a2c-a5de-05fc4a6334a6'
        }
      ]
    }


Retrieving Messages
#################

To retrieve a single message from a queue, use the *receive_message* method

``receive_message(queue_url, max_number_of_messages, region)``

* **queue_url** : URL for SQS queue
* **max_number_of_messages** : *Optional* Specify number of message to be retrieved
* **region** : Name of Amazon S3 Region

Example Python code :

::

    from flask_philo.cloud.aws.sqs import receive_message

    url = 'https://us-west-2.queue.amazonaws.com/523525905522/new_test_queue'
    region_name = 'us-west-2'

    retrieved_messages = receive_message(queue_url=url, region=region_name)
    print("Body of message retrieved :", retrieved_messages['Messages'][0]['Body'])

    ##########################
    This code yields the following printed output :
    Body of message retrieved : My new test message


Optionally, we may retrieve more than one message at a time using the ``max_number_of_messages`` attribute

::

    from flask_philo.cloud.aws.sqs import receive_message

    url = 'https://us-west-2.queue.amazonaws.com/523525905522/new_test_queue'
    region_name = 'us-west-2'

    retrieved_messages = receive_message(queue_url=url, region=region_name, max_number_of_messages=2)

    # Print all retrieved messages
    print("Messages : ")
    for message in retrieved_messages['Messages']:
        print(message['Body'])

    ##########################
    This code yields the following printed output :
    Messages :
    Test Message One
    Test Message Two

This method may also be directly invoked from the command line as follows:

::

    $ python3 manage.py aws sqs receive_message --queue_url https://us-west-2.queue.amazonaws.com/523525905522/new_test_queue --region us-west-2

    execute_command :  aws
    {
      "ResponseMetadata": {
        "RetryAttempts": 0,
        "HTTPHeaders": {
          "date": "Wed, 22 Aug 2018 17:13:05 GMT",
          "connection": "keep-alive",
          "content-length": "863",
          "x-amzn-requestid": "45dbb41c-3cb1-599f-bc19-e7eba750f0e2",
          "server": "Server",
          "content-type": "text/xml"
        },
        "RequestId": "45dbb41c-3cb1-599f-bc19-e7eba750f0e2",
        "HTTPStatusCode": 200
      },
      "Messages": [
        {
          "ReceiptHandle": "AQEBptTr8Wns1LqPFVicLxUwobj2Nlk8nheH7r/f4H47KsP61iL7pwB6B1za9iCvHMooFB/JqX1dXEjd+JlGE9NYTGWhKz9dL8GqSrdmpJfLQmCxtprX83pNO3Xxrei8q81kQAhp0813/X8bxkxCboU/+c9xp83cJ/29fiQfP5nQTURHPMShysSoZ6UmEyvF5tAlrE28mr7WhpVGLs7birZPGEJFLB9cfESTXqqSAWNIbw+xDIXKS6E53hj37XktQ6juL+xKFUpEnIWympGCrFW09pigrotiA2Ysri3pjsc2ra4VY5UTk/L7EkTd0CymiyLdKcDVrYfW9bBYea9Jy2zCmi2tv7Bm/lpO1/ZLOdl/lU8X5uN7APGCMW0a5ba3RNHV/zbUWUmXpyqA6X3jaZvvGw==",
          "MessageId": "619b1306-fe8b-4289-9ef4-aa07577b407a",
          "Body": "Queue_item 1",
          "MD5OfBody": "4b83caab69a87b19e2566f1c4d0710d5"
        }
      ]
    }


Listing Available Queues
#########################

To obtain a list of all available SQS queues grouped by region, use the *list_queues* method.
Note that this method may take some time to return, given that it must iteratively poll all accessible Amazon SQS regions

``list_queues()``

Example Python code :

::

    from flask_philo.cloud.aws.sqs import list_queues

    queues_by_region = list_queues()
    queue_url_list = queues_by_region['us-west-2']['QueueUrls']

    print("Queue URLs :")
    for queue_url in queue_url_list:
        print(queue_url)

    ##########################
    This code yields the following printed output :
    Queue URLs :
    https://us-west-2.queue.amazonaws.com/523525905522/test_queue
    https://us-west-2.queue.amazonaws.com/523525905522/new_test_queue
    https://us-west-2.queue.amazonaws.com/523525905522/my-priority-list


This method may also be directly invoked from the command line as follows:

::

    $ python3 manage.py aws sqs list_queues
    +-----------------------------------------------------------------------------+-----------+
    |                                    Queue Url                                |   Region  |
    +-----------------------------------------------------------------------------+-----------+
    |         https://us-west-2.queue.amazonaws.com/523522205522/test_queue       | us-west-2 |
    |        https://us-west-2.queue.amazonaws.com/523525905522/new_test_queue    | us-west-2 |
    |         https://us-west-2.queue.amazonaws.com/523525905522/my-priority-list | us-west-2 |
    +-----------------------------------------------------------------------------+-----------+


Create a New Queue
##################

To create new SQS queue, use the *create_queue* method

``create_queue(queue_name, region)``

* **queue_name** : Name for new SQS queue
* **region** : Name of Amazon S3 Region

::

    from flask_philo.cloud.aws.sqs import create_queue

    # Create new SQS queue
    region_name = 'us-west-2'
    new_queue = create_queue("my_test_queue", region_name)
    sqs_url = new_queue['QueueUrl']

    # Send a message to the new queue
    data = send_message(sqs_url, 'Queue_item 1', region_name)

This method may also be directly invoked from the command line as follows:

::

    $ python3 manage.py aws sqs create_queue --region us-west-2 --queue_name my-new-sqs-queue

    execute_command :  aws
    {
      "ResponseMetadata": {
        "RequestId": "426395bd-9e0d-5aee-b0f9-c64a02dd38ed",
        "RetryAttempts": 0,
        "HTTPHeaders": {
          "connection": "keep-alive",
          "content-type": "text/xml",
          "server": "Server",
          "date": "Wed, 22 Aug 2018 17:17:04 GMT",
          "x-amzn-requestid": "426395bd-9e0d-5aee-b0f9-c64a02dd38ed",
          "content-length": "338"
        },
        "HTTPStatusCode": 200
      },
      "QueueUrl": "https://us-west-2.queue.amazonaws.com/523525905522/my-new-sqs-queue"
    }



Purge a Queue of all Messages
############

To purge an SQS queue of all messages, use the *purge_queue* method

``purge_queue(queue_url, region)``

* **queue_url** : URL of an existing SQS queue
* **region** : Name of Amazon S3 Region

::

    from flask_philo.cloud.aws.sqs import purge_queue

    # Purge the SQS queue
    sqs_url = 'https://us-west-2.queue.amazonaws.com/523525901222/existing_sqs_queue'
    region_name = 'us-west-2'
    purge_queue(queue_url=sqs_url, region=region_name)

This method may also be directly invoked from the command line as follows:

::

    $ python3 manage.py aws sqs purge_queue --region us-west-2 --queue_url https://us-west-2.queue.amazonaws.com/523525905522/my-new-sqs-queue

    execute_command :  aws
    {
      "ResponseMetadata": {
        "RetryAttempts": 0,
        "HTTPStatusCode": 200,
        "RequestId": "b3f869d0-cab1-54c9-9f76-3a2757f6d6ba",
        "HTTPHeaders": {
          "content-length": "209",
          "x-amzn-requestid": "b3f869d0-cab1-54c9-9f76-3a2757f6d6ba",
          "server": "Server",
          "connection": "keep-alive",
          "content-type": "text/xml",
          "date": "Wed, 22 Aug 2018 17:19:04 GMT"
        }
      }
    }


Delete a Queue
############

To entirely delete an SQS queue and its messages, use the *delete_queue* method.

``delete_queue(queue_url, region)``

* **queue_url** : URL of an existing SQS queue for deletion
* **region** : Name of Amazon S3 Region

::

    from flask_philo.cloud.aws.sqs import delete_queue

    # Delete the SQS queue
    sqs_url = 'https://us-west-2.queue.amazonaws.com/523525901222/existing_sqs_queue'
    region_name = 'us-west-2'
    delete_queue(queue_url=sqs_url, region=region_name)

This method may also be directly invoked from the command line as follows:

::

    $ python3 manage.py aws sqs delete_queue --region us-west-2 --queue_url https://us-west-2.queue.amazonaws.com/523525905522/my-new-sqs-queue
    execute_command :  aws
    {
      "ResponseMetadata": {
        "RetryAttempts": 0,
        "HTTPHeaders": {
          "connection": "keep-alive",
          "date": "Wed, 22 Aug 2018 17:20:35 GMT",
          "server": "Server",
          "content-length": "211",
          "x-amzn-requestid": "947d5da5-6995-5cb0-b828-0e52f4368c8a",
          "content-type": "text/xml"
        },
        "HTTPStatusCode": 200,
        "RequestId": "947d5da5-6995-5cb0-b828-0e52f4368c8a"
      }
    }


External Resources
-----------------------

* `AWS SDK Boto3 <https://pypi.python.org/pypi/boto3>`_

* `AWS <https://aws.amazon.com/>`_
