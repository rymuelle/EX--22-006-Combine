# BFF Z' (MuMu)
[CADI](https://cms.cern.ch/iCMS/analysisadmin/cadilines?line=EXO-22-006)
[pub talk](https://cms-pub-talk.web.cern.ch/c/exo/exo-22-006/93)
[twiki](https://twiki.cern.ch/twiki/bin/viewauth/CMS/BFF_ZprimeMuMuAnalysis)

## Combine enviroment details and instructions

1. Branch: CMSSW_10_2_X
2. Tag: v8.2.1 

```bash
export SCRAM_ARCH=slc7_amd64_gcc700
cmsrel CMSSW_10_2_13
cd CMSSW_10_2_13/src
cmsenv
git clone https://github.com/cms-analysis/HiggsAnalysis-CombinedLimit.git HiggsAnalysis/CombinedLimit
cd $CMSSW_BASE/src/HiggsAnalysis/CombinedLimit
git fetch origin
git checkout v8.2.1
scramv1 b clean; scramv1 b # always make a clean build
```

## Included data:
Data is provided for 2016, 2017, 2018 in a csv format. ABCD predictions, signal shapes and systematics, and synthetic data is included. 

## Signal and Control regions (brief overview)
Our analysis uses an ABCD method (electron vs muon, b-tagged vs anti-btagged) for a one jet and a two jet case. Our signal regions are as follows:
1. SR1: OSS DiMuon + one b-jet 
2. SR2: OSS Dimuon + 2 jets (at least one b-jet)
Control regions require no b-jet and/or a dielectron selection, respectively. From these, a transfer fit on anti-b-tagged dielectron to dimuon regions is performed and applied to the b-tagged dielectron region, entirely on data.

## Example Commands
```bash
combine -M AsymptoticLimits datacard.txt
```
### Expected output
``` bash
 <<< Combine >>> 
>>> random number generator seed is 123456
>>> method used is AsymptoticLimits
('opening: ', '2016_shapes_df_input', '.root')
('opening: ', '2017_shapes_df_input', '.root')
('opening: ', '2018_shapes_df_input', '.root')
SimNLL created with 6 channels, 0 generic constraints, 39 fast gaussian constraints, 0 fast poisson constraints, 0 fast group constraints, 
SimNLL created with 6 channels, 0 generic constraints, 39 fast gaussian constraints, 0 fast poisson constraints, 0 fast group constraints, 
SimNLL created with 6 channels, 0 generic constraints, 39 fast gaussian constraints, 0 fast poisson constraints, 0 fast group constraints, 

 -- AsymptoticLimits ( CLs ) --
Observed Limit: r < 0.0164
Expected  2.5%: r < 0.0088
Expected 16.0%: r < 0.0120
Expected 50.0%: r < 0.0161
Expected 84.0%: r < 0.0241
Expected 97.5%: r < 0.0338

Done in 0.04 min (cpu), 0.04 min (real)
```
# Checks
done with a copy of 2016_SRX_BFFZprimeToMuMu_fit_M_125_dbs0p5.txt (combined SR1 and SR2)
t0 and t1 done
combineTool.py not found

4. Create a folder named "Checks" and copy there the following five files: fitResults_t0, fitResults_t1, impacts_t0.pdf, impacts_t1.pdf, and validation.json

* How to get fitResults_t0 and fitResults_t1:

      text2workspace.py datacard.txt
      # Load your physics model with the -P option in case you do not use the standard one

      combine -M FitDiagnostics -d datacard.root -t -1 --expectSignal 0 --rMin -10 --forceRecreateNLL -n _t0
      python $CMSSW_BASE/src/HiggsAnalysis/CombinedLimit/test/diffNuisances.py  -a fitDiagnostics_t0.root -g plots_t0.root >> ./fitResults_t0 

      combine -M FitDiagnostics -d datacard.root -t -1 --expectSignal 1  --forceRecreateNLL -n _t1
      # Increase the rMin value if (rMin * Nsig + Nbackground) < 0 for any channel
      python $CMSSW_BASE/src/HiggsAnalysis/CombinedLimit/test/diffNuisances.py  -a fitDiagnostics_t1.root -g plots_t1.root >> ./fitResults_t1 

* How to get impacts_t0.pdf and impacts_t1.pdf:

      combineTool.py -M Impacts -d datacard.root -t -1 --expectSignal 0 --rMin -10 --doInitialFit --allPars -m 1 -n t0
      combineTool.py -M Impacts -d datacard.root -t -1 --expectSignal 1 --rMin -10 --doInitialFit --allPars -m 1 -n t1

      combineTool.py -M Impacts -d datacard.root -o impacts_t0.json -t -1 --expectSignal 0 --rMin -10 --doFits -m 1 -n t0 --job-mode condor --task-name t0 --sub-opts '+JobFlavour = "espresso"\nrequirements = (OpSysAndVer =?= "CentOS7")'
      combineTool.py -M Impacts -d datacard.root -o impacts_t1.json -t -1 --expectSignal 1 --rMin -10 --doFits -m 1 -n t1 --job-mode condor --task-name t1 --sub-opts '+JobFlavour = "espresso"\nrequirements = (OpSysAndVer =?= "CentOS7")'

...wait until the condor jobs finish...

        combineTool.py -M Impacts -d datacard.root -m 1 -n t0 -o impacts_t0.json
        combineTool.py -M Impacts -d datacard.root -m 1 -n t1 -o impacts_t1.json

        plotImpacts.py -i  impacts_t0.json -o  impacts_t0
        plotImpacts.py -i  impacts_t1.json -o  impacts_t1

* How to get validation.json

       ValidateDatacards.py datacard.txt
  
5. Commit your changes to your fork
6. Go to Settings/Members on your fork's webpage and add the group `cms-exo-mci` with Max access level Reporter, 
7. Submit a merge request with your updates to this repository

## Additional resources / information

The exotica combine contact will check datacards submitted to this repository as part of the preapproval process:
- https://twiki.cern.ch/twiki/bin/view/CMS/ExoPreapprovalChecklist


The communication between the analysts and the combine expert should proceed via the analysis Hypernews.

Some good references from other PAGs for simple checks your datacards should pass are: 
- https://twiki.cern.ch/twiki/bin/view/CMS/HiggsWG/HiggsPAGPreapprovalChecks
- https://twiki.cern.ch/twiki/bin/viewauth/CMS/SUSPAGPreapprovalChecks


There are also some nice scripts put together by Carlos Erice:
- https://github.com/cericeci/combineScripts

