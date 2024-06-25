# NFT 租赁示例

NFT 租赁是一种允许没有所有权或拥有特定 NFT 的个人临时使用或体验它的机制。此流程的实施利用 Kiosk Apps 标准来建立租赁交易的基础设施。这种方法与以太坊 ERC-4907 租赁标准紧密一致，使其成为打算在 Sui 上实施的基于 Solidity 的用例的合适选择。

NFT租赁示例满足以下项目要求：

- 使贷方能够在指定的时间内提供其资产出租（出租列表）。
- 使贷方能够定义租赁期限。
    - 借款人必须遵守租赁期限。
- 借款人可以获得对 NFT 的可变或不可变访问。
    - 不可变访问是只读的。
    - 可变的，贷款人应考虑降级和升级操作并将其计入租赁费用中。
- 租赁期结束后，物品即可正常出售。
- 通过包含转让政策规则来尊重创作者定义的版税。

###  使用案例​

现实世界中 NFT 租赁示例的一些用例包括：

- 游戏
- 票务
- 虚拟土地
- 临时资产与订阅

#### 游戏

在游戏中，有多种情况下租用 NFT 可以有利于用户体验：

- 游戏内资产：NFT 可以代表独特的游戏内物品、角色、皮肤或配件。玩家可以安全地租用这些资产。
- 所有权和真实性：NFT 提供透明且不可篡改的所有权记录，确保真正拥有游戏内物品的玩家可以租用它们，并在租赁期满后收回租用的物品。这可以解决欺诈和伪造等问题。
- 跨游戏集成：租赁 NFT 可以跨多个游戏运行，允许玩家将其独特的物品或角色从一个游戏携带和租赁到另一个游戏，从而促进互操作性。
- 游戏收藏品：NFT 可以代表游戏中的数字收藏品，创建一个数字资产生态系统，玩家可以在其中租用独特的物品。

#### 票务

在票务领域，NFT 在增强可转让性方面发挥着关键作用。这些数字资产促进了门票的安全且可追踪的转让、转售或租赁，从而降低了二级市场中伪造门票的风险。 NFT 基于区块链的性质确保了每笔交易的透明度和真实性，为用户提供可靠且防欺诈的方式来参与票证相关活动。这一创新不仅简化了门票持有者的流程，还有助于建立更值得信赖和更高效的二级门票市场。

#### 虚拟土地​

在元宇宙中租用虚拟土地和办公室为企业提供了灵活的解决方案，使活动公司能够在不承诺永久收购的情况下举办聚会，并通过虚拟办公室促进远程工作。这种方法不仅提供了具有成本效益的替代方案，而且符合数字业务运营不断变化的动态。

#### 临时资产与订阅

临时资产和订阅是租赁 NFT 的著名应用，提供了虚拟体验的可访问性，例如高端虚拟赌场或策划的数字时尚。这些 NFT 迎合了不同的预算，扩大了受众范围。订阅租赁扩展到数字资产池，允许用户每月支付一定数量的物品，从而促进可访问性、用户保留和获取。持有者可以出租未使用的订阅，确保他们不会损失，潜在客户会从协议中获益，并为临时持有者提供免费试用。这展示了租赁 NFT 在不同场景下的适应性和以用户为中心的吸引力。

###  智能合约设计​

租赁智能合约使用 Kiosk Apps 标准。贷方和借方都必须安装 Kiosk 扩展才能参与，并且借用资产类型的创建者必须创建租赁策略和 ProtectedTP 对象，以允许扩展在执行特许权使用费的同时管理租赁。

### Move 模块

NFT 租赁示例使用单个模块 `nft_rental.move` 。您可以在 sui 存储库的 `examples` 目录中找到该文件的源代码。源代码包含大量注释，可帮助您理解示例的逻辑和结构。

#### nft_rental

`nft_rental` 模块提供了一个API，可以通过以下操作促进借贷：

- 出租清单
- 从租赁中除名
- 租赁
- 按参考借用和按价值借用
- 为贷款人赎回

#### 结构

`nft_rental` 模块的对象模型提供了应用程序的结构，从 `Rentables` 对象开始。该结构仅具有 drop 能力，并充当 kiosk `Rentables` 扩展的扩展密钥。

