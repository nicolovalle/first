
#   Muon chambers bitmap

[Link AliPhysics pull request](https://github.com/alisw/AliPhysics/pull/17940)

[Link O2 pull request](https://github.com/AliceO2Group/AliceO2/pull/6440)

:v: **Use the AliPhysics converter (`AliPhysics/RUN3/AliAnalysisTaskAO2Dconverter.*`) to save a muon chamber map into the AO2Ds** 

 From `AliAODTrack`:


>```
> Bool_t AliAODTrack::HitsMuonChamber(Int_t MuonChamber, Int_t cathode) const {...;}
>
>UInt_t GetMUONClusterMap() const { return (fITSMuonClusterMap& 0x3ff0000)>>16; }
>```

 From `AliESDMuonTrack`:

>```
>UInt_t   GetMuonClusterMap() const {return fMuonClusterMap;}
>```


Adding a Branch to the FwdTrack Tree:

>```
>// Associate FwdTrack branches for MUON tracks
>  TTree *tFwdTrack = CreateTree(kFwdTrack);
>  if (fTreeStatus[kFwdTrack])
>  {
>    tFwdTrack->Branch("fIndexCollisions", &fwdtracks.fIndexCollisions, "fIndexCollisions/I");
>    tFwdTrack->Branch("fIndexBCs", &fwdtracks.fIndexBCs, "fIndexBCs/I");
>    tFwdTrack->Branch("fTrackType", &fwdtracks.fTrackType, "fTrackType/I");
>    tFwdTrack->Branch("fX", &fwdtracks.fX, "fX/F");
>    tFwdTrack->Branch("fY", &fwdtracks.fY, "fY/F");
>    tFwdTrack->Branch("fZ", &fwdtracks.fZ, "fZ/F");
>    tFwdTrack->Branch("fPhi", &fwdtracks.fPhi, "fPhi/F");
>    tFwdTrack->Branch("fTgl", &fwdtracks.fTgl, "fTgl/F");
>    tFwdTrack->Branch("fSigned1Pt", &fwdtracks.fSigned1Pt, "fSigned1Pt/F");
>    tFwdTrack->Branch("fNClusters", &fwdtracks.fNClusters, "fNClusters/I");
>    tFwdTrack->Branch("fPDca", &fwdtracks.fPDca, "fPDca/F");
>    tFwdTrack->Branch("fRAtAbsorberEnd", &fwdtracks.fRAtAbsorberEnd, "fRAtAbsorberEnd/F");
>    tFwdTrack->Branch("fChi2", &fwdtracks.fChi2, "fChi2/F");
>    tFwdTrack->Branch("fChi2MatchMCHMID", &fwdtracks.fChi2MatchMCHMID, "fChi2MatchMCHMID/F");
>    tFwdTrack->Branch("fChi2MatchMCHMFT", &fwdtracks.fChi2MatchMCHMFT, "fChi2MatchMCHMFT/F");
>    tFwdTrack->Branch("fMatchScoreMCHMFT", &fwdtracks.fMatchScoreMCHMFT, "fMatchScoreMCHMFT/F");
>    tFwdTrack->Branch("fMatchMFTTrackID", &fwdtracks.fMatchMFTTrackID, "fMatchMFTTrackID/I");
>    tFwdTrack->Branch("fMatchMCHTrackID", &fwdtracks.fMatchMCHTrackID, "fMatchMCHTrackID/I");
>    tFwdTrack->Branch("fMCHBitMap", &fwdtracks.fMCHBitMap, "fMCHBitMap/s"); // <--------
>    tFwdTrack->SetBasketSize("*", fBasketSizeEvents);
>  }
>```
>
>`fMCHBitMap` is defined as:
>```
>  Int_t fMatchMFTTrackID = -1;
>  Int_t fMatchMCHTrackID = -1;
>  UShort_t fMCHBitMap = 0u; // <--------
>```
>And computed in `AliAnalysisTaskAO2Dconverter::MUONtoFwdTrack(AliESDMuonTrack &MUONTrack)`:
>```
>   convertedTrack.fPDca = AliMathBase::TruncateFloatFraction(pdca, mMuonTrCov);
>    convertedTrack.fMCHBitMap = MUONTrack.GetMuonClusterMap(); // <------------
>```
>
:x: Is there a way to check if `GetMUONClusterMap()` returns exactly the esame number? 

>`AliAnalysisTaskAO2Dconverter::MUONtoFwdTrack(AliESDMuonTrack &MUONTrack) {`

>`AliAnalysisTaskAO2Dconverter::MUONtoFwdTrack(AliAODTrack &MUONTrack) {`



---

**Add column and expand `StoredFwdTracks` table in `O2/Framework/Core/include/Framework/AnalysisDataModel.h`**

:v: Under the namespace `fwdtrack`: (it will be a static column, read from the AO2D)
>```
>DECLARE_SOA_COLUMN(MatchMCHTrackID, matchMCHTrackID, int);     //! ID of matching MCH track for GlobalMuonTracks  (ints while self indexing not available)
>DECLARE_SOA_COLUMN(MCHBitMap, MchBitMap, short); <-------------
>```


:v: Under the namespace aod:

>`DECLARE_SOA_TABLE_FULL(StoredFwdTracks,...`
>
>`..., fwdtrack::MCHBitMap);`

:v: **Add column in `ReducedMuonExtra` table in `O2/Analysis/DataModel/include/AnalysisDataModel/ReducedInfoTables.h`**

>```
>DECLARE_SOA_TABLE(ReducedMuonsExtra, "AOD", "RTMUONEXTRA", //!
>... ,
>fwdtrack::MCHBitMap); <----
>```

Probably this has to be done together with the modification in the tableMaker? It seemed so.

:v: **Use tableMaker to produce skimmed tables defined before**

Both in `O2/Analysis/Tasks/PWGDQ/tableMaker.cxx` and `.../tableMakerMuon_pp.cxx`,

>```
> muonExtended(muon.nClusters(), muon.pDca(), muon.rAtAbsorberEnd(),
>                   muon.chi2(), muon.chi2MatchMCHMID(), muon.chi2MatchMCHMFT(),
>                  muon.matchScoreMCHMFT(), muon.matchMFTTrackID(), muon.matchMCHTrackID(), muon.MchBitMap()); //nicolo
>```

# Save the information in the VarManager and HistogramManager

:v: **Add variable in VarManager** (:x: still not pull requested... to do?)

In `O2/Analysis/PWGDQ/include/PWGDQCore/VarManager.h`:
>```
> // Muon track variables
>    kMuonNClusters,
>    kMuonBitMap, //nicolo  <-----
>    kMuonPDca,
>```

>```
>// Quantities based on the muon extra table
> if constexpr ((fillMap & ReducedMuonExtra) > 0 || (fillMap & Muon) > 0) {
>    values[kMuonNClusters] = track.nClusters();
>    values[kMuonBitMap] = track.MchBitMap(); //nicolo <-------------
>```
In `O2/Analysis/PWGDQ/src/VarManager.cxx`:
>```
>  fgVariableNames[kMuonNClusters] = "muon n-clusters";
>  fgVariableUnits[kMuonNClusters] = "";
>  fgVariableNames[kMuonBitMap] = "muon tracking chambers bitmap"; //nicolo <-----------
>  fgVariableUnits[kMuonBitMap] = ""; //nicolo <-----------
>```

:v: **Add histogram**

In  `O2/Analysis/PWGDQ/include/PWGDQCore/HistogramsLibrary.h`:
>```
>  if (subGroupStr.Contains("muon")) {
>      hm->AddHistogram(histClass, "MuonNClusters", "", false, 100, 0.0, 10.0, VarManager::kMuonNClusters);
>     hm->AddHistogram(histClass, "MuonChambersBitMap", "", false, 1025, 0.0, 1025.0, VarManager::kMuonBitMap); //nicolo <----------
>```



