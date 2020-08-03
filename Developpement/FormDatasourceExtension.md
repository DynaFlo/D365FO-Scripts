**Form datasource extension tips**

# Enable field on active method with CoC

``` C#
[ExtensionOf(FormDataSourcestr(ProdJournalTransProd, ProdJournalTable))]
final class ProdJournalTransProd_ProdJournalTable_SICModel_DS_Extension
{
	public int active()
    {
        int ret;
        FormControl			formControlButton;
        ProdJournalTable	currentRecord;

		ret = next active();

        currentRecord = this.cursor();

		formControlButton = this.formRun().design().controlName(formControlStr(ProdJournalTransProd, SICProdLabelPFSFPrint));

        formControlButton.enabled(currentRecord.Posted);

        return ret;
    }

}
```

# Enable field on post click event


``` C#
[FormControlEventHandler(formControlStr(ProdJournalTransProd, PostJournal), FormControlEventType::Clicked)]
public static void PostJournal_OnClicked(FormControl sender, FormControlEventArgs e)
{
    FormControl			formControlButton;
    ProdJournalTable	currentRecord;

    currentRecord = sender.formRun().dataSource().cursor();

    formControlButton = sender.formRun().design().controlName(formControlStr(ProdJournalTransProd, SICProdLabelPFSFPrint));

    formControlButton.enabled(currentRecord.Posted);

}
```