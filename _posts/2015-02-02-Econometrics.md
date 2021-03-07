---
layout: post
title: "Econometrics - Hill-Burton Act"
description: "Investigated the allocation of Hill-Burton funding across states."
categories: [Econometrics]
tags: [Stata, Health Econometrics]
redirect_from:
  - /2015/02/02/
---

## Econometrics
The Hill-Burton program, a large-scale hospital construction program that started in the 1940s, studies the nature of competition in the hospital industry. To investigate whether the built state-level allocation formula was predictive of actual funding allocations, I analyzed the following datasets: 
- [Per capita personal income for 1943-1962](https://github.com/SFL09/SFL09.GITHUB.IO/files/6096404/Cap.txt);
- [Population for 1947-1964](https://github.com/SFL09/SFL09.GITHUB.IO/files/6096406/Pop.txt);
- [Hill-Burton Project Register data](https://github.com/SFL09/SFL09.GITHUB.IO/files/6096407/HB.txt).


### The Workflow of Data: 
First, I created a balanced state-year panel data set for 1947-1964 for 48 states (excluding Alaska, Hawaii, and Washington DC) with three variables:
- stateyear: state-year identifier variable of the form Alabama 1947;
- predicted: predicted federal Hill-Burton funds to be allocated in that state-year;
- hbfunds: actual federal Hill-Burton funds allocated in that state-year.

```Stata
use E:\L\hbpr.dta, clear
use E:\L\pinc.dta, clear
use E:\L\pop.dta, clear
use E:\L\hbprt.dta, clear

use E:\L\pinc.dta, clear
merge 1:1 areaname using E:\L\pop.dta, nogen
foreach v of varlist pcinc1943-pop1964 {
   cap destring `v', force replace
}
tab areaname
cap drop if areaname=="Alaska" | areaname=="District of Columbia" | areaname=="Far West 3/"  ///
    | areaname=="Great Lakes" | areaname=="Hawaii 3/" | areaname=="Mideast" | areaname=="New England"   ///
	| areaname=="Plains" | areaname=="Rocky Mountain" | areaname=="Southeast" | areaname=="Southwest"  ///
    | areaname=="United States"            

reshape long pcinc pop, i(areaname) j(year)
drop fips

describe
save E:\L\pinc&pop_temp.dta, replace

use E:\L\hbpr.dta, clear
keep state year hillburt*
cap drop if state=="Alaska" | state=="American Samoa" | state=="Dist of Col" | state=="Guam"  ///
    | state=="Hawaii" | state=="Puerto Rico" | state=="Virgin Islands"
rename hillburt* hbfunds
replace hbfunds=subinstr(hbfunds, ",", "", .)    
destring hbfunds, force replace
replace year=1900+year
collapse (sum) hbfunds, by(state year)
label variable hbfunds "actual federal Hill-Burton funds allocated"
format hbfunds %12.0f
drop if year>1964
describe
tab state
merge 1:1 state year using E:\L\pinc&pop_temp.dta, nogen

merge m:1 year using E:\L\hbprt.dta, nogen

drop if year>1964
save E:\L\rawpanel.dta, replace
```

### Hypothesis Testing & Regression Analysis:
Next, I assessed whether the minimum (0.33) and maximum (0.75) values are empirically relevant in practice and whether the predicted state allocations are a good predictor of actual federal Hill-Burton funding allocations.

```Stata
use E:\L\rawpanel.dta, clear
encode state, gen(stid)
order state stid year
tsset stid year
xtdes

************************************************************************************************
* Calculate a “smoothed” state per capita income by averaging over the three most recent years 
for which data is available. According to the Federal Register, in 1947 the three years of data 
used were 1943, 1944, and 1945.                                                                *
************************************************************************************************
gen pcinc_sm=(L.pcinc+L2.pcinc+L3.pcinc)/3 if year>=1948
replace pcinc_sm=(L2.pcinc+L3.pcinc+L4.pcinc)/3 if year==1947
replace pcinc_sm=(L.pcinc_sm+L2.pcinc+L3.pcinc)/3 if year==1964

************************************************************************************************
* Calculate a national average of the smoothed state per capita income variable by year,
excluding Alaska, Hawaii, and Washington DC from the average calculation.                      *
************************************************************************************************
bys year: egen pcinc_av=mean(pcinc_sm)

************************************************************************************************
* Calculate an “index number” for each state*year, equal to the state’s smoothed per capita
income in that year as a fraction of the national average of the smoothed state per capita
incomes in that year.                                                                          *
************************************************************************************************
gen index=pcinc_sm/pcinc_av

************************************************************************************************
* Calculate an “allotment percentage” for each state*year, equal to 1 - 0.5*(index number
for state s).                                                                                  *
************************************************************************************************
gen all_per=1-0.5*index
sum all_per
histogram all_per
sort state year
list state all_per if year==1947   

************************************************************************************************
* Replace values of the allotment percentage variables for each year that are below the
minimum (0.33) or above the maximum (0.75).                                                    *
************************************************************************************************
replace all_per=0.33 if all_per<0.33
replace all_per=0.75 if all_per>0.75

************************************************************************************************
* Calculate a “weighted population” for each state*year, equal to the allotment percentage
squared multiplied by the state’s population in that year.                                     *
************************************************************************************************
gen pop_w=pop*all_per^2

************************************************************************************************
* Calculate a state allocation share for each state*year, equal to the weighted population
divided by the sum of weighted populations for all states.                                     *
************************************************************************************************
bys year: egen pop_sum=sum(pop_w)
gen all_share=pop_w/pop_sum

************************************************************************************************
* Calculate a state*year predicted Hill-Burton allocation as state allocation share multiplied
by the total federal Hill-Burton appropriation for that year.                                  *
************************************************************************************************
gen predicted=hbprt*all_share
label variable predicted "predicted federal Hill-Burton funds to be allocated"
format predict %12.0f
sum predicted

************************************************************************************************
* Replace the predicted allocations according to a $100,000 minimum in 1948 and $200,000
minimum for all later years.                                                                   *
************************************************************************************************
replace predicted=100000 if predicted<100000 & year==1948    // 2 values changed
replace predicted=200000 if predicted<200000 & year>1948     // 32 values changed

keep stid state year hbfunds predicted
sort state year

pwcorr hbfunds predicted
reg hbfunds predicted
eststo ols_1
reg hbfunds predicted, nocons
test predicted==1
eststo ols_2
esttab *, mtitles b(%7.3f) t(%7.3f) wide nopa sca(N r2 r2_a F p)

scatter hbfunds predicted || lfitci hbfunds predicted
drop if year<=1946
gen stateyear=state+"_"+strofreal(year)
keep stateyear predicted hbfunds 
order stateyear predicted hbfunds
save E:\L\finaldata.dta, replace
```

### Result
Finally, according to statistical results, the mean is 0.5. The minimum is 0.2782741, and the maximum is 0.7351687. Therefore, the minimum (0.33) and maximum (0.75) allotment percentage values in the state allocation formula could provide the basis for empirical identification. Furthermore, the predicted state allocations and actual federal Hill-Burton funding allocations are highly and positively correlated (0.7879).
