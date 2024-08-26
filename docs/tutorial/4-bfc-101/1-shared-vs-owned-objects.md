# 共享对象与拥有对象

Bfc 上的对象可以是共享的（可通过任何事务进行读取和写入访问）或拥有（可通过其所有者签名的事务进行读取和写入访问）。许多应用程序可以使用使用共享对象或仅使用自有对象的解决方案来构建，并且需要权衡每个对象的权衡。

仅使用自有对象的交易受益于极低的最终延迟，因为它们不需要达成共识。另一方面，只有对象的所有者才能访问它，这使得需要处理多方拥有的对象的流程变得复杂，并且对非常热门的对象的访问需要在链外进行协调。

访问一个或多个共享对象的事务需要达成共识才能对这些对象进行顺序读取和写入，从而导致 Gas 成本稍高并增加延迟。

访问多个共享对象或特别流行的对象的事务可能会因争用而导致延迟增加。然而，使用共享对象的优点在于可以灵活地允许多个地址以协调的方式访问同一个对象。

总而言之，对延迟或 Gas 成本极其敏感、不需要处理复杂的多方交易或已经需要链下服务的应用程序可以从仅使用拥有对象的设计中受益。需要多方之间协调的应用程序通常受益于使用共享对象。

有关 Bfc 支持的对象类型的更多信息，请参阅对象所有权。

###  示例：托管​

托管示例通过以两种风格实现相同的应用程序来演示共享对象和拥有对象之间的权衡。它实现了一项服务，允许两个地址彼此执行不信任的对象交换（“交易”），并由该服务托管其对象。

### Locked&lt;T&gt; 和 Key ​

```
module escrow::lock {
    use sui::object::{Self, ID, UID};
    use sui::tx_context::TxContext;

    /// A wrapper that protects access to `obj` by requiring access to a `Key`.
    ///
    /// Used to ensure an object is not modified if it might be involved in a
    /// swap.
    struct Locked<T: store> has key, store {
        id: UID,
        key: ID,
        obj: T,
    }

    /// Key to open a locked object (consuming the `Key`)
    struct Key has key, store { id: UID }

    // === Error codes ===

    /// The key does not match this lock.
    const ELockKeyMismatch: u64 = 0;

    // === Public Functions ===

    /// Lock `obj` and get a key that can be used to unlock it.
    public fun lock<T: store>(
        obj: T,
        ctx: &mut TxContext,
    ): (Locked<T>, Key) {
        let key = Key { id: object::new(ctx) };
        let lock = Locked {
            id: object::new(ctx),
            key: object::id(&key),
            obj,
        };
        (lock, key)
    }

    /// Unlock the object in `locked`, consuming the `key`.  Fails if the wrong
    /// `key` is passed in for the locked object.
    public fun unlock<T: store>(locked: Locked<T>, key: Key): T {
        assert!(locked.key == object::id(&key), ELockKeyMismatch);
        let Key { id } = key;
        object::delete(id);

        let Locked { id, key: _, obj } = locked;
        object::delete(id);
        obj
    }

    // === Tests ===
    #[test_only] use sui::coin::{Self, Coin};
    #[test_only] use sui::bfc::BFC;
    #[test_only] use sui::test_scenario::{Self as ts, Scenario};

    #[test_only]
    fun test_coin(ts: &mut Scenario): Coin<BFC> {
        coin::mint_for_testing<BFC>(42, ts::ctx(ts))
    }

    #[test]
    fun test_lock_unlock() {
        let ts = ts::begin(@0xA);
        let coin = test_coin(&mut ts);

        let (lock, key) = lock(coin, ts::ctx(&mut ts));
        let coin = unlock(lock, key);

        coin::burn_for_testing(coin);
        ts::end(ts);
    }

    #[test]
    #[expected_failure(abort_code = ELockKeyMismatch)]
    fun test_lock_key_mismatch() {
        let ts = ts::begin(@0xA);
        let (l, _k) = lock(42, ts::ctx(&mut ts));
        let (_l, k) = lock(43, ts::ctx(&mut ts));

        unlock(l, k);
        abort 1337
    }
}
```

