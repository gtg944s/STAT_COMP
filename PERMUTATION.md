## Permutation tests

In previous classes we discuss how to estimate type-I error rate using simulations. 
However, simulating data requires making assumptions that may or may not hold. Here, we will c use permutations to esitmate the p-valuethershold we should achieve a given type-I erro rate. Permutation analysis is done using real data; thus, there is no need to make any assumptions.

The goal in a permutation analysis is to estimate the distirbution of a test statistic under the null hypothesis. To simulate data under the null hypothesis we break the association between response and predictor by permuting the response (or in some cases the predictor). 

In the following example we use the [gout data set](https://github.com/gdlc/STAT_COMP/blob/master/goutData.txt) to approximate the distribution of an statistic (the absolute value of the z-statistic for age, in a model where we regress serum urate on race sex and age).

```r

DATA=read.table("~/Desktop/gout.txt",header=T)

nPerm=10000 # number of permuations
z_stat=rep(NA,nPerm)
n=nrow(DATA)
TMP=DATA

for(i in 1:nPerm){
	# permute
	  TMP$age=sample(DATA$age,size=n) # permuting age while keeping the other variables un-touched.
	# estimate
	  fm=lm(su~race+sex+age,data=TMP)	  
	# store the test-statistic
	 z_stat[i]=summary(fm)$coef[4,3]
}

hist(z_stat,30)

plot(density(z_stat),main="Permutation distribution of a z-statistic")
x=seq(from=-4,to=4,length=100)
lines(x=x,y=dnorm(x,mean=0,sd=1),col=4,lty=2) # as expected, with this sample size, the z-statistic follows approximately a N(0,1)

```

**Empirical (permutation-based) p-value**
```r

 fm=lm(su~race+sex+age,data=DATA)
 summary(fm)$coef
 
 2*mean(z_stat>summary(fm)$coef[4,3])
```

[In-class 6](https://github.com/gdlc/STAT_COMP/blob/master/INCLASS_6.md)


### Multiple testing

Consider the following logistic regression


```r
 DATA=read.table("~/Desktop/gout.txt",header=T)
 DATA$gout=ifelse(DATA$gout=='Y',1,0)

 # fitting the model without doing any permutation
 fm0=glm(gout~su+race+sex+age,data=DATA,family='binomial')
```

Suppose we want to test H0: neither sex nor race have an effect. We can perform this test using a liklihood ratio test

```r
 fm00=glm(gout~su+age,data=DATA,family='binomial')
  
 LRT_stat=logLik(fm0)-logLik(fm00)
 pchisq(df=2,q=2*LRT_stat,lower.tail=FALSE)
  
 # or use...
  anova(fm0,fm00,test='Chisq')
```
**Pemrutation-based pvalue for multiple tests**


```r
DATA=read.table("~/Desktop/gout.txt",header=T)
DATA$gout=ifelse(DATA$gout=='Y',1,0)
# fitting the model without doing any permutation
 fm0=glm(gout~su+race+sex+age,data=DATA,family='binomial')
 fm00=glm(gout~su,data=DATA,family='binomial')
 
 nRep=10000
 LRT_stat=rep(NA,nRep)
 n=nrow(DATA)
 
 
 for(i in 1:nRep){
   TMP=DATA
   
   tmp=sample(1:n,size=n,replace=F)
   
   # shuffling data
   TMP$age=DATA$age[tmp]
   TMP$race=DATA$race[tmp]
   TMP$age=DATA$age[tmp]
   
   tmp=glm(gout~su+race+sex+age,data=TMP,family='binomial')
   LRT_stat[i]=2*(logLik(tmp)-logLik(fm00))
 }

mean(LRT_stat> 2*(logLik(fm0)-logLik(fm00)))

anova(fm00,fm0,test='Chisq')

 
```

[Back](https://github.com/gdlc/STAT_COMP/)