```
public struct Rentables has drop {}
```

`Rented` 结构代表租用的项目。该结构体包含的唯一字段是对象的 ID。当某人主动借用物品时，它用作借用者 `Bag` 条目中的动态字段键。该结构具有 `store` 、 `copy` 和 `drop` 功能，因为它们对于所有动态字段键都是必需的。

```
public struct Rented has store, copy, drop { id: ID }
```

`Listed` 结构代表一个列出的项目。该结构体包含的唯一字段是对象的 ID。在列出出租项目后，它用作承租者 `Bag` 条目中的动态字段键。与 `Rented` 一样，此结构体具有 `store` 、 `copy` 和 `drop` 功能，因为它们对于所有动态字段键都是必需的。

```
public struct Listed has store, copy, drop { id: ID }
```

`Promise` 结构是为了按值借用而创建的。 `Promise` 充当烫手山芋（一个没有任何功能的结构，只能在其模块中打包和解包），只能通过将项目返回到扩展的 `Bag` .

```
public struct Promise {
  item: Rented,
  duration: u64,
  start_date: u64,
  price_per_day: u64,
  renter_kiosk: address,
  borrower_kiosk: ID
}
```

`Rentable` 结构作为一个包装对象，保存正在租用的资产。包含与租赁期限、费用和承租人相关的信息。该结构需要 `store` 能力，因为它存储的值 `T` 肯定也具有 `store` 。

```
public struct Rentable<T: key + store> has store {
  object: T,
  /// Total amount of time offered for renting in days.
  duration: u64,
  /// Initially undefined, is updated once someone rents it.
  start_date: Option<u64>,
  price_per_day: u64,
  /// The kiosk id that the object was taken from.
  kiosk_id: ID,
}
```

`RentalPolicy` 结构是每个创建者创建的共享对象。该结构定义了创建者从每次租金调用中收到的版税。

```
public struct RentalPolicy<phantom T> has key, store {
  id: UID,
  balance: Balance<SUI>,
  /// Note: Move does not support float numbers.
  ///
  /// If you need to represent a float, you need to determine the desired
  /// precision and use a larger integer representation.
  ///
  /// For example, percentages can be represented using basis points:
  /// 10000 basis points represent 100% and 100 basis points represent 1%.
  amount_bp: u64
}
```

`ProtectedTP` 对象是创建者铸造以启用租赁的共享对象。该对象提供对空 `TransferPolicy` 的授权访问。之所以需要这样做，部分原因是 Kiosk 对强制使用费的物品及其可交易性施加了限制。此外，它允许租赁模块在扩展框架内运行，同时保证所处理的资产始终可交易。

需要受保护的空转移策略来促进租赁过程，以便扩展可以转移资产，而无需解决任何其他规则（例如锁定规则、忠诚度规则等）。如果创作者想要对租赁强制收取版税，他们可以使用前面详述的 `RentalPolicy` 。

```
public struct ProtectedTP<phantom T> has key, store {
  id: UID,
  transfer_policy: TransferPolicy<T>,
  policy_cap: TransferPolicyCap<T>
}
```

### 函数签名​

NFT 租赁示例包含以下定义项目逻辑的函数。

`install` 功能允许在信息亭中安装 `Rentables` 扩展。促进租赁流程的一方负责确保用户在其自助服务终端中安装扩展程序。

```
public fun install(
  kiosk: &mut Kiosk,
  cap: &KioskOwnerCap,
  ctx: &mut TxContext
){
  kiosk_extension::add(Rentables {}, kiosk, cap, PERMISSIONS, ctx);
}
```

`remove` 功能使信息亭的所有者（并且只有所有者）能够删除扩展。扩展存储必须为空，事务才能成功。用户不再借用或租用任何物品后，扩展存储空间将清空。 `kiosk_extension::remove` 函数在执行之前执行所有权检查。

```
public fun remove(kiosk: &mut Kiosk, cap: &KioskOwnerCap, _ctx: &mut TxContext){
  kiosk_extension::remove<Rentables>(kiosk, cap);
}
```

