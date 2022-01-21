![GitHub Classroom Workflow](../../workflows/GitHub%20Classroom%20Workflow/badge.svg?branch=main)
<img alt="points bar" align="right" height="36" src="../../blob/badges/.github/badges/points-bar.svg" />

# COMP0061 -- Privacy Enhancing Technologies -- Lab 02

This lab will introduce the basics of Petlib, encryption, signatures, and an end-to-end encryption system.

### Structure of Labs
The structure of all the labs will be similar: two Python files will be provided. 

- The first is named `Lab0XCode.py` and contains the structure of the code you need to complete. 
- The second is named `Lab0XTests.py` and contains unit tests (written for the Pytest library) that you may execute to 
partially check your answers. 

Note that the tests passing is a necessary but not sufficient condition to fulfill each task.
There are programs that would make the tests pass that would still be invalid (or blatantly insecure) implementations.

The only dependency your Python code should have, besides Pytest and the standard library, is the Petlib library, 
which was specifically developed for this course (and also for our own use!). 

The petlib documentation is [available on-line here](http://petlib.readthedocs.org/en/latest/index.html).

### Checking out code

Check out the code by using your preferred git client (e.g., git command line client, Github Desktop, Sourcetree).
Alternatively, if you are using VSCode, click the `Open in Visual Studio Code` button above to automatically check
out and open the repository.

### Setup
The intended environment for this lab is the Linux operating system with Python 3 installed.

We provide a `setup.sh` file that creates a local virtual environment, installs the dependencies needed for the lab,
and activates the virtual environment. To run the setup file, type `source setup.sh` into the terminal. The virtual
environment is needed to run the unit tests locally. 

*Alternatively:* The tests are the same as the ones that run as part of the Github Classroom automated marking system, 
so you can also run the tests by simply committing and pushing your changes to Github, without the need for a local 
setup or even having Python 3 installed.

### Working with unit tests
Unit tests are run from the command line by executing the command:

```sh
$ pytest -v Lab01Tests.py
```

Note the `-v` flag toggles a more verbose output.
If you wish to inspect the output of the full tests run you may pipe this command to the `less` utility 
(execute `man less` for a full manual of the less utility):

```sh
$ pytest -v Lab01Tests.py | less
```

You can also run a selection of tests associated with each task by adding the Pytest marker for each task to the Pytest
command:

```sh
$ pytest -v Lab01Tests.py -m task1
```
The markers are defined in the test file and listed in `pytest.ini`.

You may also select tests to run based on their name using the `-k` flag.
Have a look at the test file to find out the function names of each test.
For example the following command executes the very first test of Lab 1:

```sh
$ pytest -v Lab01Tests.py -k test_petlib_present
```

The full documentation of pytest is [available here](http://pytest.org/latest/).


### What you will have to submit
The deadline for all labs is at the end of term but labs will be progressively released throughout the term, as new
concepts are introduced. 
We encourage you to attempt labs as soon as they are made available and to use the dedicated lab time to bring up any
queries with the TAs.

Labs will be checked using Github Classroom, and the tests will be run each
time you push any changes to the `main` branch of your Github repository.
The latest score from automarking should be shown in the Readme file.
To see the test runs, look at the Actions tab in your Github repository. 

Make sure the submitted `Lab01Code.py` file at least satisfies the tests, without the need for any external dependency 
except the Python standard libraries and the Petlib library. 
Only submissions prior to the Github Classroom deadline will be marked, so make sure you push your code in time.


To re-iterate, the tests passing is a necessary but not sufficient condition to fulfill each task.
All submissions will be checked by TAs for correctness and your final marks are based on their assessment of your work.  
For full marks, make sure you have fully filled in any sections marked with `TODO` comments, including answering any
questions in the comments of the `Lab01Code.py`.



## TASK 1 -- Check installation

> Ensures that the key libraries may be loaded, and the code files are present. Nothing to do beyond ensuring this is the case.

## TASK 2 -- Build a simple 1-hop mix client

> You are provided the code of the inner decoding function of a simple, one-hop mix server. Your task is to write a function that encodes a message to be send through the mix.

## Hints:

- You can run the tests just for this task by executing:

	py.test -v Lab02Tests.py -m task2

- Your objective is to complete the function `mix_client_one_hop`. This function takes as inputs a public key (an EC element) of the mix, an address and a message. It must then encode the message to be processed by the `mix_server_one_hop` in such a way that the mix will output a tuple of (address, message) to be routed to its final destination.

- The message type is a Python NamedTuple already defined for you as `OneHopMixMessage`. The function `mix_client_one_hop` must return an object of this type. Such an object may be created simply by calling:

	OneHopMixMessage(client_public_key, expected_mac, address_cipher, message_cipher)

where the `client_public_key` is an EC point, the expected Hmac is an Hmac of the `address_cipher` and `message_cipher`, and those are AES Counter mode (AES-CTR) ciphertexts of the encoded address and message.

- Study the function `mix_server_one_hop` that implements the one-hop mix. Take note of all the `petlib` cryptographic operations and checked performed in order to process a message. You will have to ensure they decode your message correctly.

- The first element of a message is an ephemeral public key defined by the client (and the client knows its private part). The private key is used to derived a shared secret with the mix, using the mix public key. Study the code of the one-hop mix to examine the key derivation, and ensure your client mirrors it to generate messages that decode correctly.

- Study the code of the mix in `mix_server_one_hop` to determine the cryptographic operations necessary to encrypt correctly the address and message, as well as producing a valid Hmac. A helper function `aes_ctr_enc_dec` is provided to help you encrypt/decrypt using AES Counter mode. If you are not familiar with [AES Counter mode have a look online](http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Counter_.28CTR.29).

- An Hmac is a cryptographic checksum that can be used to ensure parts of a message were generated by the holder of a shared secret key. If you are unsure of what a message authentication code does, do check the [HMAC primitive on wikipedia](http://en.wikipedia.org/wiki/Hash-based_message_authentication_code). You can find functions to generate and check Hmac in `petlib.hmac`.

## TASK 3 -- Build a n-hop mix client.

> In this exercise you will be required to encode a message to be relayed by a cascade of mixes. Each of the mixes has a public key, and the messages is passed to each of the mixes in order before being output.
>
> Your task here is to complete the `mix_client_n_hop` so that it encodes an address and message to be relayed by a sequence of mixes in order (the sequence is implicit, the order of their public keys is given.)

## Hints:

- You can run the tests just for this task by executing:

	py.test -v Lab02Tests.py -m task3

- The key differences between the 1-hop mix and the n-hop mix message encoding relates to: (1) the use of a blinding factor to provide bit-wise unlikability of the public key associated with the message; (2) the inclusion of a sequence (list) of hmacs as the second part of the mix message; (3) the decryption of the hmacs (in addition to the address and message) at each step of mixing.

- The output of the function `mix_client_n_hop` should be an `NHopMixMessage`. The first element is a public key, the second is a list of hmacs, the third is the encrypted address and the final one is the encrypted message. You can build such a structure using:

	NHopMixMessage(client_public_key, hmacs, address_cipher, message_cipher)

- Study the mix operation in `mix_server_n_hop`. You will notice that the mix derives a shared key, and then uses it to check the first hmac in the list against the rest of the ciphertext. You must ensure your client encoding passes this test.

- Debugging tip: the blinding factor used to "change" the public key of each message makes this task quite complex. Ensure that the sequence of shared keys the client encoder derives is the same as the sequence of shared keys observed by each mix in order. If that sequence is different, then the rest of the decoding is very likely to fail.

- This task is fiddly, and you are likely to see a number of hmac verification failures. Do not lose hope: systematically dump the inputs to your hmacs (using `print`) as well as the keys, to ensure the client and the servers hmac the same data under the same keys.

## TASK 4 -- Simple Traffic Analysis / Statistical Disclosure.

> In this exercise you will be required to recover the social contacts of a target user, that is sending messages through an anonymity system (traffic analysis). A trace of traffic is provided to your function, as well as the number of social contacts of the target. You should return your best guess about who the user 0 has sent messages to.

## Hints:

- You can run the tests just for this task by executing:

	py.test -v Lab02Tests.py -m task4

- There is no need to use `petlib` for this task. Using other Python facilities such as the `Counter` class (from `collections`) might be helpful to keep your answer short (but not necessary).

- The trace provided is a list of tuples [(Senders, Receivers)]. Each item of the list represents one round of the anonymity system, the senders observed sending in this round, and the receivers observed receiving. The identifiers of the senders / receivers are just small integers.

- Your task is to complete the function `analyze_trace` to return the identifiers of a number of receivers that are the friends (the target has sent messages to) of the target. The number of friends sought is provided as `target_number_of_friends`, and the identifier of the target sender is provided as `target` (by default 0). You must receive a the list of receivers that Alice is sending messages to.

- Do study the function `generate_trace` that simulates a very simple anonymity system. It provides a good guide as to the types of data in the trace and their meaning, as well as a model you analysis can be based on.

- Remember the insight from the Statistical Disclosure Attack: anonymity systems provide imperfect privacy. This is mainly due to the fact that when a target is sending its small number of contacts are more likely to be receiving than other users. You will need to turn this insight into an algorithm that finds those contacts.

## TASK Q1 and Q2 -- Answer the questions with reference to the code you wrote.

- Please include these as part of the Code file submitted, as a multi-line string, where the `TODO` indicates.






