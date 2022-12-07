# Création d'une clé comptable LedgerDimension avec un compte et des dimensions

```
MainAccount             mainAccount = MainAccount::findByMainAccountId('6037000000');
LedgerDimensionAccount  ledgerDimensionAccount;

DimensionDefault                    defaultDimension;
DimensionAttributeValueSetStorage   dimAttrValueSetStorage;
DimensionAttributeValue             davACentreProfit;
DimensionAttributeValue             davBSectionAnalytique;
DimensionAttribute                  daACentreProfit = DimensionAttribute::findByName('A_CENTRES_DE_PROFITS_COUTS');
DimensionAttribute                  daBSectionAnalytique = DimensionAttribute::findByName('B_SECTIONS_ANALYTIQUES');

davACentreProfit = DimensionAttributeValue::findByDimensionAttributeAndValue(daACentreProfit, '002');
davBSectionAnalytique = DimensionAttributeValue::findByDimensionAttributeAndValue(daBSectionAnalytique, 'ZZZZZ');

dimAttrValueSetStorage = new DimensionAttributeValueSetStorage();
dimAttrValueSetStorage.addItem(davACentreProfit);
dimAttrValueSetStorage.addItem(davBSectionAnalytique);
defaultDimension = dimAttrValueSetStorage.save();

ledgerDimensionAccount = LedgerDefaultAccountHelper::getDefaultAccountFromMainAccountRecId(mainAccount.RecId);

ledgerDimensionAccount = LedgerDimensionFacade::serviceCreateLedgerDimension(ledgerDimensionAccount, defaultDimension);
```
