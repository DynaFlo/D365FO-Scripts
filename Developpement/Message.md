# Message infolog with link to record

```
str jsonData = MenuItemMessageActionProvider::createMenuItemWithFilterActionData(MenuItemType::Display,
    menuitemDisplayStr(PurchTable),
    MenuItemMessageActionFilterType::CallerRecord,
    _purchTable);

Message::AddAction(MessageSeverity::Informational,
    "@SICModel:OrderCreated",
    _purchTable.PurchId, 
    MessageActionType::DisplayMenuItem, jsonData);
```
