# Tips x++

## TryParse

``` X++
public static AmountCur getAmountFromString(str amountStr)
{
    AmountCur amount = 0;

    xSession xSession = new xSession();

    System.Globalization.CultureInfo    cultureInfo =  System.Globalization.CultureInfo::CreateSpecificCulture(xSession.PreferredLocale());
    
    System.Decimal::TryParse(amountStr, System.Globalization.NumberStyles::Any, cultureInfo, byref amount);
    
    return amount;
}
```
