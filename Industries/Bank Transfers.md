# Bank Transfers

## The Basics

Let's say a customer sends money from their bank (call it `OneBank`) to another bank (call it `OtherBank`) in the same country. How does the money actually get transferred from  one bank to another?


## The ledger

The first thing to understand is that banks maintain account balances in ledgers. Traditionally, these were large books storing the dates and details of transactions. More recently, these ledgers are stores in databases as part of a larger Core Banking System product e.g. TCS Bancs, Mambu and so on.

For each account, a ledger records Credit (what is owed) and Debit (what is owned).

## Nostro Accounts

In order to transfer money from one bank to another, banks can maintain a Nostro account with each other. This is nothing more than a shared account between two banks.

So in our example, `OneBank` would have an account in `OtherBank` and `OtherBank` would open an account in `OneBank`. 

Then, in order to process a transfer between a customer from `OneBank` to `OtherBank`, `OneBank` would:
- Debit the amount from the customer's account.
- Credit `OtherBank`'s nostro account with an instruction to credit the beneficiary's account.

On receiving the order, `OtherBank` would:
- Debit it's Nostro account in `OneBank`
- Credit the amount to the beneficiary's account.

## Central Bank

- If every bank had a nostro account with every other bank, things would get really messy.
- In practice, all banks within a country, open an account with the central bank.
- So when a customer in `OneBank` wants to send money to an account in `OtherBank`, it works as follows:

- `OneBank` debits the amount from the customer's account.
- `OneBank` Credits the amount to it's nostro account with the central bank. 
- `OneBank` sends an instruction to the central bank stating that the amount needs to be debited from the nostro account in order to be transferred to a beneficiary account in `OtherBank`

- The central bank debits the amount from the `OneBank` nostro account
- The central bank credits the amount to the `OtherBank` nostro account, with an instruction that the amount needs to be debited from the nostro account in order to be transferred to a beneficiary account.

- `OtherBank` now debits the amount from its nostro account with the central bank
- `OtherBank` credits the amount into the beneficiary's account.

The process described above is called **Real Time Gross Settlement**. 

## Bank Instructions

- Banks can send instructions using [Swift Messages](https://www.sepaforcorporates.com/swift-for-corporates/read-swift-message-structure/) or using the [ISO20022 standard](https://www.iso20022.org/sites/default/files/documents/D7/ISO20022_RTPG_pacs00800106_July_2017_v1_1.pdf)

## Transaction References

There are 3 important transfer references

- **Unique End-to-End Transfer Reference**, **End-to-End Identifier**: This is a reference that uniquely identifies the transaction end-to-end. It is assigned by the initiating bank
- **Instruction Identification**: The instruction identification is a point to point reference that can be used between the instructing party and the instructed party to refer to the individual
instruction.
- **Transaction Reference**: A reference that identifies a transaction within an instruction (an instruction can contain many transactions). It is unique only for an agreed period of time.

## Transaction Dates

**Value Date**
- The date based on which assets either become available to the account owner (in the case of a credit entry), or cease to be available to the account owner (in the case of a debit entry). Interest, penalty, and arrears calculations are also based on this date. (Reference)
- For example, if a customer transfers $10 from their `OneBank` account to your `OtherBank` account, the day the amount is credited to your account and can be used by you is the value date.

**Booking Date**
- The date on which the transaction is recorded in the ledger.

## References

- [A gentle introduction to interbank payment systems](https://bitsonblocks.net/2017/09/04/a-gentle-introduction-to-interbank-payment-systems/
)


