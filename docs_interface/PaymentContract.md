<u>[Droplinked Documentations](README.md)</u> >> Payment Contract
# <u>Droplinked Payment Contract</u>

## • <u>Payment Contract</u>
Droplinked's Smart Contract for Handling payments & Transferring tokens. This contract is the main contract for Droplinked's Payment System, and it's the only contract that can transfer tokens from one account to another. This contract would take the fee (1%) from each transfer and send it to the Droplinked's wallet.

## • <u>Payment Contract Entities</u>
This contract contains 2 entities: droplinked account and fee. The droplinked account is the account that the fee would be sent to, and the fee is the amount of fee that would be taken from each transfer(for percission 1000 as fee = 1%, so for example 20% fee means it should be 20000!).

```rust
pub struct DroplinkedStorage {
    droplinked_account : AccountId, 
    fee : u128, // 1000 = 1% fee
}
```

## • <u>Payment Contract Methods</u>

### 1. <u>Direct Pay</u>
This method is used for direct payment from buyer to seller. This method would take the fee from the buyer and send it to the droplinked account, and then transfer the rest of the amount to the seller. This method takes price of product, shipping and tax, and the seller's account as input, also takes the (product_price + shipping + tax) tokens as deposit. It will calculate 
`(product_price * fee / 100000)` as droplinked fee, and `(product_price + shipping + tax - droplinked_fee)` as seller's amount, and then transfer the seller's amount to the seller's account, and the droplinked fee to the droplinked account. Below is the method's implementation in NEAR Protocol : 

```rust
#[payable]
    pub fn direct_pay(&mut self, product_price : String, shipping_price : String, tax_price : String, recipient : AccountId) -> Promise {
        // Get inputs & parse them
        let product_price_u128 = product_price.parse::<u128>().unwrap();
        let shipping_price_u128 = shipping_price.parse::<u128>().unwrap();
        let tax_price_u128 = tax_price.parse::<u128>().unwrap();
        
        // Check if deposit is enough
        if near_sdk::env::attached_deposit() < product_price_u128 + shipping_price_u128 + tax_price_u128{
            near_sdk::env::panic_str("deposit is too low");
        }

        // Calculate fee and do the transfers
        let droplinked_share = product_price_u128 * self.fee / 100000;
        Promise::new(recipient).transfer(((product_price_u128 + shipping_price_u128 + tax_price_u128 )-droplinked_share).into());
        Promise::new(self.droplinked_account.clone()).transfer(droplinked_share.into())
    }
```

### 2. <u>Pay for Recorded Product</u> 
This method is used for paying for a recorded product. This method would take the fee from the buyer and send it to the droplinked account, and then transfer the rest of the amount to publisher and seller based on the commission that publisher takes, and the rest of the amount to the seller. This method takes price of product, shipping and tax, the seller's account, the publisher's account, and the publisher's commission as input, also takes the (product_price + shipping + tax) tokens as deposit. It will calculate 
`(product_price * fee / 100000)` as droplinked fee, and `(product_price + shipping + tax - droplinked_fee)*commission/100000` as publisher's amount, and `(product_price + shipping + tax - droplinked_fee)*(100-commission)/100000` as seller's amount, and then transfer the seller's amount to the seller's account, the publisher's amount to the publisher's account, and the droplinked fee to the droplinked account.

Note : commission is like fee in this method, so for percission 1000 as commission = 1%, so for example 20% commission means it should be 20000. 
Note : The contract would check the caller of this method, and would proceed only if the caller is droplinked's contract!

Below is the method's implementation in NEAR Protocol : 

```rust
#[payable]
    pub fn recorded_pay(&mut self, product_price : String, shipping_price : String, 
        tax_price : String, publisher : AccountId,producer : AccountId, commission : AccountId) -> Promise {
        //TODO
    }
```