
**Idea:** we want to **generate random variables** that follow a certain CDF, given that CDF
- pretty much, if we have the CDF, we can generate our own random data that will follow it

CDF vs PMF: 
- PMF is the probability mass function, aka rolling a 1 in a dice probability is 1/6, PMF is 1/6 for each value of the dice
- CDF is cumulative, aka sum of probability of a variable plus the ones before it
	- EX: rolling 1 in dice = 1/6, CDF of 2 is 2/6, 3=3/6 ... etc
		- CDF will look like this: P(a){
			- 0, a $\epsilon$ (-$\infty$, 1)
			- 1/6, a $\epsilon$ \[1, 2)
			- 2/6, a $\epsilon$ \[2, 3)
			- 3/6, a $\epsilon$ \[3, 4)
			- 4/6, a $\epsilon$ \[4, 5)
			- 5/6, a $\epsilon$ \[5, 6)
			- 1, a $\epsilon$ \[6, $\infty$)
			- }
		- note that in CDF the bounds will always be 0 to 1

**Inverse transform method:** by finding the inverse of the CDF, we will be able to get the variable value, given the probability, aka we inverse 1/6 with \[1, 2) etc
- aka probability will be the input and the variable value will be the output
- steps for the inverse transform method:
	1) Find the CDF, aka F(a)
	2) Invert the CDF, aka G(u) aka $F^{-1}(u)$
	3) Draw from U ~ Unif(0,1)
		- Unif is just a uniform distribution, aka draw a number 0 to 1 where each number is equally likely
		- we will need to generate this random number ourselves, aka use numpy.random
	4) Input the Unif draw into the inverted CDF, aka G(U)
	
*** IMPORTANT: some distributions are NOT invertible


Two ways of using inverse transform method:
- For **discrete distributions**:
	- aka output must be a discrete value (ex 2 or a 3, no 2.5) (like in a dice roll)
```python
def ivt():

	F = 1/6 # initial probability mass
	p = 1/6 # probability mass as a function of a (fixed in this instance)
	a = 1 # outcome in the sample space
	U = np.random.uniform(0, 1) # Standard Uniform Draw
	
	while U > F: # Incrementing the CDF by the PMF until probability threshold is met
	F += p
	a += 1
	
	return a
print(ivt())
```
- explanation: 
	- F is the initial pmf, for our case its 1/6 since that is the lowest probability in our CDF
	- P is the pmf, for our case its fixed to 1/6, each step in our CDF increases by 1/6
		- this can be changed to something else, depending on the situation. it can be a brownian probability, binomial, etc
		- it can be combined with a and show dynamic probability steps
	- a is the outcome, aka the output of our inverse transformation method
	- U is the draw from a uniform distribution between 0 to 1
	- while loop explanation:
		- while U > F: while our U draw is greater than PMF, we will increment F by P and A by the next value until the correct CDF value is met
		- note that since U cannot be greater than 1, the most this loop is going to run is 5 times
		- EX: if U is 0.60, the while loop will be as follows:
			- 0.60 > 0.166, f-> 0.33 a-> 2
			- 0.60 > 0.33, f-> 0.5 a-> 3
			- 0.60 > 0.5, f-> 0.66 a->4
			- 0.6 ~~>~~ 0.66, so the function returns a = 4
			- if we look at our CDF 4 is between 0.5 and 0.66 so the return is correct

- for **continuous distributions**:
	- continuous means it can be any value, int or decimal between the set of values
		- EX 1.56897, 2.34556, etc
	- continuous distributions are generally defined by an equation, using the inverse transform method, all we have to do is find the inverse of that equation and plug in our U
	- Ex CDF of an exponential distribution is: $F(x) = 1-e^{-\lambda x}, x \ge 0$ 
	- Inverse: $u = 1-e^{-\lambda x} \rightarrow e^{-\lambda x} = 1-u \rightarrow -\lambda x = ln(1-u) \rightarrow x= F^{-1}(u)=-\frac{ln(1-u)}{\lambda}$ 
```python
def inverse_cdf_exponential(u, lam):
	return -np.log(1-u) / lam
print(inverse_cdf_exponential(0.60, 1.0))
```