`setup_renting` 函数创建并共享 `T` 类型的 `ProtectedTP` 和 `RentalPolicy` 对象。 `T` 类型的发布者是唯一可以执行该操作的实体。

```
public fun setup_renting<T>(publisher: &Publisher, amount_bp: u64, ctx: &mut TxContext) {
  // Creates an empty TP and shares a ProtectedTP<T> object.
  // This can be used to bypass the lock rule under specific conditions.
  // Storing inside the cap the ProtectedTP with no way to access it
  // as we do not want to modify this policy
  let (transfer_policy, policy_cap) = transfer_policy::new<T>(publisher, ctx);

  let protected_tp = ProtectedTP {
    id: object::new(ctx),
    transfer_policy,
    policy_cap,
  };

  let rental_policy = RentalPolicy<T> {
    id: object::new(ctx),
    balance: balance::zero<SUI>(),
    amount_bp,
  };

  transfer::share_object(protected_tp);
  transfer::share_object(rental_policy);
}
```

`list` 函数允许在 `Rentables` 扩展包中列出资产，创建一个包条目，其中资产的 `ID` 作为键， `Rentable` 包装对象作为价值。要求存在只有 `T` 类型的创建者才能创建的 `ProtectedTP` 传输策略。该函数假设物品已放置（并且可以选择锁定）在信息亭中。

```
public fun list<T: key + store>(
  kiosk: &mut Kiosk,
  cap: &KioskOwnerCap,
  protected_tp: &ProtectedTP<T>,
  item_id: ID,
  duration: u64,
  price_per_day: u64,
  ctx: &mut TxContext,
) {
    
  // Aborts if Rentables extension is not installed.
  assert!(kiosk_extension::is_installed<Rentables>(kiosk), EExtensionNotInstalled);

  // Sets the kiosk owner to the transaction sender to keep metadata fields up to date.
  // This is also crucial to ensure the correct person receives the payment.
  // Prevents unexpected results in cases where the kiosk could have been transferred 
  // between users without the owner being updated.
  kiosk.set_owner(cap, ctx);

  // Lists the item for zero SUI.
  kiosk.list<T>(cap, item_id, 0);

  // Constructs a zero coin.
  let coin = coin::zero<SUI>(ctx);
  // Purchases the item with 0 SUI.
  let (object, request) = kiosk.purchase<T>(item_id, coin);

  // Resolves the TransferRequest with the empty TransferPolicy which is protected and accessible only via this module.
  let (_item, _paid, _from) = protected_tp.transfer_policy.confirm_request(request);

  // Wraps the item in the Rentable struct along with relevant metadata.
  let rentable = Rentable {
    object,
    duration,
    start_date: option::none<u64>(),
    price_per_day,
    kiosk_id: object::id(kiosk),
  };

  // Places the rentable as listed in the extension's bag (place_in_bag is a helper method defined in nft_rental.move file).
  place_in_bag<T, Listed>(kiosk, Listed { id: item_id }, rentable);
}
```

`delist` 函数允许租用者删除当前未租用的项目。该功能还将对象放置（或锁定，如果存在锁定规则）回所有者的信息亭。即使您不想申请任何版税，您也应该铸造一个空的 `TransferPolicy` 。如果在某个时候您确实想要强制收取版税，您可以随时更新现有的 `TransferPolicy` 。

```
public fun delist<T: key + store>(
  kiosk: &mut Kiosk,
  cap: &KioskOwnerCap,
  transfer_policy: &TransferPolicy<T>,
  item_id: ID,
  _ctx: &mut TxContext,
) {

  // Aborts if the cap doesn't match the Kiosk.
  assert!(kiosk.has_access(cap), ENotOwner);

  // Removes the rentable item from the extension's Bag (take_from_bag is a helper method defined in nft_rental.move file). 
  let rentable = take_from_bag<T, Listed>(kiosk, Listed { id: item_id });

  // Deconstructs the Rentable object.
  let Rentable {
    object,
    duration: _,
    start_date: _,
    price_per_day: _,
    kiosk_id: _,
  } = rentable;

  // Respects the lock rule, if present, by re-locking the asset in the owner's Kiosk.
  if (has_rule<T, LockRule>(transfer_policy)) {
    kiosk.lock(cap, transfer_policy, object);
  } else {
    kiosk.place(cap, object);
  };
}
```

