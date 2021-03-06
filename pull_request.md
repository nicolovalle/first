
1. Fork the repository: for example, go to `https://github.com/AliceO2Group/AliceO2/fork` and clone it in your github space.
2. Download your repo:
```
git clone https://github.com/nicolovalle/AliceO2
```
3. `cd` into it and set credentials:
```
git config user.name "Nicolo Valle"
git config user.email "nicolo.valle@cern.ch"
git config user.github "nicolovalle"
```
4. Create a branch in your fork:
```
git checkout -b mchbitmap
```
5. Pull: `git push --set-upstream origin mchbitmap`
6. Modify the code
7. Add the changes:
```
git add Framework/Core/include/Framework/AnalysisDataModel.h Analysis/DataModel/include/AnalysisDataModel/ReducedInfoTables.h Analysis/Tasks/PWGDQ/tableMaker*
```
8. Commit:
```
git commit -m "Muon chambers bitmap"
```
9. Push: `git push`
