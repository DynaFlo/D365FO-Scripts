# Reporting SSRS

## Add new custom business documents

``` C#
class XXXPrintMgmtDelegatesHandler
{


    [SubscribesTo(classstr(PrintMgmtDocType), delegatestr(PrintMgmtDocType, getDefaultReportFormatDelegate))]
    public static void getDefaultReportFormatDelegate(PrintMgmtDocumentType _docType, EventHandlerResult _result)
    {
        switch (_docType)
        {
            case PrintMgmtDocumentType::SalesOrderConfirmation:
                           _result.result(ssrsReportStr(XXXSalesConfirm, Report));
                break;
        }
    }

}

```
