# Cycles.Finance

### Disclaimers.

**This project is in beta and may have defects.  It‘s a Dapp, please be knowledgeable and participate voluntarily and at your own risk.**

##  Overview

Cycles.Finance is an ICP/Cycles marketplace that supports bidirectional exchange of ICP, Cycles, using a multiplicative constant K model (A*B=K), similar to UniSwapV2.

#### Restrictions on Swapping

The project is still in beta and limits have been placed on a swap in order to control risk. 
- ICP: max 10 icp each swap, min 10,000 e8s each swap.
- Cycles: maximum 3*10^14 cycles each swap, minimum 10^8 cycles each swap.
- Swap volatility limit: each swap causing price fluctuations more than 20% will be rejected.
 
#### Swap fees

Swapping fee: 1%, on a post-fee model, charged for ICPs or Cycles.  
Swapping fee usage: All ICP charged is moved to the liquidity reward pool, 80% of Cycles charged is moved to the liquidity reward pool (the other 20% is used for Canister gas).

#### Liquidity (AMM)

Market making model: automatic market making model(AMM). Using a multiplicative constant K model (AB=K).     
Liquidity reward pool:  
- Liquidity providers receive liquidity rewards in proportion to the time-weighted share they hold.  
- [Plan] Participate in the ICLighthouse liquidity mining program and receive ICL token rewards.

## Usage

**Canister Id：ium3d-eqaaa-aaaak-aab4q-cai**  
**Module hash: 73e1a8f888c91abe086fedc24fe660d9cf068c2d78f6b172dd58f92cd5b503f6**  

**Notes**
- The basic unit of ICP in canister is e8s, 1 icp = 10^8 e8s;
- The ICP/Cycles rate on IC network changes dynamically and is pegged to the XDR value, 1 XDR = 10^12 cycles (value approx. 1.4 USD).
- - The ICP/Cycles rate on this canister is automatically formed by the market and may deviate from other markets.
- Your `ICP account principal` and `Cycles wallet account principal` are used to interact with this canister, please note the difference between them.

### Query ICP/Cycles price
````
dfx canister --network ic call ium3d-eqaaa-aaaak-aab4q-cai liquidity '(null)'
````
The `e8s` (or `5_035_232`) field in the return value is divided by the `cycles` (or `2_190_693_645`) field to indicate how many cycles can be exchanged for 1 e8s. This value is multiplied by 10^8 to indicate how many cycles can be exchanged for 1 icp. this is an estimate.
````
(
  record {
    icp = record { e8s = 787_146_478 : nat64 };
    vol = record {
      swapIcpVol = 1_740_878 : nat;
      swapCyclesVol = 573_069_740_022 : nat;
    };
    shareWeighted = record {
      updateTime = 1_638_592_854 : nat;
      shareTimeWeighted = 3_894_326_391_123 : nat;
    };
    unitValue = record { 329155.999121 : float64; 0.972376 : float64 };
    share = 809_508_285 : nat;
    cycles = 266_454_525_225_963 : nat;
    priceWeighted = record {
      updateTime = 1_638_592_854 : nat;
      icpTimeWeighted = 3_800_565_037_457 : nat;
      cyclesTimeWeighted = 1_277_301_377_584_917_917 : nat;
    };
    swapCount = 0 : nat64;
  },
)
````

### ICP to Cycles

Step1: Get your dedicated ICP deposit account-id (**DepositAccountId**)
````
dfx canister --network ic call ium3d-eqaaa-aaaak-aab4q-cai getAccountId '(principal "<your_icp_account_principal>")'
````
Return `DepositAccountId`(example)
````
("f2d1945ebc293bdc2cc6ef**************e84cf61f51ce6798fc4283") 
````

Step2: Send ICP to `DepositAccountId`
````
dfx ledger --network ic transfer <your_DepositAccountId> --memo 0 --e8s <icp_e8s_amount>
````