两种实现都使用原语来锁定值，它提供以下接口：

```
module escrow::lock {
    public fun lock<T: store>(obj: T, ctx: &mut TxContext): (Locked<T>, Key);
    public fun unlock<T: store>(locked: Locked<T>, key: Key): T
}
```

任何 `T: store` 都可以被锁定，得到一个 `Locked<T>` 和对应的 `Key` ，反之，可以使用锁定的值及其对应的 `key` 来取回包裹的对象。

该接口提供的重要属性是锁定的值无法修改，除非先解锁它们（然后重新锁定它们）。由于解锁会消耗密钥，因此可以通过记住锁定密钥的 ID 来检测对锁定值的篡改。这可以防止交换中的一方更改他们提供的对象以降低其价值的情况。

###  拥有的对象​

使用拥有的对象实现的通过托管进行交换的协议从双方锁定各自的对象开始。

这用于证明在同意交换后该对象没有被篡改。如果任何一方不想在这个阶段继续，他们只需解锁他们的对象。

假设双方都乐意继续，下一步需要双方交换密钥。

第三方充当托管人。托管人持有等待其对应对象到达的对象，当它们到达时，它会将它们匹配以完成交换。

```
// examples/trading/contracts/escrow/sources/owned.move

public fun create<T: key + store>(
    key: Key,
    locked: Locked<T>,
    exchange_key: ID,
    recipient: address,
    custodian: address,
    ctx: &mut TxContext,
) {
    let escrow = Escrow {
        id: object::new(ctx),
        sender: ctx.sender(),
        recipient,
        exchange_key,
        escrowed_key: object::id(&key),
        escrowed: locked.unlock(key),
    };

    transfer::transfer(escrow, custodian);
}
```

`create` 函数准备 `Escrow` 请求并将其发送到 `custodian` 。该方提供的对象通过其密钥传入并锁定，并且所请求的对象由其锁定的密钥的 ID 来标识。在准备请求时，所提供的对象被解锁，同时记住其密钥的 ID。

尽管托管人被信任能够保持活跃性（如果它拥有交换的双方，则完成交换，并在需要时返回对象），但所有其他正确性属性都在 Move 中维护：即使托管人拥有正在交换的两个对象，唯一有效的属性他们被允许采取的行动是将它们与正确的对应物相匹配以完成交换，或者归还它们：

```
// examples/trading/contracts/escrow/sources/owned.move

/// Function for custodian (trusted third-party) to perform a swap between
/// two parties.  Fails if their senders and recipients do not match, or if
/// their respective desired objects do not match.
public fun swap<T: key + store, U: key + store>(
    obj1: Escrow<T>,
    obj2: Escrow<U>,
) {
    let Escrow {
        id: id1,
        sender: sender1,
        recipient: recipient1,
        exchange_key: exchange_key1,
        escrowed_key: escrowed_key1,
        escrowed: escrowed1,
    } = obj1;

    let Escrow {
        id: id2,
        sender: sender2,
        recipient: recipient2,
        exchange_key: exchange_key2,
        escrowed_key: escrowed_key2,
        escrowed: escrowed2,
    } = obj2;
    id1.delete();
    id2.delete();

    // Make sure the sender and recipient match each other
    assert!(sender1 == recipient2, EMismatchedSenderRecipient);
    assert!(sender2 == recipient1, EMismatchedSenderRecipient);

    // Make sure the objects match each other and haven't been modified
    // (they remain locked).
    assert!(escrowed_key1 == exchange_key2, EMismatchedExchangeObject);
    assert!(escrowed_key2 == exchange_key1, EMismatchedExchangeObject);

    // Do the actual swap
    transfer::public_transfer(escrowed1, recipient1);
    transfer::public_transfer(escrowed2, recipient2);
}
```

`swap` 函数通过比较各自的密钥 ID 来检查发送者和接收者是否匹配，以及每一方是否想要另一方提供的对象。如果托管人试图将两个不相关的托管请求匹配在一起进行交换，则交易将不会成功。

###  共享对象​

