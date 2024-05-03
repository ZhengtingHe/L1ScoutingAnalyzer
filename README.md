# Example L1Trigger Scouting Analyzer

## Demo Analyzer
Simple example of an `EDAnalyzer` used to analyze L1T Scouting data.
The `DemoAnalyzer` processes BX containing at least a muon and produces a ROOT file containing 4 histograms:
1. The BX occupancy of muons in the orbit
2. Pt distribution of the muons
3. Jet multiplicity for BX where at least a muon is present
4. The BX occupancy such that at least a muon is present and there are no jets in the previous BX (example of how to use objects from a different BX)

## Ntuplizer

Example of an `EDAnalyzer` creating a ROOT tree with L1T scouting data.
Along with the standard `L1Scouting` stream, containing all the BX in a given orbit, the plugin can be used to process data selected by the an online selection, available in the `L1ScoutingSelection` stream.
The output file can be read, for example, with `uproot`:
```
orbit: uint32,
bx: uint32,
nJets: int32,
jetEt: var * float32,
jetEta: var * float32,
jetPhi: var * float32,
nEGammas: int32,
egEt: var * float32,
egEta: var * float32,
egPhi: var * float32,
egIso: var * int32,
...
```

## L1 Scouting data

### 2023
Data collected during 2023 available in DAS, more info in [this](https://indico.cern.ch/event/1381539/contributions/5806977/attachments/2799342/4883215/L1scoutingdataavailability.pdf) slide.
A new data tier `L1SCOUT` will to be used for 2024 data.
DAS query:
```
file dataset=/L1ScoutUGMTCALO/Run2023C-v1/RAW run=368636
```

Event content:
```
$ edmDumpEventContent root://cms-xrd-global.cern.ch//store/data/Run2023C/L1ScoutUGMTCALO/RAW/v1/000/368/636/00000/run368636_ls0400.root
Type                                  Module             Label     Process   
-----------------------------------------------------------------------------
OrbitCollection<l1ScoutingRun3::BxSums>    "CaloUnpacker"     ""        "SCPU"    
OrbitCollection<l1ScoutingRun3::EGamma>    "CaloUnpacker"     ""        "SCPU"    
OrbitCollection<l1ScoutingRun3::Jet>    "CaloUnpacker"     ""        "SCPU"    
OrbitCollection<l1ScoutingRun3::Muon>    "GmtUnpacker"      ""        "SCPU"    
OrbitCollection<l1ScoutingRun3::Tau>    "CaloUnpacker"     ""        "SCPU"
```

Each event contains data from one orbit, i.e. a collection of BX, and are stored in an [OrbitCollection](https://github.com/cms-sw/cmssw/blob/master/DataFormats/L1Scouting/interface/OrbitCollection.h).

L1 Scouting objects are stored as integers hw quantities. The physical values can be obtained using a set utilities functions defined [here](https://github.com/cms-sw/cmssw/blob/master/L1TriggerScouting/Utilities/interface/conversion.h).
For example, hw quantities of scouting muons can be coverted using
```
l1ScoutingRun3::ugmt::fPt(l1ScMuon.hwPt());
```
and, in a similar way for calo objects
```
l1ScoutingRun3::demux::fEt(l1ScJet.hwEt());
```

If all quantities are needed, converting the L1Scouting object to an L1 trigger object may be more convenient (functions available [here](https://github.com/cms-sw/cmssw/blob/master/L1TriggerScouting/Utilities/interface/convertToL1TFormat.h))
```
l1t::Muon l1muon = getL1TMuon(scMuon);
l1t::EGamma l1egamma = getL1TEGamma(scEGamma);
```

Members of `l1t::Jet`, `l1t::EGamma`, `l1t::EtSum`, `l1t::Tau` and `l1t::Muon` objects can be found [here](https://github.com/cms-sw/cmssw/tree/master/DataFormats/L1Trigger/interface).

### 2024

2024 data have the same data format as 2023, but two streams are available: 
* `StreamL1Scouting`: contains all BX in an orbit but prescaled, i.e. only a fraction of the full orbits is kept.
* `StreamL1ScoutingSelection`: constains a selection of BX in an orbit (e.g. BX with at least two jets), but for all the orbits.

## Instructions for the demo

### Demo Analyzer
```
# create a project area
cmsrel CMSSW_14_1_0_pre1 

cd CMSSW_14_1_0_pre1/src
cmsenv

mkdir Demo
cd Demo

git clone https://github.com/Mmiglio/L1ScoutingAnalyzer DemoAnalyzer

cd DemoAnalyzer
scram b

# init proxy
voms-proxy-init --voms cms

# process 10k orbits
cmsRun python/demo_cfg.py inFile=root://cms-xrd-global.cern.ch//store/data/Run2023C/L1ScoutUGMTCALO/RAW/v1/000/368/636/00000/run368636_ls0400.root outFile=test.root numOrbits=10000
```

### Ntuplizer
```
# Follow the steps used to setup the Demo Analyzer example project area

# ntuplize BX selected by the Dijet30Barrel selection
cmsRun python/ntuplizer_cfg.py inFile=file:run380074_ls0200_streamL1ScoutingSelection.root outFile=ntuple_dijet.root numOrbits=10 onlineSelection=Dijet30Barrel

# ntuplize all BX for a prescaled fraction of orbits
cmsRun python/ntuplizer_cfg.py inFile=file:run380074_ls0200_streamL1Scouting.root outFile=ntuple_fullOrbit.root numOrbits=10
```
