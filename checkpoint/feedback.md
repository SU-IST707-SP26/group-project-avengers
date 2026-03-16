- Does your proposal include all of the above mentioned sections? 1/1
- Are your objectives concrete and do you have a clear stakeholder need? 2/2
- Do you have a good data source and have you done a thorough job investigating its provenance and credibility? 1/1
Did you do a thorough job exploring your data 2/2

Note that you could probably log scale your data (duration) to get something very close to a normal distribution, and that would likely improve your results.

> Also - you didn't comment on this, but you should observe the "banding" in your data.  On the y-axis of the PCA, you've got roughly three orange bands, and roughly seven on the x-axis.  This is *important* - when you see these sorts of things that indicates that some sort of discrete or cyclical variable is dominating variance.  If you look at the loadings on your two dimensions, I'm betting you'll see day of week on the x-axis, and possibly trip type (?) on the y.  

> This kind of banding tells us that you're going to need a non-linear model - a tree should do nicely.  Also note that UMAP will likely give you a much better separation than PCA

- Have you done some initial modeling of your problem and do you have some early baseline results? 3/3
- Do you have a clear path forward 1/1

----------------

I think you've done a really nice job with this so far.  I suspect you'll have a pretty successful model when you're done.  You might think about how you make this even more compelling - like, suppose a person know's their going to hail a cab at a given point.  Suppose they have some flexibility about where and when they might travel.  Can you recommend their best option?  Where should they hail a cab (perhaps you can make some inferences here?)

Nice job.

10/10
