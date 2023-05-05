Download Link: https://assignmentchef.com/product/solved-data606-lab-5b
<br>
If you have access to data on an entire population, say the opinion of every adult in the United States on whether or not they think climate change is affecting their local community, it’s straightforward to answer questions like, “What percent of US adults think climate change is affecting their local community?”. Similarly, if you had demographic information on the population you could examine how, if at all, this opinion varies among young and old adults and adults with different leanings. If you have access to only a sample of the population, as is often the case, the task becomes more complicated. What is your best guess for this proportion if you only have data from a small sample of adults? This type of situation requires that you use your sample to make inference on what your population looks like.

<strong>Setting a seed: </strong>You will take random samples and build sampling distributions in this lab, which means you should set a seed on top of your lab. If this concept is new to you, review the lab on probability.

set.seed(1)

<h1>Getting Started</h1>

<strong>Load packages</strong>

In this lab, we will explore and visualize the data using the <strong><sub>tidyverse </sub></strong>suite of packages, and perform statistical inference using <strong><sub>infer</sub></strong>. Let’s load the packages.

library(tidyverse) library(openintro) library(infer)

<strong>The data</strong>

A 2019 Pew Research report states the following:

To keep our computation simple, we will assume a total population size of 100,000 (even though that’s smaller than the population size of all US adults).

Roughly six-in-ten U.S. adults (62%) say climate change is currently affecting their local community either a great deal or some, according to a new Pew Research Center survey. <strong>Source: </strong><a href="https://www.pewresearch.org/fact-tank/2019/12/02/most-americans-say-climate-change-impacts-their-community-but-effects-vary-by-region/">Most Americans say climate change impacts their community, but effects vary by region</a>

In this lab, you will assume this 62% is a true population proportion and learn about how sample proportions can vary from sample to sample by taking smaller samples from the population. We will first create our population assuming a population size of 100,000. This means 62,000 (62%) of the adult population think climate change impacts their community, and the remaining 38,000 does not think so.

us_adults &lt;- tibble(

climate_change_affects = c(rep(“Yes”, 62000), rep(“No”, 38000))

)

The name of the data frame is <sub>us_adults </sub>and the name of the variable that contains responses to the question <em>“Do you think climate change is affecting your local community?” </em>is climate_change_affects. We can quickly visualize the distribution of these responses using a bar plot.

<table width="632">

 <tbody>

  <tr>

   <td width="632">ggplot(us_adults, aes(x = climate_change_affects)) + geom_bar() + labs(x = “”, y = “”,title = “Do you think climate change is affecting your local community?”) +coord_flip()</td>

  </tr>

 </tbody>

</table>

Do you think climate change is affecting your local community?

<table width="589">

 <tbody>

  <tr>

   <td width="29"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="44"> </td>

  </tr>

  <tr>

   <td width="29"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="44"> </td>

  </tr>

  <tr>

   <td width="29"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="86"> </td>

   <td width="44"> </td>

  </tr>

 </tbody>

</table>

Yes

No

0                                                                  20000                                                              40000                                                60000

We can also obtain summary statistics to confirm we constructed the data frame correctly.

us_adults %&gt;%

count(climate_change_affects) %&gt;% mutate(p = n /sum(n))

## # A tibble: 2 x 3

##           climate_change_affects               n           p

## * &lt;chr&gt;                                              &lt;int&gt; &lt;dbl&gt;

## 1 No  38000 0.38 ## 2 Yes 62000 0.62

In this lab, you’ll start with a simple random sample of size 60 from the population.

n &lt;- 60

samp &lt;- us_adults %&gt;% sample_n(size = n)

<ol>

 <li>What percent of the adults in your sample think climate change affects their local community? <strong><sub>Hint: </sub></strong>Just like we did with the population, we can calculate the proportion of those <strong><sub>in this sample </sub></strong>who think climate change affects their local community.</li>

</ol>

We can see that in the sample 0.667 people think that climate change affects them while 0.333 people think that climate change does not affect them

<ol>

 <li>Would you expect another student’s sample proportion to be identical to yours? Would you expect it to be similar? Why or why not?</li>

</ol>

<h1>Confidence intervals</h1>

