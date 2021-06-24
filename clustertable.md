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
