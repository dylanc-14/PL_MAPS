


## **Associations of Climate Exposures with Physical Activity among Older Adults with Knee Pain Supplementary Info** 
Dylan Clark<sup>1</sup>, Kushang Patel<sup>1,2</sup>
<sup>1</sup>University of Washington Department of Anesthesiology & Pain Medicine 
<sup>2</sup>Harborview Injury Prevention and Research Center

## Abstract 
There is still a debate within chronic pain research about the degree to which climate affects the pain severity of knee osteoarthritis (OA). However, there may be a confounding association between climate and individual engagement with physical activity (PA) treatments, which are known to reduce OA pain severity. We sought to identify this association between four daily climate variables and  the daily PA in the form of walking activity (step counts) for individuals ≥65 years old with knee OA living in King County as previously collected by the UW Anesthesiology & Pain Medicine’s PACIFIC Study ($n$ = 212) from October 2019 to June 2023. Four daily climate variable measurements– maximum 2-minute Wind Speed (mph), Total Precipitation (in), Average Temperature (F), and AQI– were taken from the NOAA Boeing Field weather station and the US EPA King County AQI monitoring stations over same timeframe as the PACIFIC study period. We asked whether there existed people in the PACIFIC study population whose PA levels have different sensitivities to different weather conditions. A statistical analysis was done to answer this question using linear mixed models (LMMs) in *STATA*. Our model showed that greater wind speed was associated with decreased walking activity ($\beta$ = -31.73, $p$ = 0.005). Worse air quality may also be associated with a decreased walking activity ($\beta$ = -3.84, $p$ = 0.074). There were no associations of air temperature and precipitation with walking activity Moving forward, we hope our analysis of the association between climate and PA levels within the PACIFIC study population can be used as support for new PA treatment plans that are both 1) dependent upon individual exercise preferences and 2) modified to fit a given region’s changing climate.

## Methods

### Linear Regression

Simple linear regression was used to model the relationship between each daily climate exposure and daily step counts in *STATA*. Then, correlation between each daily climate exposure and daily step count was done in *R* using Spearman's rank correlation coefficient $\rho$ [[wiki]](https://en.wikipedia.org/wiki/Spearman's_rank_correlation_coefficient). This rank correlation coefficient is a method used to assess the statistical dependence between two continuous variables. Spearman’s rank coefficient was used because it can assess whether $X$ and $Y$ are related by any monotonic function, which is a step beyond a simple linear regression which can only assess linearity between $X$ and $Y$. Anyways, as my poster shows, there was little correlation between any of the climate exposure variables and step counts which was suprising.

### Multilevel Linear Mixed Model

