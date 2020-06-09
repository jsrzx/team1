# 第三课作业  PoE 2

[TOC]

课程里会给出参考资料，大家一定要自己敲一遍**代码**！

注：

1. 提交源代码，运行`cargo test`的测试结果截图，前端UI的截图；
2. 测试应覆盖所有的业务逻辑，如不同的错误场景，以及一切正常的情况下，检查存储的数据是不是预期的那样。
3. 附加题不是必答的，但可以酌情加分。
4. 代码修改在本目录 substrate-node-template 和 substrate-front-end-template 的程序文件里。

## 第一题：编写存证模块的单元测试代码

### 1.1 创建存证的测试用例

#### （1）编写测试用例

```rust
use super::*;
use crate::{mock::*, Error};
use frame_support::{assert_noop, assert_ok};

// test cases for create_claim
// 创建存证成功
#[test]
fn create_claim_works() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1];
        assert_ok!(PoeModule::create_claim(Origin::signed(1), claim.clone()));
        assert_eq!(
            Proofs::<Test>::get(&claim),
            (1, system::Module::<Test>::block_number()),
        );
    })
}

// 创建存证失败：存证内容重复
#[test]
fn create_claim_failed_when_claim_already_exist() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1];
        let _ = PoeModule::create_claim(Origin::signed(1), claim.clone());

        // 判断错误码
        assert_noop!(
            PoeModule::create_claim(Origin::signed(1), claim.clone()),
            Error::<Test>::ProofAlreadyExist
        );
    })
}

// 创建存证失败：存证内容过长
#[test]
fn create_claim_failed_when_claim_is_too_long() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1, 2, 3, 4, 5, 6];

        assert_noop!(
            PoeModule::create_claim(Origin::signed(1), claim.clone()),
            Error::<Test>::ProofTooLong
        );
    })
}
```

#### （2）执行测试用例

![image-20200609115456358](imgs/image-20200609115456358.png)

### 1.2 撤销存证的测试用例

#### （1）编写测试用例

```rust
use super::*;
use crate::{mock::*, Error};
use frame_support::{assert_noop, assert_ok};

// test cases for revoke_claim
// 撤销存证成功
#[test]
fn revoke_claim_works() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1];
        let _ = PoeModule::create_claim(Origin::signed(1), claim.clone());

        // 判断错误码
        assert_ok!(PoeModule::revoke_claim(Origin::signed(1), claim.clone()));
    })
}

// 撤销存证失败：存证不存在
#[test]
fn revoke_claim_failed_when_claim_is_not_exist() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1];

        assert_noop!(
            PoeModule::revoke_claim(Origin::signed(1), claim.clone()),
            Error::<Test>::ClaimNotExist
        );
    })
}

// 撤销存证失败：存证非所有者
#[test]
fn revoke_claim_failed_with_wrong_owner() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1];

        let _ = PoeModule::create_claim(Origin::signed(1), claim.clone());

        assert_noop!(
            PoeModule::revoke_claim(Origin::signed(2), claim.clone()),
            Error::<Test>::NotClaimOwner
        );
    })
}
```

#### （2）执行测试用例

![image-20200609120403443](imgs/image-20200609120403443.png)

### 1.3 转移存证的测试用例

#### （1）编写测试用例

```rust
use super::*;
use crate::{mock::*, Error};
use frame_support::{assert_noop, assert_ok};

// test cases for transfer_claim
// 转移存证成功
#[test]
fn transfer_claim_works() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1];
        let _ = PoeModule::create_claim(Origin::signed(1), claim.clone());

        assert_ok!(PoeModule::transfer_claim(
            Origin::signed(1),
            claim.clone(),
            2
        ));

        assert_eq!(
            Proofs::<Test>::get(&claim),
            (2, system::Module::<Test>::block_number()),
        );
    })
}

// 转移存证失败：存证不存在
#[test]
fn transfer_claim_failed_when_claim_is_not_exist() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1];

        assert_noop!(
            PoeModule::transfer_claim(Origin::signed(1), claim.clone(), 2),
            Error::<Test>::ClaimNotExist
        );
    })
}

// 转移存证失败：存证非所有者
#[test]
fn transfer_claim_failed_with_wrong_owner() {
    new_test_ext().execute_with(|| {
        let claim = vec![0, 1];

        let _ = PoeModule::create_claim(Origin::signed(1), claim.clone());

        assert_noop!(
            PoeModule::transfer_claim(Origin::signed(2), claim.clone(), 3),
            Error::<Test>::NotClaimOwner
        );
    })
}
```

#### （2）执行测试用例

![image-20200609123647209](imgs/image-20200609123647209.png)





## 第二题：编写存证模块的UI

### 2.1 创建存证的UI





### 2.2 删除存证的UI





### 2.3 转移存证的UI



## 第三题（附加题）：实现购买存证的功能代码

* 用户A为自己的某个存证记录设置价格；
* 用户B可以以一定的价格购买某个存证，当出价高于用户A设置的价格时，则以用户A设定的价格将费用从用户B转移到用户A，再将该存证进行转移。如果出价低于用户A的价格时，则不进行转移，返回错误。

