
#   Muon chambers bitmap



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




# Expand the cluster table with a chamber index

This is to know the clusters attached to the tracks on each chamber



>```
>namespace muoncluster
>{
>DECLARE_SOA_INDEX_COLUMN_FULL(Track, track, int, Muons, ""); //! points to a >muon track in the Muon table
>DECLARE_SOA_COLUMN(X, x, float);                             //!
>DECLARE_SOA_COLUMN(Y, y, float);                             //!
>DECLARE_SOA_COLUMN(Z, z, float);                             //!
>DECLARE_SOA_COLUMN(ErrX, errX, float);                       //!
>DECLARE_SOA_COLUMN(ErrY, errY, float);                       //!
>DECLARE_SOA_COLUMN(Charge, charge, float);                   //!
>DECLARE_SOA_COLUMN(Chi2, chi2, float);                       //!
>} // namespace muoncluster
>
>DECLARE_SOA_TABLE(MuonClusters, "AOD", "MUONCLUSTER", //!
>                  muoncluster::TrackId,
>                  muoncluster::X, muoncluster::Y, muoncluster::Z,
>                  muoncluster::ErrX, muoncluster::ErrY,
>                  muoncluster::Charge, muoncluster::Chi2);
>
>```