Step3: Converting to Cycles. Parameters `icp_e8s_amount` is the amount sent in Step2 and `your_cycles_wallet_principal` is the principal of your cycles wallet (note: not your ICP account principal).
````
dfx canister --network ic call ium3d-eqaaa-aaaak-aab4q-cai icpToCycles '(<icp_e8s_amount>:nat,principal "<your_cycles_wallet_principal>",null)'
````
Check your wallet balance
````
dfx wallet --network ic balance
````

### Cycles to ICP

Step1: Use the didc tool to encode the parameters. Note, didc tool resources: https://github.com/dfinity/candid/tree/master/tools/didc
````
didc encode '(principal "<your_icp_account_principal>",null)' -t '(principal,opt blob)' -f blob
````
Return `CallArgs`(example)
````
blob "DIDL\02n\01m{\02h\00\01\**************\88\01\e1\18\fd6G\02\00"
````

Step2: Converting to ICP. The parameter `cycles_amount` is the amount of cycles you want to convert, and the parameter `call_args` is the `CallArgs` got from Step1. 
````
dfx canister --network ic call <your_cycles_wallet_principal> wallet_call '(record {canister=principal "ium3d-eqaaa-aaaak-aab4q-cai"; method_name="cyclesToIcp"; cycles=<cycles_amount>:nat64; args=<call_args>})'
````
Check your account balance
````
dfx ledger --network ic balance
````

### Add liquidity

To add liquidity, both ICP and Cycles to be added to the liquidity pool, the proportion is calculated based on the current price and the excess will refunded.

Step1: Get your dedicated ICP deposit account-id（**DepositAccountId**）
````
dfx canister --network ic call ium3d-eqaaa-aaaak-aab4q-cai getAccountId '(principal "<your_icp_account_principal>")'
````
Return (example)
````
("f2d1945ebc293bdc2cc6ef**************e84cf61f51ce6798fc4283") 
````

Step2: Send ICP to `DepositAccountId`
````
dfx ledger --network ic transfer <your_DepositAccountId> --memo 0 --e8s <icp_e8s_amount>
````

Step3: Use the didc tool to encode the parameters. Note, didc tool resources: https://github.com/dfinity/candid/tree/master/tools/didc
````
didc encode '(principal "<your_icp_account_principal>",null)' -t '(principal,opt blob)' -f blob
````
Return `CallArgs`(example)
````
blob "DIDL\02n\01m{\02h\00\01\**************\88\01\e1\18\fd6G\02\00"
````

Step4: Send Cycles, add liquidity. The parameter `cycles_amount` is the amount of cycles you want to add, and the parameter `call_args` is the `CallArgs` got from Step3.
````
dfx canister --network ic call <your_cycles_wallet_principal> wallet_call '(record {canister=principal "ium3d-eqaaa-aaaak-aab4q-cai"; method_name="add"; cycles=<cycles_amount>:nat64; args=<call_args>})'
````

Step5: Enquire about liquidity shares
````
dfx canister --network ic call ium3d-eqaaa-aaaak-aab4q-cai liquidity '(opt principal "<your_icp_account_principal>")'
````
Return (example). The `share` (or `2_082_268_383`) field is the share of liquidity you hold.
````
(
  record {
    icp = record { e8s = 48_521_783 : nat64 };
    vol = record {
      swapIcpVol = 1_648_218 : nat;
      swapCyclesVol = 541_650_948_359 : nat;
    };
    shareWeighted = record {
      updateTime = 1_638_528_867 : nat;
      shareTimeWeighted = 695_045_889_662 : nat;
    };
    unitValue = record { 329748.544469 : float64; 0.970629 : float64 };
    share = 49_990_000 : nat;   
    cycles = 16_484_143_085_896 : nat;
    priceWeighted = record {
      updateTime = 1_638_528_867 : nat;
      icpTimeWeighted = 689_683_291_306 : nat;
      cyclesTimeWeighted = 224_229_508_922_468_505 : nat;
    };
    swapCount = 0 : nat64;
  },
)
````

### Remove liquidity

