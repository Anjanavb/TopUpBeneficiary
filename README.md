ABout Project
------------
.net frame work : .net core 8 with entitiframe work
Approach : Code first 

2 solution
	1)RechargeBeneficiary: Topup service for the customer for the beneficiaries under him 
	2)AccountBalanceService:To get the bank acount balance

Database : used inbuild database with sample data for testing below scenarios.

Incase database not available, run add-migration and update-database comments.

External service end point can be configured in appsettings.json for  RechargeBeneficiary project


Test
-------

1
	Cutomer 1 --> ID:1 ->Verified--> 1 Beneficiary --> Balance 5000-->Can test adding beneficiary with nick name not more than 20 charectors
	
	Validation message :
	https://localhost:7220/api/Beneficiary/CreateBeneficiary
	{
	  "beneficiaryName": "BeneficiaryA2abcdefghijklmnopqr",
	  "phoneNumber": "971000000002",
	  "customerID": 1
	}

	Add success:
	https://localhost:7220/api/Beneficiary/CreateBeneficiary
	{
	  "beneficiaryName": "BeneficiaryA2",
	  "phoneNumber": "971000000002",
	  "customerID": 1
	}

2.
	CUstomer 2-->ID :2 -->Verified --> 5 Beneficiary --> Cannot Add more beneficiary
	
	validation message : The customer has reached the maximum number of active beneficiaries .
	https://localhost:7220/api/Beneficiary/CreateBeneficiary
	{
	  "beneficiaryName": "BeneficiaryB6",
	  "phoneNumber": "97200000006"
	  "customerID": 2
	}

3
	Cutomer 3 --> ID:3 -->Verified--> 1 beneficiary --> Bank balance 5--> Cannot topup
	Validation message : Insufficient balance for this topup
	https://localhost:7220/api/TopupTransactions/TopupBeneficiary
		{
	  "topupOptionID": 5,
	  "customerID": 3,
	  "beneficiaryID": 7
	  }


4
	Cutomer 4--> ID: 4--> Verified--> 1 Beneficiary-->Current balance 500-->Has 4 transaction of total amount 404 for Beneficiary ID 8, hence cannot perform topup of 100(ID 7)
	validation message : Recharge limit exceeded for this beneficiary this month.
	
	https://localhost:7220/api/TopupTransactions/TopupBeneficiary

	{
	  "topupOptionID": 7,
	  "customerID": 4,
	  "beneficiaryID": 8
	}
	
	But he will be able to topup small amount (topupOptionID 3 = AED 20)
	Success
	https://localhost:7220/api/TopupTransactions/TopupBeneficiary

	{
	  "topupOptionID": 3,
	  "customerID": 4,
	  "beneficiaryID": 8
	}
	
	Also check if the balance is 500-(20+1)=479
	https://localhost:7206/api/AccountBalance/GetBalance?accountId=4
	
5
	
	Cutomer 5--> ID: 5-->Not Verified--> 1 Beneficiary-->current balance 1000-->Has 10 transaction of total amount960 
	validation message : Recharge limit exceeded for this beneficiary this month.
	
	https://localhost:7220/api/TopupTransactions/TopupBeneficiary

	{
	  "topupOptionID": 7,
	  "customerID": 5,
	  "beneficiaryID": 10
	}
	
6 
	Cutomer 6--> ID: 6-->Not Verified--> 5 active Beneficiary-->current balance 1000--> Total transactions
	( 
		Beneficry 11 --> 9 *101=909
		Beneficry 12 --> 9 *101=909
		Beneficry 13 --> 9 *101=909
		Beneficry 14 --> 2 *101=202 + 1* 51=51
					Total = 2980
		3000-2980=20
		
		not allowed to do topup 20 or above
		Validation message : Oops! Your transaction amount is higher than the allowed limit of 3000 AED. Please enter a lower amount below {balanceamount} and try again
				https://localhost:7220/api/TopupTransactions/TopupBeneficiary
				{
				  "topupOptionID": 3,
				  "customerID": 6,
				  "beneficiaryID": 14
				}
			
			
		possible topup are either a) or b)
		 a) is allowed to do only 3 transaction  for Benefciary 14/15 : Top up 5 ( 3*(5+1)=18<20)
		 
			*	https://localhost:7220/api/TopupTransactions/TopupBeneficiary
				{
				  "topupOptionID": 1,
				  "customerID": 6,
				  "beneficiaryID": 15
				}
				https://localhost:7206/api/AccountBalance/GetBalance?accountId=6
				available balance : 994
				
			*	https://localhost:7220/api/TopupTransactions/TopupBeneficiary
				{
				  "topupOptionID": 1,
				  "customerID": 6,
				  "beneficiaryID": 15
				}
				https://localhost:7206/api/AccountBalance/GetBalance?accountId=6
				available balance: 988
				
			*	https://localhost:7220/api/TopupTransactions/TopupBeneficiary
				{
				  "topupOptionID": 1,
				  "customerID": 6,
				  "beneficiaryID": 15
				}
				https://localhost:7206/api/AccountBalance/GetBalance?accountId=6
				available balance: 982
				
			if you try 4th topup
			Validation message :Oops! Your transaction amount is higher than the allowed limit of 3000 AED for this month.Please try next month
			*	https://localhost:7220/api/TopupTransactions/TopupBeneficiary
				{
				  "topupOptionID": 1,
				  "customerID": 6,
				  "beneficiaryID": 14
				}
			
		 b) is allowed to do only 2 transaction for beneficiary 14/15 : topup 10 + topup 5 ( (10+1)+(5+1)=17<20)
				
			*	https://localhost:7220/api/TopupTransactions/TopupBeneficiary
				{
				  "topupOptionID": 2,
				  "customerID": 6,
				  "beneficiaryID": 14
				}
				https://localhost:7206/api/AccountBalance/GetBalance?accountId=6
				available balance: 989
				
			*	https://localhost:7220/api/TopupTransactions/TopupBeneficiary
				{
				  "topupOptionID": 1,
				  "customerID": 6,
				  "beneficiaryID": 14
				}
				https://localhost:7206/api/AccountBalance/GetBalance?accountId=6
				available balance: 983
		
		
		
	