```
// examples/trading/contracts/escrow/sources/shared.move

/// An escrow for atomic swap of objects using shared objects without a trusted
/// third party.
///
/// The protocol consists of three phases:
///
/// 1. One party `lock`s their object, getting a `Locked` object and its `Key`.
///    This party can `unlock` their object to preserve livness if the other
///    party stalls before completing the second stage.
///
/// 2. The other party registers a publicly accessible, shared `Escrow` object.
///    This effectively locks their object at a particular version as well,
///    waiting for the first party to complete the swap.  The second party is
///    able to request their object is returned to them, to preserve liveness as
///    well.
///
/// 3. The first party sends their locked object and its key to the shared
///    `Escrow` object.  This completes the swap, as long as all conditions are
///    met:
///
///    - The sender of the swap transaction is the recipient of the `Escrow`.
///
///    - The key of the desired object (`exchange_key`) in the escrow matches
///      the key supplied in the swap.
///
///    - The key supplied in the swap unlocks the `Locked<U>`.
module escrow::shared {
    use sui::{
        event,
        dynamic_object_field::{Self as dof}
    };

    use escrow::lock::{Self, Locked, Key};

    /// The `name` of the DOF that holds the Escrowed object.
    /// Allows easy discoverability for the escrowed object.
    public struct EscrowedObjectKey has copy, store, drop {}

    /// An object held in escrow
    /// 
    /// The escrowed object is added as a Dynamic Object Field so it can still be looked-up.
    public struct Escrow<phantom T: key + store> has key, store {
        id: UID,

        /// Owner of `escrowed`
        sender: address,

        /// Intended recipient
        recipient: address,

        /// ID of the key that opens the lock on the object sender wants from
        /// recipient.
        exchange_key: ID,
    }

    // === Error codes ===

    /// The `sender` and `recipient` of the two escrowed objects do not match
    const EMismatchedSenderRecipient: u64 = 0;

    /// The `exchange_for` fields of the two escrowed objects do not match
    const EMismatchedExchangeObject: u64 = 1;

    // === Public Functions ===

    public fun create<T: key + store>(
        escrowed: T,
        exchange_key: ID,
        recipient: address,
        ctx: &mut TxContext
    ) {
        let mut escrow = Escrow<T> {
            id: object::new(ctx),
            sender: ctx.sender(),
            recipient,
            exchange_key,
        };

        event::emit(EscrowCreated {
            escrow_id: object::id(&escrow),
            key_id: exchange_key,
            sender: escrow.sender,
            recipient,
            item_id: object::id(&escrowed),
        });

        dof::add(&mut escrow.id, EscrowedObjectKey {}, escrowed);

        transfer::public_share_object(escrow);
    }

    /// The `recipient` of the escrow can exchange `obj` with the escrowed item
    public fun swap<T: key + store, U: key + store>(
        mut escrow: Escrow<T>,
        key: Key,
        locked: Locked<U>,
        ctx: &TxContext,
    ): T {
        let escrowed = dof::remove<EscrowedObjectKey, T>(&mut escrow.id, EscrowedObjectKey {});

        let Escrow {
            id,
            sender,
            recipient,
            exchange_key,
        } = escrow;

        assert!(recipient == ctx.sender(), EMismatchedSenderRecipient);
        assert!(exchange_key == object::id(&key), EMismatchedExchangeObject);

        // Do the actual swap
        transfer::public_transfer(locked.unlock(key), sender);

        event::emit(EscrowSwapped {
            escrow_id: id.to_inner(),
        });

        id.delete();

        escrowed
    }

    /// The `creator` can cancel the escrow and get back the escrowed item
    public fun return_to_sender<T: key + store>(
        mut escrow: Escrow<T>,
        ctx: &TxContext
    ): T {

        event::emit(EscrowCancelled {
            escrow_id: object::id(&escrow)
        });

        let escrowed = dof::remove<EscrowedObjectKey, T>(&mut escrow.id, EscrowedObjectKey {});

        let Escrow {
            id,
            sender,
            recipient: _,
            exchange_key: _,
        } = escrow;

        assert!(sender == ctx.sender(), EMismatchedSenderRecipient);
        id.delete();
        escrowed
    }

    // === Events ===
    public struct EscrowCreated has copy, drop {
        /// the ID of the escrow that was created
        escrow_id: ID,
        /// The ID of the `Key` that unlocks the requested object.
        key_id: ID,
        /// The id of the sender who'll receive `T` upon swap
        sender: address,
        /// The (original) recipient of the escrowed object
        recipient: address,
        /// The ID of the escrowed item
        item_id: ID,
    }

    public struct EscrowSwapped has copy, drop {
        escrow_id: ID
    }

    public struct EscrowCancelled has copy, drop {
        escrow_id: ID
    }

    // === Tests ===
    #[test_only] use sui::coin::{Self, Coin};
    #[test_only] use sui::bfc::BFC;
    #[test_only] use sui::test_scenario::{Self as ts, Scenario};

    #[test_only] const ALICE: address = @0xA;
    #[test_only] const BOB: address = @0xB;
    #[test_only] const DIANE: address = @0xD;

    #[test_only]
    fun test_coin(ts: &mut Scenario): Coin<BFC> {
        coin::mint_for_testing<BFC>(42, ts.ctx())
    }

    #[test]
    fun test_successful_swap() {
        let mut ts = ts::begin(@0x0);

        // Bob locks the object they want to trade.
        let (i2, ik2) = {
            ts.next_tx(BOB);
            let c = test_coin(&mut ts);
            let cid = object::id(&c);
            let (l, k) = lock::lock(c, ts.ctx());
            let kid = object::id(&k);
            transfer::public_transfer(l, BOB);
            transfer::public_transfer(k, BOB);
            (cid, kid)
        };

        // Alice creates a public Escrow holding the object they are willing to
        // share, and the object they want from Bob
        let i1 = {
            ts.next_tx(ALICE);
            let c = test_coin(&mut ts);
            let cid = object::id(&c);
            create(c, ik2, BOB, ts.ctx());
            cid
        };

        // Bob responds by offering their object, and gets Alice's object in
        // return.
        {
            ts.next_tx(BOB);
            let escrow: Escrow<Coin<BFC>> = ts.take_shared();
            let k2: Key = ts.take_from_sender();
            let l2: Locked<Coin<BFC>> = ts.take_from_sender();
            let c = escrow.swap(k2, l2, ts.ctx());

            transfer::public_transfer(c, BOB);
        };

        // Commit effects from the swap
        ts.next_tx(@0x0);

        // Alice gets the object from Bob
        {
            let c: Coin<BFC> = ts.take_from_address_by_id(ALICE, i2);
            ts::return_to_address(ALICE, c);
        };

        // Bob gets the object from Alice
        {
            let c: Coin<BFC> = ts.take_from_address_by_id(BOB, i1);
            ts::return_to_address(BOB, c);
        };

        ts::end(ts);
    }

    #[test]
    #[expected_failure(abort_code = EMismatchedSenderRecipient)]
    fun test_mismatch_sender() {
        let mut ts = ts::begin(@0x0);

        let ik2 = {
            ts.next_tx(DIANE);
            let c = test_coin(&mut ts);
            let (l, k) = lock::lock(c, ts.ctx());
            let kid = object::id(&k);
            transfer::public_transfer(l, DIANE);
            transfer::public_transfer(k, DIANE);
            kid
        };

        // Alice wants to trade with Bob.
        {
            ts.next_tx(ALICE);
            let c = test_coin(&mut ts);
            create(c, ik2, BOB, ts.ctx());
        };

        // But Diane is the one who attempts the swap
        {
            ts.next_tx(DIANE);
            let escrow: Escrow<Coin<BFC>> = ts.take_shared();
            let k2: Key = ts.take_from_sender();
            let l2: Locked<Coin<BFC>> = ts.take_from_sender();
            let c = escrow.swap(k2, l2, ts.ctx());

            transfer::public_transfer(c, DIANE);
        };

        abort 1337
    }

    #[test]
    #[expected_failure(abort_code = EMismatchedExchangeObject)]
    fun test_mismatch_object() {
        let mut ts = ts::begin(@0x0);

        {
            ts.next_tx(BOB);
            let c = test_coin(&mut ts);
            let (l, k) = lock::lock(c, ts.ctx());
            transfer::public_transfer(l, BOB);
            transfer::public_transfer(k, BOB);
        };

        // Alice wants to trade with Bob, but Alice has asked for an object (via
        // its `exchange_key`) that Bob has not put up for the swap.
        {
            ts.next_tx(ALICE);
            let c = test_coin(&mut ts);
            let cid = object::id(&c);
            create(c, cid, BOB, ts.ctx());
        };

        // When Bob tries to complete the swap, it will fail, because they
        // cannot meet Alice's requirements.
        {
            ts.next_tx(BOB);
            let escrow: Escrow<Coin<BFC>> = ts.take_shared();
            let k2: Key = ts.take_from_sender();
            let l2: Locked<Coin<BFC>> = ts.take_from_sender();
            let c = escrow.swap(k2, l2, ts.ctx());

            transfer::public_transfer(c, BOB);
        };

        abort 1337
    }

    #[test]
    #[expected_failure(abort_code = EMismatchedExchangeObject)]
    fun test_object_tamper() {
        let mut ts = ts::begin(@0x0);

        // Bob locks their object.
        let ik2 = {
            ts.next_tx(BOB);
            let c = test_coin(&mut ts);
            let (l, k) = lock::lock(c, ts.ctx());
            let kid = object::id(&k);
            transfer::public_transfer(l, BOB);
            transfer::public_transfer(k, BOB);
            kid
        };

        // Alice sets up the escrow
        {
            ts.next_tx(ALICE);
            let c = test_coin(&mut ts);
            create(c, ik2, BOB, ts.ctx());
        };

        // Bob has a change of heart, so they unlock the object and tamper with
        // it before initiating the swap, but it won't be possible for Bob to
        // hide their tampering.
        {
            ts.next_tx(BOB);
            let k: Key = ts.take_from_sender();
            let l: Locked<Coin<BFC>> = ts.take_from_sender();
            let mut c = lock::unlock(l, k);

            let _dust = c.split(1, ts.ctx());
            let (l, k) = lock::lock(c, ts.ctx());
            let escrow: Escrow<Coin<BFC>> = ts.take_shared();
            let c = escrow.swap(k, l, ts.ctx());

            transfer::public_transfer(c, BOB);
        };

        abort 1337
    }

    #[test]
    fun test_return_to_sender() {
        let mut ts = ts::begin(@0x0);

        // Alice puts up the object they want to trade
        let cid = {
            ts.next_tx(ALICE);
            let c = test_coin(&mut ts);
            let cid = object::id(&c);
            let i = object::id_from_address(@0x0);
            create(c, i, BOB, ts.ctx());
            cid
        };

        // ...but has a change of heart and takes it back
        {
            ts.next_tx(ALICE);
            let escrow: Escrow<Coin<BFC>> = ts.take_shared();
            let c = escrow.return_to_sender(ts.ctx());

            transfer::public_transfer(c, ALICE);
        };

        ts.next_tx(@0x0);

        // Alice can then access it.
        {
            let c: Coin<BFC> = ts.take_from_address_by_id(ALICE, cid);
            ts::return_to_address(ALICE, c)
        };

        ts::end(ts);
    }

    #[test]
    #[expected_failure]
    fun test_return_to_sender_failed_swap() {
        let mut ts = ts::begin(@0x0);

        // Bob locks their object.
        let ik2 = {
            ts.next_tx(BOB);
            let c = test_coin(&mut ts);
            let (l, k) = lock::lock(c, ts.ctx());
            let kid = object::id(&k);
            transfer::public_transfer(l, BOB);
            transfer::public_transfer(k, BOB);
            kid
        };

        // Alice creates a public Escrow holding the object they are willing to
        // share, and the object they want from Bob
        {
            ts.next_tx(ALICE);
            let c = test_coin(&mut ts);
            create(c, ik2, BOB, ts.ctx());
        };

        // ...but then has a change of heart
        {
            ts.next_tx(ALICE);
            let escrow: Escrow<Coin<BFC>> = ts.take_shared();
            let c = escrow.return_to_sender(ts.ctx());
            transfer::public_transfer(c, ALICE);
        };

        // Bob's attempt to complete the swap will now fail.
        {
            ts.next_tx(BOB);
            let escrow: Escrow<Coin<BFC>> = ts.take_shared();
            let k2: Key = ts.take_from_sender();
            let l2: Locked<Coin<BFC>> = ts.take_from_sender();
            let c = escrow.swap(k2, l2, ts.ctx());

            transfer::public_transfer(c, BOB);
        };

        abort 1337
    }
}
```

