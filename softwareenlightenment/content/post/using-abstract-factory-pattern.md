+++

author = "Jacob Hell"
title = "Have a lot of if/else Statements Doing Similar Things? Use an Abstract Factory!"
date = "2020-03-19"
description = "Using an Abstract Factory Pattern."
tags = [
    "java",
    "abstract factory"
]

+++

When working on a legacy system, one of the more common problems I see are a lot of similar action happening within if/else or control statements.

<!--more-->

Take this code snippet for example:

```java
public LedgerEntry getLedgerEntry(TransactionDetails details)
{
	LedgerEntry entry = null;
	String transactionType = details.getTransactionType();
	if(transactionType.equals("check")) {
		entry = new CheckEntry();
		entry.setCheckType("check");
		entry.setCheckPurchaser(details.getPurchaser());
		entry.setCheckDate(details.getDate());
		entry.setRoutingNumber(details.getRoutingNumber());
		entry.setAccountNumber(details.getAccountNumber());
	} else if(transactionType.equals("creditcard")) {
		entry = new CreditCardLedgerEntry();
		entry.setType("creditcard");
		entry.setCreditCardPurchaser(details.getPurchaser());
		entry.setCreditCardDate(details.getDate());
		entry.setCreditCardNumber(details.getCreditCardNumber());
	} else if(transactionType.equals("cash")) {
		entry = new CashEntry();
		entry.setType("cash");
		entry.setCashPurchaser(details.getPurchaser());
		entry.setCashDate(details.getDate());
	} else if(transactionType.equals("crypto")) {
		entry = new CryptoEntry();
		entry.setType("crypto");
		entry.setCryptoPurchaser(details.getPurchaser());
		entry.setCryptoDate(details.getDate());
		entry.setCyrptoPublicKey(details.getPublicKey());
	}
	

	return entry;

}
```

Let's say that this snippet is from a larger accounting program that will save financial records to a database. 

This method is doing the following:
1. Determining what type of transaction occurred based on the transaction type
2. Creating a concrete instance of the `LedgerEntry` abstract class
3. Returning this entry

This method probably works fine, but what if we needed to do something for both cash and cryptocurrency transactions:

```java
public LedgerEntry getLedgerEntry(TransactionDetails details)
{
	// previous code
	

	if(transactionType.equals("cash") || transactionType.equals("crypto"))
	{
		entry.setAlertFBIAndPresidentOfUSA(true);
	}
	return entry;

}
```



And what if further we needed to do something special to other transaction types? This could get complicated fast.

My go to solution for problems such as this to use the **Abstract Factory Pattern**. This is one of the patterns found in the famous book: [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Object-Oriented-Addison-Wesley-Professional-ebook/dp/B000SEIBB8).

The Abstract Factory Pattern relies on the client creating a **Factory** and then calling the abstract method **create**.

The nice thing about this is the client doesn't have to know about the inner workings of each create method. 

Let's see what this method looks like after using the Abstract Factory Pattern:

```java
public LedgerEntry getLedgerEntry(TransactionDetails details)
{
	AbstractLedgerFactory ledgerFactory = null;
	String transactionType = details.getTransactionType();
	if(transactionType.equals("check")) {
		ledgerFactory =  new CheckLedgerFactory();
	} else if(transactionType.equals("creditcard")) {
		ledgerFactory = new CreditCardLedgerFactory();
	} else if(transactionType.equals("cash")) {
		ledgerFactory = new CashLedgerFactory();
	} else if(transactionType.equals("crypto")) {
		ledgerFactory = new CryptoLedgerFactory();
	}
	

	return ledgerFactory.create(details);

}
```

Much easier to read! And if more logic needs to happen for transaction types, it can be done in the concrete factory method.

Here is the `AbstractLedgerFactory` and `CryptoLedgerFactory`:

```java
public abstract class AbstractLedgerFactory
{
	public abstract LedgerEntry create(TransactionDetails details);
}
```

```java
public class CryptoLedgerFactory extends AbstractLedgerFactory
{
	@Override
	public LedgerEntry create(TransactionDetails details)
	{
		LedgerEntry entry = new CryptoEntry();
		entry.setType("crypto");
		entry.setCryptoPurchaser(details.getPurchaser());
		entry.setCryptoDate(details.getDate());
		entry.setCyrptoPublicKey(details.getPublicKey());
		entry.setAlertFBIAndPresidentOfUSA(true);
		

		return entry;
	}

}
```

