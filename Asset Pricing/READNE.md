# Overview

Asset pricing has always been of fundamental importance to modern financial system. This is especially true when we are talking about pricing derivatives – which grows in popularity ever since created and has surpassed the volume of actual stock market by far. There are already plentiful discussions around the rather challenging task to price options. In the following sessions, we will mainly be focusing on pricing specific kinds of options using Monte Carlo Simulation and Lattice method.

# Methods
### Background

The properties we are including are: 1) Asian Call 2) Asian Put 3) Lookback Call 4) Lookback Put 5) Floating lookback Call 6) Floating lookback Put and 7) American Put options. These options diverge in payoff due to the mechanism used when being exercised are different. Though most of the options are similar in a way that they can only be exercise at maturity (in our case 2 months) except for American Put option which can be exercised at any time. Therefore, American Put option is treated as a special case and pricing strategy for it is implemented separately.

Furthermore, we are given the following information:

- Risk-free rate is 2% annually.

- The stock of interest has a price $100 now with volatility equals 25%.

- All options have a strike price $105.

- Simulation unit time (time interval between periods) is set to 1 week.


### The Monte Carlo Simulation Method

The method of Monte Carlo simulation (MCS) is a method of probability analysis done by incrementally running a set of random variables through a model to determine the different outcomes at each increment or unit step. By using this method, one will be able to determine the range of possibilities as well as their probability of occurrence. In our case, the Monte Carlo method is used to predict the different paths in the stock price caused by its volatility over time.

The simulation was achieved by following steps:

