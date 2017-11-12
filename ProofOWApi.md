A standardized api (Like OAuth?) for a service to recieve work from clients (likely as payment). 

Client sends a request.
Service responds with a token and a work assignment. Work assignment could be specified as a range of values to perform a known algorithm on.
Client performs work and polls service with token. These polls could include incremental work product or proofs of work. The service could also have independent methods of verify work. 
Once the service decides enough work has been done by the client it responds to a poll with the results of the original request. 

Nature of work assignments:
    could be a range of values to perform a known computation on. (ex: do this hash on all numbers in this range. proof of work is the hashes, or the first x digits of the hashes.)
    could be to store a file for x ammount of time.
    could be to serve a file to any requesters for x ammount of time.
    could be to verify that someone else is performing their work correctly.
    could be a javascript program to perform arbitrary (sandboxed) actions.
    
Example: a cryptocurrency broker
    User provides the broker with a transaction they want recorded. (request phase)
    The broker assigns the user with a range of values to perform the cryptocurrencies proof of work on. 
    The user submits their work for verification. 
    Once the broker gets enough work from the user (or the transaction makes it into a block) the broker tells the user they can stop working. 
    Alternatively once the user sees their transaction in a block they can stop on their own. 
    
Example: A backup redundancy coop.
    Clients send a request to a service containing the data they want backed up. 
    Service gives the client chunks of data from other clients to store.
    Service stores the clients data with other clients. 
    Service stores gives the client information that allows it to retrieve its data from the other clients. 
    The service also gives clients the task of requesting stored data from other clients and reporting to the server for proof of work purposes.