Following a similar method to Yimer *et. al*, we reasoned that if no association existed on the population level then perhaps there were individual variation within the data (perhaps resulting from differential pain processing) that were not accounted for when using the population-average models from the previous section. A linear mixed model (LMM) model can account for variation across individuals by introducing a “random variable” component into the regression equation which allows for there to be error not just associated with the error term [[wiki]](https://en.wikipedia.org/wiki/Mixed_model)[[STATA doc]](https://www.stata.com/bookstore/pdf/xt_xtmixed.pdf). Basically, a mixed model contains two types of variables called *fixed effects* and *random effects*, with the fixed effects being standard regression coefficients ($\beta$), and the random effects being random variables which can be estimated. The statistics is a bit complicated. The equation we modeled took the following form:

$$Y = X\beta + Z\mu + \epsilon$$

This looks pretty fancy but it is really just a linear equation with this extra "random variable" component. In our model, this is is what each parameter represented ($n$ = number of subjects, $p$ = number of fixed effects, $l$ = number of random effects):

 - $Y$ - **Predictor Variable** which is an $n * 1$ matrix of step counts (`totalsteps`).
 - $X$ - **Fixed Effects Design Matrix** which is usually an $n * p$ matrix where $n$ is number of participants and $p$ is the number of fixed effects. The fixed effects include:
	 - average daily temperature (`average_temp`)
	 - daily maximum two minute wind-speed (`wind_speed`)
	 - daily AQI (`AQI`)
	 - total daily precipitation (`precip`)
	 - subject age (`age`)
	 - subject sex (`sex`)
	 - day of subject's baseline visit (`day_1` ... `day_7`). Each baseline visit was 7 days. This made date a categorical variable rather than continuous.
	 - Global Intercept
-  $\beta$ - **Fixed Effects Vector** - *what we want!* This is the $p * 1$ vector of coefficients that represent the associations between our climate and step counts.
- $Z$ - **Random Effects Design Matrix** -  An $n * l$ matrix corresponding to $\mu$. In this case $l = 1$ since we are only considering a random intercept.
- $\mu$ - **Random Effects Vector** - Randomly sampled variables go into this matrix to account for differences among each `subject_id`
- $\epsilon$ - **Error Vector** which is assumed to be normally distributed and is $n * 1$

This was modeled with the *STATA* command: 
```stata
xi: xtmixed totalsteps i.day precip wind_speed aqi average_temp age i.sex subject_id:
``` 

The estimate was done using maximum likelihood with 3 iterations. 

### Bayesian Linear Mixed Effects Model

Towards the end of the internship, I became further interested in the approach of Yimer *et al.* and decided to try an implement an analogous  Bayesian model for my data. This model used the ``brms`` package in *R*:

```r
# Define the Bayesian multilevel mixed model formula with covariates
formula <-brms::bf(totalsteps~wind_speed+s(date,k=4)+
average_temp+AQI+precip+gender+age+comorbitities+
pain_catastrophizing+(1+average_temp+AQI+precip+wind_speed|subject_id))

# Specify prior distribution for the model including covariates
prior <- prior(normal(50, 10), class = "b", coef = "average_temp") +
         prior(normal(14, 4.75), class = "b", coef = "wind_speed") +
         prior(student_t(1,38,6), class = "b", coef = "AQI") +
         prior(normal(0,3), class = "b", coef = "precip") +
         prior(student_t(3,0,10), class = "sd") +
         prior(normal(0, 10), class = "Intercept")

# Fit the Bayesian multilevel mixed model with covariates using brms()
#number of chains should equal the number of cores
#changed to 4 chains and 4 cores
model <- brms::brm(formula, data = dat, prior = prior,
 chains = 4, init=0,core=4, iter = 8000, autocor=NULL,
 control = list(adapt_delta = 0.85))

# Summary of the model
summary(model)
```
The initial posterior predictive check is [[here]](https://ibb.co/yRJJPpx). I would like to continue working on this model in the future.

## Climate Data Access
[[NOAA ISD]](https://www.ncei.noaa.gov/cdo-web/search?datasetid=GHCND) - NOAA's Daily Summaries Website. Accessed climate daily data from 10/01/2019 to 07/01/2023 corresponding to SEATTLE BOEING FIELD (USW0002434)

[[US EPA Daily AQI]](https://www.epa.gov/outdoor-air-quality-data/air-quality-index-daily-values-report) - Selected `pm_2.5` option and county-wide (King County) daily summary for years 2019 to 2023. The 5 `.csv` files were then appended in STATA.

## Works Cited 
1.  Zhang Y, Jordan JM. Epidemiology of Osteoarthritis. Clinics in Geriatric Medicine. 2010;26(3):355-369. doi:[10.1016/j.cger.2010.03.001](https://doi.org/10.1016/j.cger.2010.03.001)  
2.  Raposo F, Ramos M, Lúcia Cruz A. Effects of exercise on knee osteoarthritis: A systematic review. Musculoskeletal Care. 2021;19(4):399-435. doi:[10.1002/msc.1538](https://doi.org/10.1002/msc.1538)
3.  Fransen M, McConnell S, Harmer AR, Van Der Esch M, Simic M, Bennell KL. Exercise for osteoarthritis of the knee. Cochrane Musculoskeletal Group, ed. Cochrane Database of Systematic Reviews. 2015;2015(1). doi:[10.1002/14651858.CD004376.pub3](https://doi.org/10.1002/14651858.CD004376.pub3)
4. Yimer BB, Schultz DM, Beukenhorst AL, et al. Heterogeneity in the association between weather and pain severity among patients with chronic pain: a Bayesian multilevel regression analysis. PR9. 2022;7(1):e963. doi:[10.1097/PR9.0000000000000963](https://doi.org/10.1097/PR9.0000000000000963)
5.  Ferreira ML, Zhang Y, Metcalf B, et al. The influence of weather on the risk of pain exacerbation in patients with knee osteoarthritis - a case-crossover study. Osteoarthritis Cartilage. 2016;24(12):2042-2047. doi:[10.1016/j.joca.2016.07.016](https://doi.org/10.1016/j.joca.2016.07.016)
6. Ed Hawkins. #ShowYourStripes - Washington Climate 1895-2022.; 2023. [https://tinyurl.com/waclimgraph](https://tinyurl.com/waclimgraph)

