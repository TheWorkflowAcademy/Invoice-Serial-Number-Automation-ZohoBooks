## Core Idea
Some companies find it meaningful to have Serial Numbers of items reflected in the line items of invoices for traceability. Currently in Zoho Books, Serial Numbers are stored in **Packages** and **Purchase Order Receipts**, while **Invoices** only reflect the line items. When an **Invoice** is created, this custom function queries the package(s) associated to the **Invoice**'s Sales Order, gets the Serial Number(s) of the item(s) and updates the line item description for each Serialized Item with the Serial Number(s) as string values. 

## Tutorial

### Get Associated Package(s)
**Packages** are linked to **Sales Orders** that links to **Invoices**.
```
//Get the Invoice Record
invoiceId = invoice.get("invoice_id");
organizationId = organization.get("organization_id");
invoiceRecord = zoho.inventory.getRecordsByID("Invoices",organizationId,invoiceId);
//Get the Invoice Line Items for later
invoiceItems = invoiceRecord.get("invoice").get("line_items");
//Get the Sales Order Record
salesorderId = invoiceRecord.get("invoice").get("salesorders").get("0").get("salesorder_id");
salesorderRecord = zoho.inventory.getRecordsByID("SalesOrders",organizationId,salesorderId);
//Get all related Packages
packages = salesorderRecord.get("salesorder").get("packages");
//Get a list of all Package Records
packageRecords = List();
for each  p in packages
{
	packageId = p.get("package_id");
	packageRecords.add(zoho.inventory.getRecordsByID("Packages",organizationId,packageId));
}
```

### Get the Serial Number(s) of Each Line Item and Update
Multiple `For Loops` are used to iterate through every **package**, line item(s) in the package and Serial Number(s) within each line item. Another loop is used with an `If` condition to iterate through every line item in **Invoices**, to get the Serial Number(s) of each *item_id* and *line_item_id* in which the item is stored in the **Invoice**. 

Once the iteration is complete, variable *updatelineitems* which sits outside the loops, holds the list of Serial Numbers and its respective *line_item_id* in a map format that is ready for update. Once the map is updated into the **Invoice**, it updates the line item description for each Serialized Item with the Serial Number(s) as string values. 
```
//Get the item IDs each of the Packages, match them with each of the item IDs in Invoice, then create a map to update the serial numbers in the respective descriptions of each Item in the Invoice
updatelineitems = List();
for each  PR in packageRecords
{
	PackageItems = PR.get("package").get("line_items");
	for each  i in PackageItems
	{
		itemId = i.get("item_id");
		serialNum = i.get("serial_numbers");
		for each  R in invoiceItems
		{
			if(R.get("item_id") = itemId)
			{
				description = serialNum.toString();
				lineItem = Map();
				lineItem.put("description",description);
				lineItem.put("line_item_id",R.get("line_item_id"));
				updatelineitems.add(lineItem);
			}
		}
	}
}
updatemap = Map();
updatemap.put("line_items",updatelineitems);
update = zoho.books.updateRecord("Invoices",organizationId,invoiceId,updatemap);
```