`rent` 函数可以租用列出的 `Rentable` 。它允许任何人代表其他用户借用物品，前提是他们安装了 `Rentables` 扩展。 `rental_policy` 定义了作为费用保留并添加到租赁政策余额中的代币部分。

```
public fun rent<T: key + store>(
  renter_kiosk: &mut Kiosk,
  borrower_kiosk: &mut Kiosk,
  rental_policy: &mut RentalPolicy<T>,
  item_id: ID,
  mut coin: Coin<SUI>,
  clock: &Clock,
  ctx: &mut TxContext,
) {

  // Aborts if Rentables extension is not installed.
  assert!(kiosk_extension::is_installed<Rentables>(borrower_kiosk), EExtensionNotInstalled);

  let mut rentable = take_from_bag<T, Listed>(renter_kiosk, Listed { id: item_id });

  // Calculates the price of the rental based on the days it was rented for by ensuring the outcome can be stored as a u64.
  let max_price_per_day = MAX_VALUE_U64 / rentable.duration;
  assert!(rentable.price_per_day <= max_price_per_day, ETotalPriceOverflow);
  let total_price = rentable.price_per_day * rentable.duration;

  // Accepts only exact balance for the payment and does not give change.
  let coin_value = coin.value();
  assert!(coin_value == total_price, ENotEnoughCoins);

  // Calculate fees_amount using the given basis points amount (percentage), ensuring the
  // result fits into a 64-bit unsigned integer.
  let mut fees_amount = coin_value as u128;
  fees_amount = fees_amount * (rental_policy.amount_bp as u128);
  fees_amount = fees_amount / (MAX_BASIS_POINTS as u128);

  // Calculate fees_amount using the given basis points amount (percentage), ensuring the result fits into a 64-bit unsigned integer.
  let fees = coin.split(fees_amount as u64, ctx);

  // Merges the fee balance of the given coin with the RentalPolicy balance.
  coin::put(&mut rental_policy.balance, fees);
  // Transfers the payment to the renter.
  transfer::public_transfer(coin, renter_kiosk.owner());
  rentable.start_date.fill(clock.timestamp_ms());

  place_in_bag<T, Rented>(borrower_kiosk, Rented { id: item_id }, rentable);
}
```

`borrow` 功能使借款人能够通过引用从其包中获取 `Rentable` 。

```
public fun borrow<T: key + store>(
  kiosk: &mut Kiosk,
  cap: &KioskOwnerCap,
  item_id: ID,
  _ctx: &mut TxContext,
): &T {
  // Aborts if the cap doesn't match the Kiosk.
  assert!(kiosk.has_access(cap), ENotOwner);
  let ext_storage_mut = kiosk_extension::storage_mut(Rentables {}, kiosk);
  let rentable: &Rentable<T> = &ext_storage_mut[Rented { id: item_id }];
  &rentable.object
}
```

`borrow_val` 功能使借款人能够临时获取 `Rentable` 并同意或承诺归还。 `Promise` 存储有关 `Rentable` 的所有信息，便于在对象返回时重建 `Rentable` 。

```
public fun borrow_val<T: key + store>(
  kiosk: &mut Kiosk,
  cap: &KioskOwnerCap,
  item_id: ID,
  _ctx: &mut TxContext,
): (T, Promise) {
  // Aborts if the cap doesn't match the Kiosk.
  assert!(kiosk.has_access(cap), ENotOwner);
  let borrower_kiosk = object::id(kiosk);

  let rentable = take_from_bag<T, Rented>(kiosk, Rented { id: item_id });

  // Construct a Promise struct containing the Rentable's metadata.
  let promise = Promise {
    item: Rented { id: item_id },
    duration: rentable.duration,
    start_date: *option::borrow(&rentable.start_date),
    price_per_day: rentable.price_per_day,
    renter_kiosk: rentable.kiosk_id,
    borrower_kiosk
  };

  // Deconstructs the rentable and returns the promise along with the wrapped item T.
  let Rentable {
    object,
    duration: _,
    start_date: _,
    price_per_day: _,
    kiosk_id: _,
  } = rentable;

  (object, promise)
}
```

