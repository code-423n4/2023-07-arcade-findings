I'm busy forcing myself to focus on multiple codebases per day in order to fasttrack my learning journey and progress. So I used this audit contest as a speedrun to help me improve my speed when auditing.

Approach was to go through the codebase first all the in scope contracts quick overview pass, and then go over again but dive deeper. Went too slow in the beginning, and had to rush on last day. Good practice for improving my speed though.

Yes, learned quite a bit about governance and especially NFT based voting power boosting, as well as how well the protocol protects against a rogue user registering multiple delegatees successfully, or registering both himself and delegatee. The checks prevent this 100%. Trust me, I checked thoroughly.

My findings are from hurrying through the smart contracts the last couple hours before deadline.

I thought I found an attack vector where the msg.sender user could register multiple delegatees using different addresses, but the immutable storage slot PER user/msg.sender is a formidable preventative measure against that, as well as the check where if registration.delegatee == address(0) then make it equal to msg.sender address(the user calling the function), otherwise make registration.delegatee = _delegatee. And then the check which checks if registration.delegatee != address(0) means the user has already registered, so not allowed unless he changes the delegatee.

Wish I had more time to hunt for bugs here.

### Time spent:
16 hours