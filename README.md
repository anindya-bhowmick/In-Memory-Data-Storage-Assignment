  
## Programming Challenge Solution


The Programming Challenge required us to complete the findAll() function in the project/src/main/java/com/redislabs/university/RU102J/dao/SiteDaoRedisImpl.java 
  


It has been completed and tested using mvn test -Dtest=SiteDaoRedisImplTest after removing the @Ignore annotations from the respective functions. The test was a success.

## Rate Limiter Solution

In the rate limiter problem, we have to build a rate limiter that limits user calls to 10 calls per minute per user.

### Solution - 

#### Limiting user to 10 calls within fixed 1 minute window intervals :

Here we keep windows of Redis epoch time X to X+1 minute (upper limit is not inclusive) and within one window, one user can only make 10 calls.

Algorithm:
- When an user with user ID - UID makes a call to a server at an epoch time X + t , where 0 <= t < 1 minute, we consider a key User:UID:X .
- We check if the key already exists in the Redis Database or not. (EXISTS User:UID:X)
	- If it exists, we Check if the value corresponding to the key is less than 10 or not ( GET User:UID:X and then compare)
		- If it is less than 10, we increment this value and answer the call with the proper response (INCR User:UID:X and then send response)
		- Else if it is greater than or equal to 10 we do not answer the call and send an error response.
	- If the Key did not previously exist, we create a new Key with expiration time set to X+1 epoch time and with an initial value of 1. We then answer the call with the appropriate response. ( SET User:UID:X 1 EXAT X+1 and send response)
	
	



#### Limiting user to 10 calls within any span of 1 minute :

Here the user can make atmost 10 calls in the span of any one minute. No fixed windows are considered for counting the no. of calls. 

Algorithm:

- When an user with user ID - UID makes a call to a server at an epoch time Y, we consider a key User:UID .
- We check if the key already exists in the Redis Database or not. (EXISTS User:UID)
	- If it exists we first remove elements from the left of the list one by one till we find an element which is greater than Y - 1 epoch time. Then we check the length of the list. 
	(LINDEX User:UID 0 to get leftmost element and LPOP User:UID to remove. LLEN User:UID)
	
		- If the length of the list is equal to 10, we send an error response.
		- If the length of the list ranges from 1- 9 , we push the current epoch time, Y to the right of the List. We send the proper resposnse to the user. ( RPUSH User:UID Y send response)
		- If all elements of the List were removed, we have to proceed as if the Key does not exist and do the instruction that follows, because redis removes empty Lists from Database.
	- If it does not exist we create a List with the given key and push the current Epoch time ,i.e, Y to the right of the List. ( i.e., oldest values are stored on the left, newest added to the right). We then send the proper response to the user. ( RPUSH User:UID Y and send response)
	
