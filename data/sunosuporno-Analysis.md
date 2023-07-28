### Arcade.xyz Analysis

Firstly, thanks to Arcade for giving me this opportunity to audit their contract. This is the first audit contest I properly participated and managed to find some issues which will hopefully make it into the final report.
Coming to the approach in analyzing the codebase, I noticed the codebase has been audited once, so I went through that first. Next the findings of the bots in C4's bot race. As Arcade has forked Element Fi's Council, their documentation for Coundil's use cases led me to miss out on contexts of why some implementations were being done on Arcade's end. That led to reading the documentation not being the very best experience I have ever had. Thankfully, the in-contract documentation was a lot of help. After that, I started going through the whole codebase. 
I feel the code is well written and most places have all the necessary checks in my eyes. The arrangement of the files was also easy to understand and the test coverage is commendable. 
One thing I learned from this codebase is how cleanly revert errors can be managed.

### Time spent:
9 hours