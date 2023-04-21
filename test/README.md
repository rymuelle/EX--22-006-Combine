I'm having issues where a background uncertainty in a combine card is not reflected as expected in the final limits where the "postfit uncertainty is only ~5% of the initial uncertainty".


# Overview of datacard:

I have a smoothly falling background and a crystal ball signal (centered around 350 GeV) that I am using to compute Asymptotic Limits in a binned higgs combine card. In this (simplified) example, I introduce two uncertainties:

1. A LnN lumi uncertainty applied to the signal (order a few percent)
2. A lnN background normalization uncertainty of order 20% 

See datacard in 'datacard.txt'

See plot of the shapes in 'back_350.png'


# Overview of issue:


If I run FitDiagnostics for prefit asimov toys (as described here: https://twiki.cern.ch/twiki/bin/view/CMS/HiggsWG/HiggsPAGPreapprovalChecks):

```bash
rm -rf  fitResults_t*
text2workspace.py datacard.txt


combine -M FitDiagnostics -d datacard.root -t -1 --expectSignal 0 --forceRecreateNLL -n _t0
python $CMSSW_BASE/src/HiggsAnalysis/CombinedLimit/test/diffNuisances.py  -a fitDiagnostics_t0.root -g plots_t0.root >> ./fitResults_t0 
cat fitResults_t0
      
      
combine -M FitDiagnostics -d datacard.root -t -1 --expectSignal 1  --forceRecreateNLL -n _t1
python $CMSSW_BASE/src/HiggsAnalysis/CombinedLimit/test/diffNuisances.py  -a fitDiagnostics_t1.root -g plots_t1.root >> ./fitResults_t1 

cat fitResults_t1

```

For the fitResults_t0:

```bash
[rymuelle@lxplus793 test]$ cat fitResults_t0
diffNuisances run on fitDiagnostics_t0.root, at 2023-04-13 20:14:43.702470 with the following options ... {'regex': '.*', 'absolute_values': False, 'show_all_parameters': True, 'format': 'text', 'stol2': 0.5, 'pullDef': '', 'vtol': 0.3, 'stol': 0.1, 'plotfile': 'plots_t0.root', 'skipFitB': False, 'sortBy': 'correlation', 'vtol2': 2.0, 'poi': 'r', 'skipFitS': False}

name                                              b-only fit            s+b fit         rho  approx impact
back_closure_2016                                +0.00, 0.08     ! +0.00, 0.08!     !-0.13!      -0.001
lumi                                             +0.00, 1.00        -0.00, 1.00       +0.00      +0.000
```

...I see "+0.00, 0.08" for the background b-only fit, while I expect "0.00 0.99" or "0.00 1.00" like the lumi uncertainty.

The twiki linked above mentiones:

"The second number is the ratio between the uncertainty after the fit and the one in the model. Values smaller than unity imply that your data is such that information can be gained about this parameter. If that is the case, you should justify the origin of that information gain."

I'm unsure if I should expect this result. I can come up with a weak justification, but I don't feel confident in justifying it since I am not an expert. Can someone help me understand the +0.08, and if I am doing anything wrong?



# most recent test:

125-> narrow peak at 350

350-> Wide peak at 350

125 fit results:

t0:
name                                              b-only fit            s+b fit         rho  approx impact
back_closure_2016                                -0.00, 0.07     ! -0.00, 0.08!     !-0.24!      -0.013
lumi                                             +0.00, 1.00        -0.00, 1.00       -0.00      -0.000


t1:
name                                              b-only fit            s+b fit         rho  approx impact
lumi                                             +0.00, 1.00        -0.00, 1.00       -0.53      -0.025
back_closure_2016                                +0.97, 0.07     ! -0.00, 0.08!     !-0.15!      -0.001



350:

t0:
name                                              b-only fit            s+b fit         rho  approx impact
back_closure_2016                                -0.00, 0.07     ! -0.00, 0.14!     !-0.85!      -1.626
lumi                                             +0.00, 1.00        -0.00, 1.00       +0.00      +0.027


t1:
name                                              b-only fit            s+b fit         rho  approx impact
back_closure_2016                                +0.97, 0.07     ! +0.00, 0.16!     !-0.86!      -0.020
lumi                                             +0.00, 1.00        +0.00, 0.99       -0.21      -0.030


# other test: scale background down

use 350

t0:
name                                              b-only fit            s+b fit         rho  approx impact
back_closure_2016                                -0.00, 0.07     ! +0.00, 0.08!     !-0.13!      -0.001
lumi                                             +0.00, 1.00        -0.00, 1.00       -0.00      -0.000

t1:
name                                              b-only fit            s+b fit         rho  approx impact
lumi                                             +0.00, 1.00        +0.00, 1.00       -0.21      -0.024
back_closure_2016                                +0.14, 0.07     ! +0.00, 0.08!     !-0.09!      -0.001

red:

t0:
name                                              b-only fit            s+b fit         rho  approx impact
back_closure_2016                                +0.00, 0.23     ! +0.00, 0.23!     !-0.10!      -0.055
lumi                                             +0.00, 1.00        +0.00, 1.00       +0.00      +0.000


t1:
name                                              b-only fit            s+b fit         rho  approx impact
lumi                                             +0.00, 1.00        +0.00, 1.00       -0.26      -0.025
back_closure_2016                                +1.24, 0.21     ! -0.00, 0.23!     !-0.05!      -0.001


# one bin test

t0:
name                                              b-only fit            s+b fit         rho  approx impact
back_closure_2016                                -0.00, 0.07     ! -0.01, 1.01!       -1.00     -11.800
lumi                                             +0.00, 1.00        +0.00, 1.00       +0.00      +0.018

t1:
name                                              b-only fit            s+b fit         rho  approx impact
back_closure_2016                                +0.14, 0.07     ! +0.00, 0.71!     *-0.99*      -7.379
lumi                                             +0.00, 1.00        +0.00, 1.01       -0.12      -1.253