1. Using the present price or the price of a time in the future (St_(j-1)) to estimate (St_j) the price after one unit time (δt). The mathematical formular for this procedure was as follows.
   
   ![image](https://github.com/pengp5781/Financial-Engineering/assets/111671117/6030e655-08ce-422d-9ace-5c63993557e1)

2. Repeat step one to generate a sufficient number of possible paths of the stock price. In this case, step one will be repeated for 250 times, and the simulation results will be stored in memory for option pricing. Below is the visualization of 250 paths:
   
![image](https://github.com/pengp5781/Financial-Engineering/assets/111671117/99937d92-0c55-4e01-8568-2e45de40ee79)

3. The average price of the path (S ̅), the maximum (S_max) and the minimum (S_min) price along the path, and the price at maturity S_T are used to calculate the path-dependent option prices.
4. Find the average option payoff over the 250 sample paths at maturity, and discount this averaged price to find the current price of the option. Also, calculate the sample variance.
  
#### Pricing of the "European - Like" Options

  Results as follwoing:
  
  <img width="338" alt="1b0ba577f8e1af354c8c248cc046abf" src="https://github.com/pengp5781/Financial-Engineering/assets/111671117/5a63026e-2d9e-470f-98c8-d7fe43628352">

#### Pricing of the American Put Options

The pricing of an American option is more challenging than pricing an European option because an American option can be exercised at any time before maturity (of cause the investor can still choose to wait until maturity). The pricing of the American option now becomes a problem of finding the optimal stopping time along the path from [0, T].

One common practice is to use dynamic programming (DP) to form an exercise boundary for the American put option [1]. The algorithm goes backward from maturity, at which the exercise boundary would be the strike price K. Let θj denotes the exercise boundary at the time t_j, therefore the exercise boundary at maturity would be θ_m=K. Due to the time value of money (a positive risk-free rate), the present value of an early exercised option at the time t_j is strictly greater than the present value of the same option but exercised later in time at the same price. Thus, we have θ_j≤θ(j+1) at all t_j before maturity. To facilitate the DP, we first try θj=θ(j+1) as a potential value for the exercise boundary at the time t_j. Then, all the simulated paths will be evaluated at the time t_j, to see if the current stock price St_j of this path is lower than or equal to θj. If so, we would exercise the option at this time and let the payoff be the present value of this path, if not, we would find the optimal exercise time between [t(j+1),t_m] and discount this optimal payoff to find the present value of this path. 

After repeating this process to all the samples, we can find the average present value of the payoff from exercising all paths at this time t_j using the trial θj, and the determined present value will be denoted as P_cap. Then, we will try all St_j < θ__(j+1) as the potential value for θ_j and repeat the previous process to find a set of P_cap. Finally, we will take the θ_j that maximizes P_cap to be the exercise boundary at the time t_j. If we continue this DP from t(m-1) to t_0, we will find the optimal exercise boundary of the American put option.

![image](https://github.com/pengp5781/Financial-Engineering/assets/111671117/daa84b37-ee81-4709-a96e-2eca14c8af58)

Now, we go forward from time t_0 to t_m. By comparing a path to the exercise boundary, we can find the optimal stopping time for this path. In other words, if a path goes below the boundary, we will exercise it at this unit step. In case an optimal stopping time for a path wasn’t found, we will exercise it at maturity. Repeat this process for all paths, and we will be able to find the payoffs of all the paths at their optimal stopping time. Finally, we can repeat what we did in section 3.2.1 to discount these payoffs, and to find the current price of the option by averaging these discounted values.

Results:

The price of the American put option with a 95% CI is 6.6036699307820115 ± 0.8358053154253827

The average optimal stopping time is 8.74 , and the variance for optimal stopping time is 0.4903614457831325

### The Lattice Method

The Lattice model we are using is binomial -- meaning that during the lifespan of the option, we assume the price of stock will only go up or down (therefore only 2 possible direction for each period) by a certain amount at certain possibility but it certainly can be extended to include more diverges with some minor modifications.

The binomial model wil yield price paths similar in the left of following figure. Where the price in each period goes up by u percent at p probability and goes down by d percent at 1-p probability. The probablity here is not the 'actual' probability in real life but a risk-neutral probability that is calculated by risk-free rate and volatility under the risk-neutral assumption. This would allow one to get the appropriate price of derivatives when the market is efficient. The same reasoning goes to u and d as well.

![image](https://github.com/pengp5781/Financial-Engineering/assets/111671117/a99a5bfc-87ae-411a-a0ad-df409732a867)

Pricing procedure for all the options excluding American puts are identical: Firstly, the full price path of 2 months/9 periods is simulated using the model described above. The possibility of each path calculated by mutiplying stage possibilities together is also stored for later use. Secondly, payoff of each option is calculated with each possible path. Lastly, the payoffs are discounted back to current time and a weighted average using path possibility from step 1 is taken over all possible paths to determine the risk-neutral price of each type of options.

However, as mention at the beginning, an American option can be exercised at any point so it needs special treatment. Essentially, it differs from normal options by that we are not only checking for payoffs at last stage but instead we check them at every stage to see if there is an opportunity for early exercising. 

- The Generated paths:

  ![image](https://github.com/pengp5781/Financial-Engineering/assets/111671117/df98f0e4-bea8-40b8-aff8-fbd1380538a5)

#### Pricing of the "European - Like" Options

<img width="262" alt="1693600661(1)" src="https://github.com/pengp5781/Financial-Engineering/assets/111671117/d3846452-b057-4e79-a554-fc5fddaf554e">

#### Pricing of the American Put Options

Current price of an American Put option is:  6.98524


# Conclusion

In terms of performance, both models give quantitatively similar result – all the results by Lattice model are within 95% CI of Monte Carlo Simulation. However, when comparing the pricing for American put option, we notice the Lattice model underestimate the price which is expected due to only limited number of periods is used.

The 2 models differ the most in terms of how randomness is account for – Monte Carlo simulation is in some sense more “practical-based” where randomness is addressed by a known normal distribution and the path generated each time is random and non-replicating just like one would expect in real life. While the Lattice model use expectation to catch the influence from probability and comes to a closed form solution.

Each method has its own strength and weakness. For Monte Carlo simulation, it is easy to implement, and the approximation is already good enough even though we only sampled 250 times. But it also has issues like the dependency on normality assumption, result being only approximation never exact value and the exponentially larger computational and memory cost if the problem increases in size or demands higher accuracy. The Lattice model has a closed-form solution which is both intuitive and theoretically efficient. However, the determination can be a problem when the market is more stochastic, and many assumptions do not hold.

Overall, we argue that both methods are strong candidates to the pricing problem. We also think that they complement each other: One could use Lattice model first to get a quick grasp of what to expect and then do multiple Monte Carlo simulations to support the theory and expand the boundary to reflect realistic scenarios.


