chainMapReduce
==============

This code is trying to implement a friend recommendation algorithm by automatically chaining 2 Hadoop MapReduce jobs. 
The details of this algorithm are described in Stanford Course CS 246, 2014 HW1. Each line in the input text file 
(http://snap.stanford.edu/class/cs246-data/hw1q1.zip) records UserID&lt;tab>FriendID where 
FriendID is a list of user's friend separated by comma. The algorithm will recommend 10 users
with most common friends outside the friendlist for each UserID with a format UserID&lt;tab>RecommendFriendID 
where RecommendFriendID is a list of recommended UserID separated by comma. The recommended UserID will 
be in a decreasing order of the number of mutual friends. If there are recommended users with the same 
number of mutual friends, then output those user IDs in numerically ascending order.



The idea is to use chained MapReduce jobs with the work flow map1-reduce1-map2-reduce2.

1. To implement the chained MapReduce work flow, I use "JobControl" from 
org.apache.hadoop.mapred.jobcontrol.JobControl to make sure the second MapReduce job only runs after 
the first MapReduce job completes. Then I put the JobControl in a Thread. This method can avoid using 
Oozie which is designed to handle various Hadoop workflows.

2. The first MapReduce job is called "jobListFriends". The Mapper aims to construct pairs <userID1, userID2> 
who have the common friend userID where both userID1 and userID2 are from FriendID. Then the Reducer will count 
the number of common friends for each pair <userID1, userID2> and delete those pairs who are already friends. 
The Reducer will output the KeyValue format "userID1<tab>userID2,numberOfCommonFriends" which contains the 
suggested friend userID2 with number of common friends for userID1. This pair design strategy is very critical 
as there are no other ways to solve this problem in a MapReduce way. The idea is shown in the following chart:

userID0<tab>*****,userID1,userID2,*******
-> 
<userID1, userID2>, 1


3. The second MapReduce job is called "jobRecommendFriends". It will map the "userID1<tab>userID2,numberOfCommonFriends" 
to my designed structure "userID1<tab>FriendInformation where FriendInformation has a String variable to store "userID2" 
and a int variable to store "number of common friends". In the Reducer, I construct a comparator CompareRec to sort 
the top 10 friends to recommend as required by the algorithm.

The codes have been attached and some explanations are provided in between lines. 
