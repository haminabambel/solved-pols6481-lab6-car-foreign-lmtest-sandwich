Download Link: https://assignmentchef.com/product/solved-pols6481-lab6-car-foreign-lmtest-sandwich
<br>
The objectives are to practice methods for diagnosing and resolving the problem of non-constant error variance, also known as heteroskedasticity. There exist other, simpler methods for both tasks, but they were designed for (and are most easily implemented in) bivariate regression; we will focus instead on methods that are applicable to multivariate regression.

<ol>

 <li><strong> Datasets</strong>: <em>abortions.dta</em>, <em>populations.dta</em></li>

</ol>

<strong>III. Packages</strong>:  <em>car</em>, <em>foreign</em>, <em>lmtest</em>, <em>sandwich</em>

<ol>

 <li><strong> Preparation</strong></li>

</ol>

<ul>

 <li>Open RStudio by double-clicking the icon or selecting RStudio from the Windows Start menu.</li>

 <li>Open the project from the Projects menu</li>

 <li>Pull from Github for any updates</li>

 <li>Navigate in the lower right file area to the Lab 3 section</li>

 <li>5) Open the R script by typing <em>Ctrl</em>+<em>O</em> or by clicking on File in the upper-left corner, using the dropdown menu, and navigating to the script in your working directory.</li>

</ul>

7) Run lines 1 through 11 to load 5 libraries you need. The packages should already be installed from previous labs, but if any of them are not, you can uncomment the appropriate code or use the <em>install.packages()</em> command to install them.




<ol>

 <li><strong> Instructions for Lab 06</strong></li>

</ol>




In this lab, you will explore states’ abortion rates (number performed per 1,000 women between the ages of 15 and 44) in 1992. Begin by loading the datasets by running lines 13 or typing the following lines. If you have previously installed “here,” run line 1 of the script, R should find the data automatically in the data subdirectory of your updated project folder:

abdta&lt;-read.dta(here(“data”,”abortions.dta”))

popdta&lt;-read.dta(here(“data”,”populations.dta”))




The dependent variable is named <em>abortion</em>.




There are four explanatory variables:

<em>price</em> (average price of a procedure in non-hospital facilities),

<em>income</em> (dollars per capita in 1992),

<em>picket</em> (percent of survey respondents who reported experiencing protests at abortion clinics that involved physical contact or blocking of patients), and

<em>funds</em> (a dummy variable, = 1 if state health funds can pay for abortion procedures).




Run line 16 to regress states’ abortion rate on the four explanatory variables. Then, write the coefficients and standard errors in the space provided on page 6.

&gt; base&lt;-lm(abortion~price+income+picket+funds, data=abdta)

&gt; summary(base)

<strong> </strong>

<strong> </strong>

<ol>

 <li><strong> Diagnosing Heteroskedasticity</strong></li>

</ol>




Run line 17 to generate the added variable plots, which illustrate the estimated relationships between each explanatory variable and the dependent variable:

&gt; avPlots(base, pch = 19)

Added variable plots might provide some insight into heteroskedasticity on a variable-by-variable basis. Does the residuals’ <u>variance</u> appear to be systematically related to any of these variables?




Line 18 plots the residuals versus fitted values (this is the first of four plots generated by typing plot(base) and then typing &lt;return&gt;); you might examine the plot for any deviation from an even, linear pattern.




Run line 19 to generate residuals (<em>uhat</em>) and fitted values (<em>yhat</em>) and add them to the data frame:

&gt; abdta$uhat &lt;- base$residuals; abdta$yhat &lt;- base$fitted.values




I included code for some optional plots: line 20 plots predicted value of abortion rates against income, and line 21 plots the residual against income. These can be useful if you believe one explanatory variable, such as income, is the primary source of heteroskedasticity.

&gt; plot(abdta$income, abdta$yhat); abline(lm(yhat~income, abdta))

&gt; plot(abdta$income, abdta$uhat); abline(h=0, col=”red”)




Run lines 24-25  to perform the Breusch-Pagan test, which performs <u>one</u> auxiliary regression of the squared residual on all four explanatory variables. The first step is to generate the squared residuals (line 24); we then estimate the linear model (line 25). Examine the F statistic which is shown if you use the summary(bp) command after running the regression model.

&gt; abdta$uhatsq &lt;- abdta$uhat^2

&gt; bp&lt;-lm(uhatsq ~ price + income + picket + funds, abdta)

&gt; summary(bp)




We could also use either a “score statistic,” a.k.a. the LM test statistic, which is distributed chi-squared and is the product of N and the R<sup>2</sup> from this regression (and is computed in lines 28-29)

&gt; rsq &lt;- summary(bp)$r.squared

&gt; nrsq = NROW(abdta$uhat)*rsq




The LM test statistic follows a chi-square distribution with degrees of freedom equal to the number of explanatory variables, so you can use the following two lines of code (lines 30-31) to save the product of N and <em>R</em><sup>2</sup> and then to perform the chi-square test.

&gt; dfbp = bp$rank-1

&gt; pchisq(nrsq, dfbp, ncp=0, lower.tail=FALSE, log.p=FALSE)