Step1: Query your liquidity share, the `share` (or `2_082_268_383`) field in the return is your share held.

````
dfx canister --network ic call ium3d-eqaaa-aaaak-aab4q-cai liquidity '(opt principal "<your_icp_account_principal>")'
````

Step2: Remove liquidity. The parameter `share_amount` must be equal to or less than the value queried by Step1, and the parameter `your_cycles_wallet_principal` is wallet principal used to receive the cycles, and caller  principal will receive icp.

````
dfx canister --network ic call ium3d-eqaaa-aaaak-aab4q-cai remove '(<share_amount>:nat, principal "<your_cycles_wallet_principal>", null)'
````
Check your account balance
````
dfx ledger --network ic balance
dfx wallet --network ic balance
````

### Claim Rewards

Claim Returns. The parameter `your_cycles_wallet_principal` is wallet principal used to receive cycles, and caller principal will receive icp.
````
dfx canister --network ic call ium3d-eqaaa-aaaak-aab4q-cai claimReturns '(principal "<your_cycles_wallet_principal>", null)'
````
Check your account balance
````
dfx ledger --network ic balance
dfx wallet --network ic balance
````

## DID

````
type Vol = record { swapCyclesVol: nat; swapIcpVol: nat; };
type Txid = blob;
type TokenType = variant { Cycles; DRC20: principal; Icp; };
type Timestamp = nat;
type Time = int;
type SwapResult = record { cycles: BalanceChange; icpE8s: BalanceChange; share: BalanceChange; txid: Txid; };
type SwapRecord = record {
   account: principal;
   cyclesWallet: opt principal;
   data: opt blob;
   fee: record { token0Fee: nat; token1Fee: nat; };
   operation: Operation;
   share: LiquidityShareChange;
   time: Time;
   token0: TokenType;
   token0Value: BalanceChange;
   token1: TokenType;
   token1Value: BalanceChange;
   txid: Txid;
};
type ShareWeighted = record { shareTimeWeighted: nat; updateTime: Timestamp; };
type PriceWeighted = record { cyclesTimeWeighted: nat; icpTimeWeighted: nat; updateTime: Timestamp; };
type Operation = variant { AddLiquidity; ClaimReturns; RemoveLiquidity; Swap; };
type LiquidityShareChange = variant { Burn: nat; Mint: nat; NoChange; };
type Liquidity = record {
   cycles: nat;
   icp: ICP;
   priceWeighted: PriceWeighted;
   share: nat;
   shareWeighted: ShareWeighted;
   swapCount: nat64;
   unitValue: record { float64; float64;};
   vol: Vol;
};
type ICP = record {e8s: nat64;};
type BalanceChange = variant { In: nat; Out: nat;};
type CyclesMarket = service {
   getAccountId: (_account: principal) -> (text) query;
   cyclesToIcp: (_account: principal, _data: opt blob) -> (SwapResult);
   icpToCycles: (_icpE8s: nat, _cyclesWallet: principal, _data: opt blob) -> (SwapResult);
   add: (_account: principal, _data: opt blob) -> (SwapResult);
   remove: (_share: opt nat, _cyclesWallet: principal, _data: opt blob) -> (SwapResult);
   claimReturns: (_cyclesWallet: principal, _data: opt blob) -> (SwapResult);
   liquidity: (_account: opt principal) -> (Liquidity) query;
   lastSwaps: () -> (nat, vec Txid) query;
   swapRecord: (_txid: Txid) -> (opt SwapRecord) query;
};
service : () -> CyclesMarket
````


## Roadmap

(doing) Development of UI interface, open source contract code.

Upgrade to v2.0 with scalable storage of transaction records.

Opening up liquidity mining.


## Community

Web: https://cycles.finance/  

Github: https://github.com/iclighthouse/Cycles.Finance 

Twitter: https://twitter.com/ICLighthouse

Medium: https://medium.com/@ICLighthouse 

Discord: https://discord.gg/FQZFGGq7zv