Return for a moment to the question that first motivated this lab: based on this sample, what can you infer about the population? With just one sample, the best estimate of the proportion of US adults who think climate change affects their local community would be the sample proportion, usually denoted as <em><sub>p</sub></em>ˆ (here we are calling it <sub>p_hat</sub>). That serves as a good <strong><sub>point estimate</sub></strong>, but it would be useful to also communicate how uncertain you are of that estimate. This uncertainty can be quantified using a <strong><sub>confidence interval</sub></strong>. One way of calculating a confidence interval for a population proportion is based on the Central Limit

Theorem, as <em><sub>p</sub></em>ˆ<em>±</em><em><sub>z</sub><sup>?</sup></em><em><sub>SEp</sub></em><sub>ˆ </sub>is, or more precisely, as

<em>p</em>ˆ<em>±z</em><em>?</em>r<em>p</em><u>ˆ(1 </u><em>−p</em><u>ˆ)</u>

<em>n</em>

Another way is using simulation, or to be more specific, using <strong><sub>bootstrapping</sub></strong>. The term <strong><sub>bootstrapping </sub></strong>comes from the phrase “pulling oneself up by one’s bootstraps”, which is a metaphor for accomplishing an impossible task without any outside help. In this case the impossible task is estimating a population parameter (the unknown population proportion), and we’ll accomplish it using data from only the given sample. Note that this notion of saying something about a population parameter using only information from an observed sample is the crux of statistical inference, it is not limited to bootstrapping.

In essence, bootstrapping assumes that there are more of observations in the populations like the ones in the observed sample. So we “reconstruct” the population by resampling from our sample, with replacement. The bootstrapping scheme is as follows:

<ul>

 <li><strong><sub>Step 1. </sub></strong>Take a bootstrap sample – a random sample taken <strong><sub>with replacement </sub></strong>from the original sample, of the same size as the original sample.</li>

 <li><strong><sub>Step 2. </sub></strong>Calculate the bootstrap statistic – a statistic such as mean, median, proportion, slope, etc. computed on the bootstrap samples.</li>

 <li><strong><sub>Step 3. </sub></strong>Repeat steps (1) and (2) many times to create a bootstrap distribution – a distribution of bootstrap statistics.</li>

 <li><strong><sub>Step 4. </sub></strong>Calculate the bounds of the XX% confidence interval as the middle XX% j knof the bootstrap distribution.</li>

</ul>

Instead of coding up each of these steps, we will construct confidence intervals using the <strong><sub>infer </sub></strong>package. Below is an overview of the functions we will use to construct this confidence interval:

<table width="632">

 <tbody>

  <tr>

   <td width="392">Function</td>

   <td width="240">Purpose</td>

  </tr>

  <tr>

   <td width="392">specify</td>

   <td width="240">Identify your variable of interest</td>

  </tr>

  <tr>

   <td width="392">generate</td>

   <td width="240">The number of samples you want to generate</td>

  </tr>

  <tr>

   <td width="392">calculate</td>

   <td width="240">The sample statistic you want to do inference with, or you can also think of this as the population parameter you want to do inference for</td>

  </tr>

  <tr>

   <td width="392">get_ci</td>

   <td width="240">Find the confidence interval</td>

  </tr>

 </tbody>

</table>

This code will find the 95 percent confidence interval for proportion of US adults who think climate change affects their local community.

samp %&gt;%

specify(response = climate_change_affects, success = “Yes”) %&gt;% generate(reps = 1000, type = “bootstrap”) %&gt;% calculate(stat = “prop”) %&gt;% get_ci(level = 0.95)

## # A tibble: 1 x 2

##          lower_ci upper_ci

##              &lt;dbl&gt;          &lt;dbl&gt;

## 1             0.55          0.783

<ul>

 <li>In <sub>specify </sub>we specify the <sub>response </sub>variable and the level of that variable we are calling a <sub>success</sub>.</li>

 <li>In <sub>generate </sub>we provide the number of resamples we want from the population in the <sub>reps </sub>argument (this should be a reasonably large number) as well as the type of resampling we want to do, which is</li>

</ul>

“bootstrap” in the case of constructing a confidence interval.