Line 34 provides code for a shortcut method of performing the Breusch-Pagan test; is it reporting the LM statistic or the F statistic?

&gt; bptest(base)

The null hypothesis that we are seeking to reject with the Breusch-Pagan test is that the errors are homoskedastic. Based on the results of the test, do we reject (<em>p</em> &lt; .05) or retain (<em>p</em> ³ .05) the null hypothesis?




Line 35 performs a similar test – the Cook-Weisberg Test – which you can learn a little more about by typing <em>?ncvTest</em> in the console window.

&gt; ncvTest(base)

Run lines 38-42 to perform Wooldridge’s special form of White’s test, which regresses the squared residual on  and <sup>2</sup>.  The test statistic is the product of N and the R<sup>2</sup> from this regression (i.e., the LM statistic), but you can also examine the F statistic to see whether <em>p</em> &lt; .05.

&gt; abdta$yhatsq = (abdta$yhat)^2

&gt; whitetest&lt;-lm(uhatsq ~ yhat + yhatsq, abdta); summary(whitetest)

&gt; nr2 = NROW(whitetest$residuals)*summary(whitetest)$r.squared

&gt; dfwhite = whitetest$rank-1

&gt; pchisq(nr2, dfwhite, ncp=0, lower.tail=FALSE, log.p=FALSE)




Although the F-statistic allows you to reject the null hypothesis of homoscedasticity, neither coefficient for <em>yhat </em>or for <em>yhatsq </em>is statistically significant. Why do you think that might be? After pondering this question, check for multicollinearity (between <em>yhat </em>and <em>yhatsq</em>).

<strong> </strong>

Open the R script “white-test.R” and select every line of code, from line 1 to line 65. Run the script, then return to the R script “Lab06.R” where lines 39–40 perform the original version of White’s test. To perform the original version of the test, you regress the squared residual on every explanatory variable, the square of every explanatory variable, and the interaction of every explanatory variable with every other explanatory variable. Even with only four explanatory variables, this test consumes 14 degrees of freedom (with an N of just 50).

&gt; white.test(base)

&gt; rm(white.test)




Is the <em>p</em> value under the .05 threshold?




<ol>

 <li><strong> Resolving Heteroskedasticity</strong></li>

</ol>




In the simplest form of the Weighted Least Squares method for resolving heteroskedasticity, one theorizes that <em>one</em> explanatory variable is interfering with homoscedasticity, and then one divides all variables – independent and dependent – by the square root of that variable. This process makes the most sense when the error has a variance proportional to <em>one</em> explanatory variable. For example, when we are using aggregate data collected from geographic units of varying populations, the error variance may be inversely proportional to population size.




Run line 49 to merge two datasets – the active dataset plus one with states’ populations as measured in the 1990 census. Line 50 divides states’ populations by 1000, so that the scale is the same as the dependent variable.

&gt; mdata &lt;- merge(abdta, popdta, by=”state”)

&gt; mdata$pop1990k &lt;- mdata$pop1990/1000




Line 51 plots the residual against states’ populations; does the error variance appear larger for the states with smaller populations? (California and New York, the two largest states, also have high residuals, but the remaining states’ residuals do seem to fit a nice ‘cone’ pattern.)

&gt; plot(mdata$pop1990k, mdata$uhat, pch=19); abline(h=0, col=”red”)




Run lines 54-66  to perform ‘Feasible Generalized Least Squares’ the <em>loooooong</em> way. First, lines 54-55 generate the squared population and squared residuals, and line 56  regresses the squared residuals on population and squared population.

&gt; mdata$pop1990ksq &lt;- mdata$pop1990k^2

&gt; mdata$uhatsq &lt;- (mdata$uhat)^2

&gt; fglsmod &lt;- lm(mdata$uhatsq ~ mdata$pop1990k + mdata$pop1990ksq)




Lines 57-58 generate the weights (<em>w</em>) as the square root of the predicted residuals:

&gt; mdata$wsq &lt;- predict(fglsmod)

&gt; mdata$w &lt;- sqrt(mdata$wsq)

<strong> </strong>

Next, the constant and all the variables must all be divided by the weight (lines 59-64):

&gt; mdata$one_w &lt;- 1/mdata$w

&gt; mdata$abortion_w &lt;- mdata$abortion/mdata$w

&gt; mdata$price_w &lt;- mdata$price/mdata$w

&gt; mdata$income_w &lt;- mdata$income/mdata$w

&gt; mdata$picket_w &lt;- mdata$picket/mdata$w

&gt; mdata$funds_w &lt;- mdata$funds/mdata$w




Finally, run the regression on the weighted variables (line 59 or line 60). You must modify the command to ensure that the constant is dropped, because we replace the constant with 1/<em>w</em>.

&gt; fglsmod.1 &lt;- lm(abortion_w ~ one_w + price_w + income_w + picket_w + funds_w – 1, data=mdata)

&gt; fglsmod.2 &lt;- lm(abortion_w ~ 0 + one_w + price_w + income_w + picket_w + funds_w, data=mdata)

<strong> </strong>

Earlier I asked you to write down the coefficients and standard errors from the ordinary least squares regression; write down these values for the weighted least squares regression on page 6.