共享对象情况下的协议不太对称，但仍然从第一方锁定他们想要交换的对象开始。

然后，第二方可以查看被锁定的对象，如果他们决定要与之交换，他们可以通过创建交换请求来表明他们的兴趣：

```
// examples/trading/contracts/escrow/sources/shared.move

public fun create<T: key + store>(
    escrowed: T,
    exchange_key: ID,
    recipient: address,
    ctx: &mut TxContext
) {
    let mut escrow = Escrow<T> {
        id: object::new(ctx),
        sender: ctx.sender(),
        recipient,
        exchange_key,
    };

    event::emit(EscrowCreated {
        escrow_id: object::id(&escrow),
        key_id: exchange_key,
        sender: escrow.sender,
        recipient,
        item_id: object::id(&escrowed),
    });

    dof::add(&mut escrow.id, EscrowedObjectKey {}, escrowed);

    transfer::public_share_object(escrow);
}
```

这次 `create` 请求直接接受托管的对象（不锁定），并创建一个共享的 `Escrow` 对象。该请求会记住发送它的地址（如果交换尚未发生，则允许谁回收该对象）以及预期的接收者，然后预计该接收者将通过提供他们最初锁定的对象来继续交换：

```
// examples/trading/contracts/escrow/sources/shared.move

/// The `recipient` of the escrow can exchange `obj` with the escrowed item
public fun swap<T: key + store, U: key + store>(
    mut escrow: Escrow<T>,
    key: Key,
    locked: Locked<U>,
    ctx: &TxContext,
): T {
    let escrowed = dof::remove<EscrowedObjectKey, T>(&mut escrow.id, EscrowedObjectKey {});

    let Escrow {
        id,
        sender,
        recipient,
        exchange_key,
    } = escrow;

    assert!(recipient == ctx.sender(), EMismatchedSenderRecipient);
    assert!(exchange_key == object::id(&key), EMismatchedExchangeObject);

    // Do the actual swap
    transfer::public_transfer(locked.unlock(key), sender);

    event::emit(EscrowSwapped {
        escrow_id: id.to_inner(),
    });

    id.delete();

    escrowed
}
```

即使 `Escrow` 对象是任何人都可以访问的共享对象，Move 接口也确保只有原始发送者和目标接收者才能成功与其交互。 `swap` 检查锁定的对象是否与创建 `Escrow` 时请求的对象匹配（再次通过比较密钥 ID），并假设预期接收者想要托管对象（如果他们没有，他们就不会调用 `swap` ）。

假设所有检查都通过，则提取 `Escrow` 中保存的对象，删除其包装器并将其返回给第一方。第一方提供的锁定对象也被解锁并发送给第二方，完成交换。

### 比较

本主题探讨了实现两个对象之间交换的两种方法。在这两种情况下，都会存在一方提出请求而另一方尚未做出回应的情况。此时，双方可能都想访问 `Escrow` 对象：一方取消交换，另一方完成交换。

在一种情况下，协议仅使用拥有的对象，但需要托管人充当中介。这样做的优点是完全避免了共识的成本和延迟，但涉及更多步骤，并且需要信任第三方的活性。

在另一种情况下，该对象被托管在共享对象的链上。这需要达成共识，但涉及的步骤较少，并且不需要第三方。