`return_val` 函数使借阅者能够归还借出的物品。

```
public fun return_val<T: key + store>(
  kiosk: &mut Kiosk,
  object: T,
  promise: Promise,
  _ctx: &mut TxContext,
) {
  assert!(kiosk_extension::is_installed<Rentables>(kiosk), EExtensionNotInstalled);

  let Promise {
    item,
    duration,
    start_date,
    price_per_day,
    renter_kiosk,
    borrower_kiosk,
  } = promise;

  let kiosk_id = object::id(kiosk);
  assert!(kiosk_id == borrower_kiosk, EInvalidKiosk);

  let rentable = Rentable {
    object,
    duration,
    start_date: option::some(start_date),
    price_per_day,
    kiosk_id: renter_kiosk,
  };

  place_in_bag(kiosk, item, rentable);
}
```

:::note
`reclaim` 功能是手动调用的，租赁服务提供商负责确保提醒承租人 `reclaim` 。因此，这可能会导致借款人持有资产的时间超过租赁期。这可以通过修改当前合同来缓解，方法是在 `borrow` 和 `borrow_val` 函数中添加断言来检查租赁期是否已过期。
:::

`reclaim` 功能使所有者能够在租赁期结束后收回其资产并将其放入其自助服务终端内。如果存在锁定规则，该示例还会锁定所有者信息亭内的项目。

```
public fun reclaim<T: key + store>(
  renter_kiosk: &mut Kiosk,
  borrower_kiosk: &mut Kiosk,
  transfer_policy: &TransferPolicy<T>,
  clock: &Clock,
  item_id: ID,
  _ctx: &mut TxContext,
) {

  // Aborts if Rentables extension is not installed.
  assert!(kiosk_extension::is_installed<Rentables>(renter_kiosk), EExtensionNotInstalled);

  let rentable = take_from_bag<T, Rented>(borrower_kiosk, Rented { id: item_id });

  // Destructures the Rentable struct to place it back to the renter's Kiosk.
  let Rentable {
    object,
    duration,
    start_date,
    price_per_day: _,
    kiosk_id,
  } = rentable;

  // Aborts if provided kiosk is different that the initial kiosk the item was borrowed from.
  assert!(object::id(renter_kiosk) == kiosk_id, EInvalidKiosk);

  let start_date_ms = *option::borrow(&start_date);
  let current_timestamp = clock.timestamp_ms();
  let final_timestamp = start_date_ms + duration * SECONDS_IN_A_DAY;

  // Aborts if rental duration has not elapsed.
  assert!(current_timestamp > final_timestamp, ERentingPeriodNotOver);

  // Respects the lock rule, if present, by re-locking the asset in the owner's kiosk.
  if (transfer_policy.has_rule<T, LockRule>()) {
    kiosk_extension::lock<Rentables, T>(
      Rentables {},
      renter_kiosk,
      object,
      transfer_policy,
    );
  } else {
    kiosk_extension::place<Rentables, T>(
      Rentables {},
      renter_kiosk,
      object,
      transfer_policy,
    );
  };
}
```

###  序列图​

:::note
此实现假设每个创建者作为启用操作创建一个 `TransferPolicy` （即使为空），以便 `Rentables` 扩展可以运行。这是除了调用 `setup_renting` 方法之外的要求。
:::

####  初始化​

初始化过程是流程的一部分，但每个实体仅发生一次：

- 对于创作者希望允许出租的新类型
    - 涉及使用可选的锁定规则调用 `setup_renting` 和 `TransferPolicy` 创建
- 对于在使用此框架之前从未借过钱的借款人
    - 如果用户不存在自助服务终端，则应创建一个
    - 涉及在其 kiosk 中安装扩展程序
- 对于使用此框架之前从未租赁过的租户
    - 如果用户不存在 kiosk，则应创建一个
    - 涉及在其 kiosk 中安装扩展程序