Line 69 provides a simpler way to perform Feasible Generalized Least Squares by using the analytic weights function:

&gt; wlsmod &lt;- lm(abortion ~ price + income + picket + funds, data=mdata, weights = 1/wsq)

The coefficients should be identical to those from the long version of WLS, so you could have skipped directly from line 52 to line 64. (But, where’s the fun in that?)




Notice, however, the measures of model fit are very different, because we had changed the dependent variable to abortion_w in line 60 and lines 65-66, whereas we use the original dependent variable, abortion, in line 16 and in line 69.




Lines 72-76  run Wooldridge’s version of WLS, which uses the log of the squared residuals in the process of generating the weights. Note that I could have shortened the code by one line (line 69) by using 1/mdata$woolfit as the weights in line 70.

&gt; mdata$luhatsq &lt;- log(mdata$uhatsq)

&gt; wooldridge &lt;- lm(luhatsq ~ price + income + picket + funds + pop1990k + pop1990ksq, data=mdata)

&gt; mdata$woolfit &lt;- exp(wooldridge$fitted.values)

&gt; mdata$wool.w = 1/mdata$woolfit

&gt; wooldridge.wls &lt;- lm(abortion~price+income+picket+funds, data=mdata, weights=mdata$wool.w)

Write down the estimated coefficients and standard errors from the model wooldridge.wls on page 6; how do the results of this model compare to the results shown after line 63?







Run lines 79 and either line 80 or line 81 to run a regression with heteroskedasticity-robust standard errors, and write down the coefficients and the standard errors in the appropriate space on page 6. Compare the coefficients, standard errors, and <em>t</em> statistics to the earlier results.

&gt; lm.mod &lt;- lm(abortion ~ price + income + picket + funds, mdata)

&gt; robse&lt;-vcovHC(lm.mod, type=”HC1″); coeftest(lm.mod,robse)




A slightly different version of this test uses the <em>hccm</em>() command in the <em>car</em> package:

&gt; robust&lt;-hccm(lm.mod); coeftest(lm.mod,robust)




You can compare the variance-covariance matrices of the estimators by typing their names; in order to round to four decimal points (for easier viewing), you can type the following:

&gt; round(vcov(lm.mod), digits=4)

&gt; round(robse, digits=4)

&gt; round(robust, digits=4)

Remember that the estimated variances of the estimated coefficients are on the main diagonal. Does any version of the variance-covariance matrix contain systematically larger variances?

Does any version of the variance-covariance matrix contain systematically smaller variances?




Finally, run lines 84-85  to regress the natural log of the abortion rate on the same four variables. The topic we are taking up next Thursday is nonlinear models, of which the log-linear model (which uses the natural log of <em>y </em>as the dependent variable, but maintains all <em>x </em>in their original form) is one variation.

<strong> </strong>

Plot the residuals against the fitted values using line 86, and plot the residuals against population (which was <u>not</u> among the regressors) using line 87.. After examining the plot, conduct a test of heteroskedasticity using one of the commands or procedures in part A of the lab; one option is shown in line 88. Does transforming the dependent variable resolve the heteroskedasticity problem?













Space is provided on the here for you to write down some estimates of the coefficients and standard errors, based on four different methods for estimating the multiple regression model.




Ordinary Least Squares estimates:




<table>

 <tbody>

  <tr>

   <td width="193"><strong><em>explanatory variable</em></strong></td>

   <td width="192"><strong><em>coefficient</em></strong></td>

   <td width="190"><strong><em>standard error</em></strong></td>

  </tr>

  <tr>

   <td width="193"><em>constant</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>price</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>income</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>picket</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>funds</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

 </tbody>

</table>




Weighted Least Squares estimates (main method in line 59, 60, or 63):




<table>

 <tbody>

  <tr>

   <td width="193"><strong><em>explanatory variable</em></strong></td>

   <td width="192"><strong><em>coefficient</em></strong></td>

   <td width="190"><strong><em>standard error</em></strong></td>

  </tr>

  <tr>

   <td width="193"><em>constant</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>price</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>income</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>picket</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>funds</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

 </tbody>

</table>




Weighted Least Squares estimates (another method in line 70):




<table>

 <tbody>

  <tr>

   <td width="193"><strong><em>explanatory variable</em></strong></td>

   <td width="192"><strong><em>coefficient</em></strong></td>

   <td width="190"><strong><em>standard error</em></strong></td>

  </tr>

  <tr>

   <td width="193"><em>constant</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>price</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>income</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>picket</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>funds</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

 </tbody>

</table>




Ordinary Least Squares estimates with robust standard errors (either line 74 or line 75):




<table>

 <tbody>

  <tr>

   <td width="193"><strong><em>explanatory variable</em></strong></td>

   <td width="192"><strong><em>coefficient</em></strong></td>

   <td width="190"><strong><em>standard error</em></strong></td>

  </tr>

  <tr>

   <td width="193"><em>constant</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>price</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>income</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>picket</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

  <tr>

   <td width="193"><em>funds</em></td>

   <td width="192"> </td>

   <td width="190"> </td>

  </tr>

 </tbody>

</table>