<ul>

 <li>Then, we <sub>calculate </sub>the sample statistic of interest for each of these resamples, which is <sub>prop</sub></li>

</ul>

Feel free to test out the rest of the arguments for these functions, since these commands will be used together to calculate confidence intervals and solve inference problems for the rest of the semester. But we will also walk you through more examples in future chapters.

To recap: even though we don’t know what the full population looks like, we’re 95% confident that the true proportion of US adults who think climate change affects their local community is between the two bounds reported as result of this pipeline.

<h1>Confidence levels</h1>

<ol>

 <li>In the interpretation above, we used the phrase “95% confident”. What does “95% confidence” mean?</li>

</ol>

In this case, you have the rare luxury of knowing the true population proportion (62%) since you have data on the

<ol>

 <li>Does your confidence interval capture the true population proportion of US adults who think climate change affects their local community? If you are working on this lab in a classroom, does your neighbor’s interval capture this value?</li>

 <li>Each student should have gotten a slightly different confidence interval. What proportion of those intervals would you expect to capture the true population mean? Why?</li>

</ol>

In the next part of the lab, you will collect many samples to learn more about how sample proportions and confidence intervals constructed based on those samples vary from one sample to another.

<ul>

 <li>Obtain a random sample.</li>

 <li>Calculate the sample proportion, and use these to calculate and store the lower and upper bounds of the confidence intervals.</li>

 <li>Repeat these steps 50 times.</li>

</ul>

Doing this would require learning programming concepts like iteration so that you can automate repeating running the code you’ve developed so far many times to obtain many (50) confidence intervals. In order to keep the programming simpler, we are providing the interactive app below that basically does this for you and created a plot similar to Figure 5.6 on <a href="https://www.openintro.org/os">OpenIntro Statistics, 4th Edition (page 182).</a>

<ol>

 <li>Given a sample size of 60, 1000 bootstrap samples for each interval, and 50 confidence intervals constructed (the default values for the above app), what proportion of your confidence intervals include the true population proportion? Is this proportion exactly equal to the confidence level? If not, explain why. Make sure to include your plot in your answer.</li>

</ol>

<h1>More Practice</h1>

<ol>

 <li>Choose a different confidence level than 95%. Would you expect a confidence interval at this level to me wider or narrower than the confidence interval you calculated at the 95% confidence level? Explain your reasoning.</li>

 <li>Using code from the <strong><sub>infer </sub></strong>package and data fromt the one sample you have (<sub>samp</sub>), find a confidence interval for the proportion of US Adults who think climate change is affecting their local community with a confidence level of your choosing (other than 95%) and interpret it.</li>

</ol>

Figure 1: Confidence Interval samp %&gt;%

specify(response = climate_change_affects, success = “Yes”) %&gt;% generate(reps = 1000, type = “bootstrap”) %&gt;% calculate(stat = “prop”) %&gt;% get_ci(level = 0.99)

## # A tibble: 1 x 2

##          lower_ci upper_ci

## &lt;dbl&gt; &lt;dbl&gt; ## 1 0.5 0.817

We can see that with a confidence interval of 99% that the confidence range is now larger compared to the 95% confidence interval.

<ol>

 <li>Using the app, calculate 50 confidence intervals at the confidence level you chose in the previous question, and plot all intervals on one plot, and calculate the proportion of intervals that include the true population proportion. How does this percentage compare to the confidence level selected for the intervals?</li>

 <li>Lastly, try one more (different) confidence level. First, state how you expect the width of this interval to compare to previous ones you calculated. Then, calculate the bounds of the interval using the <strong><sub>infer </sub></strong>package and data from <sub>samp </sub>and interpret it. Finally, use the app to generate many intervals and calculate the proportion of intervals that are capture the true population proportion.</li>

 <li>Using the app, experiment with different sample sizes and comment on how the widths of intervals change as sample size changes (increases and decreases).</li>

</ol>

Figure 3: Confidence Inteval 90

<ol>

 <li>Finally, given a sample size (say, 60), how does the width of the interval change as you increase the number of bootstrap samples. <strong><sub>Hint: </sub></strong>Does changing the number of bootstap samples affect the standard error?</li>

</ol>