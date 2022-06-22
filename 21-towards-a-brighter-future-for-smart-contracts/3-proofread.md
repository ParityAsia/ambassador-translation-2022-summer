---
proofreader: Jimmy Chu (jimmy.chu@parity.io)
completion-date: 2022-06-22
---

# 迈向智能合约更光明的未来

您最近可能听说在 Parity，我们的以太坊 Kovan 测试网现在支持用 WebAssembly 编写的智能合约。在以太坊基金会火山巢穴(volcano lair)的深处，开始了向以太坊本身整合相同能力的活动。事后我意识到我的措辞会使那成为“(bowel movement)排便”，但我没有改变它。继续。

目前，为以太坊编写智能合约的唯一可行方法是使用为 EVM 构建的语言。Solidity、Bamboo、Vyper。这些不是适用于以太坊的通用语言，它们是从头开始构建以在以太坊上使用的。这是因为 EVM 的执行语义，从外交上讲，是疯了。它有很多向后兼容的伤痕，而且很多特性（比如 256 位算术和有限的类型选择）是你不禁要暴露给程序员的东西。在以太坊环境有严格限制的环境中隐藏它太复杂了。您必须尽可能多地暴露底层系统，以便让程序员针对 gas 价格进行优化。

然而，这也有很多缺点——Solidity 是迄今为止最流行和最受支持的 EVM 语言，它的工具支持仍然落后于在它之后创建的通用语言，例如 Go 和 Rust。Solidity 存在 linter，但它们的规则相对简单，其中许多作为标准包含在更流行语言的编译器中（根本不需要外部工具）。可以编写更复杂的 linting 规则，但这会占用开发人员宝贵的时间，尽管我们喜欢假装以太坊正在接管世界，但事实是，愿意并且能够为 EVM 语言编写复杂工具的开发人员池是小的。

不仅如此，Solidity 还特别需要工具，因为它是一种具有许多枪和笨拙特性的语言。新旧智能合约开发人员很容易犯非常小的错误，从而造成重大的损失。

理想的情况是为区块链提供通用语言的工具支持和专业语言设计，而这正是WebAssembly可以帮助提供的。WebAssembly 是一个 VM 目标（与大多数虚拟机不同），它通常会尝试匹配 ARM 或 x86 等 CPU 架构的语义，而不是尝试匹配在其上运行的语言的语义。因此，许多语言可以原封不动地编译成它。由于大多数编程语言都是为了在某种程度上公开 CPU 的语义而构建的，因此让 VM 模拟 CPU 是一个很好的最低公分母。它还允许我们使用为 x86 和 ARM 等 ISA 构建的世界级优化编译器 - 当您按指令收费时非常重要。solc有一个--optimize标志，但据我所知，它所做的只是从程序集元数据中允许的目标列表中删除未使用的跳转目标，这更像是一种安全功能而不是优化。当然，它甚至落后于业余爱好者 C 编译器，也落后于 LLVM 之类的东西，它通常比人类手工编译相同的代码编写更好的汇编。

然而，一个缺点是这些通用语言不是为编写智能合约而构建的。然而，这可以通过正确的库来解决。如果你完全关注 Parity，你可能会发现我们非常喜欢 Rust，因为我们几乎所有的代码都是用它编写的. 我们在 Rust 中编写智能合约的实验已经有很长时间了，但是 API 仍然非常笨重。您必须手动将要存储的任何状态转换为 256 位数字（因为这是 EVM 存储的内容），并且由于我们在编译时生成代码，因此为 Rust 构建的许多工具不会工作正常，这否定了我们一开始就想使用通用语言的大部分原因。使用当前的 API，要编写一个只对存储的数字进行基本算术的简单合约，您需要编写大量代码：

```rs
pub mod foo_contract {
  use pwasm_ethereum;
  use pwasm_std::*;
  use pwasm_std::hash::H256;
  use bigint::U256;

  use pwasm_abi_derive::eth_abi;

  static STATE_KEY: H256 = H256([
    2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
    0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0
  ]);

  #[eth_abi(FooEndpoint)]
  pub trait FooInterface {
    /// This is called when the contract gets deployed on-chain
    fn constructor(&mut self, initial: U256);
    /// Add a number to the total
    fn add(&mut self, to_add: U256);
    /// Get the total
    fn get(&mut self) -> U256;
  }

  pub struct FooContract;

  impl FooInterface for FooContract {
    fn constructor(&mut self, initial: U256) {
      pwasm_ethereum::write(
        &STATE_KEY,
        &initial.into()
      );
    }

    fn add(&mut self, to_add: U256) {
      let current_state: U256 =
        pwasm_ethereum::read(&STATE_KEY).into();

      let new_state = current_state + to_add;

      pwasm_ethereum::write(
        &STATE_KEY, &new_state.into()
      );
    }

    fn get(&mut self) -> U256 {
      pwasm_ethereum::read(&STATE_KEY).into()
    }
  }
}

// We have to import a trait in order to use `dispatch`
// and `dispatch_ctor`.
use pwasm_abi::eth::EndpointInterface;

#[no_mangle]
pub fn call() {
  let mut endpoint = foo_contract::FooEndpoint::new(
    foo_contract::FooContract
  );

  pwasm_ethereum::ret(
    &endpoint.dispatch(&pwasm_ethereum::input())
  );
}

#[no_mangle]
pub fn deploy() {
  let mut endpoint = foo_contract::FooEndpoint::new(
    foo_contract::FooContract
  );

  endpoint.dispatch_ctor(&pwasm_ethereum::input());
}
```

一句话，这太可怕了。样板代码的行数比实际逻辑代码行数多 10 比 1（大约 3 行实际逻辑代码埋在 50 多行样板代码中）。它有一堆根本没有用的绒毛——这个参数永远不会被使用，因为所有的状态都是用andself读写的。您需要重写and函数。你没有办法弄清楚pwasm_ethereum::readpwasm_ethereum::write#[no_mangle]calldeploypwasm_ethereum::ret(&endpoint.dispatch(&pwasm_ethereum::input()))没有教程的咒语，它是完全无法发现的。如果不编译整个二进制文件，您将无法获得可靠的代码完成，因为它使用过程宏（可以运行任意代码）。不仅如此，您还需要过程宏，因为没有它，您将没有机会正确处理函数调用，除了过程宏实际上为您做的唯一事情是将传入的方法分派到正确的位置并反序列化输入/序列化输出。另一个令人沮丧的事情是，尽管 Rust 对强制不变性有极好的支持（比 Solidity 的等价物要好得多），但我们从未在这里使用它。我们只取&mut self一切，然后永远不要使用它。这是一个很好的第一次尝试，让 Rust 代码真正在我们的测试网上运行是令人兴奋的，但要让普通人用于真正的智能合约开发还有很长的路要走。

然而，上周，我开始勾勒出一个更好的智能合约 API 应该是什么样子的想法。我有几个目标：

* 智能合约代码应该易于阅读和编写。
* 我们应该能够在编译时捕获明显的错误，并且应该很容易为编译时无法捕获的任何内容编写 linter。
* 它应该编译成非常紧凑的代码，以确保部署智能合约的成本低廉。
* 即使没有教程（仅使用rustdoc），也应该很容易发现如何使用 API 。
* 我们应该尽可能地与生态系统的其他部分集成，以便我们可以重用他们的工具并避免重新实现已经在其他地方实现的代码。

考虑到这一点，这就是我想象以前的合同使用这个新 API 的样子（我选择的 crate 名称只是pwasm，但那是为了骑自行车）：

```rs
#[macro_use]
extern crate pwasm;
#[macro_use]
extern crate serde_derive;
extern crate serde;

use pwasm::{Contract, ContractDef};

#[derive(Serialize, Deserialize)]
pub struct State {
  current: U256,
}

contract!(foo_contract);

fn foo_contract() -> impl ContractDef<State> {
  messages! {
    Add(U256);
    Get() -> U256;
  }

  Contract::new()
    .constructor(|_txdata, initial| State {
      current: initial,
    })
    // These type annotations aren't necessary, they're just
    // to show that we don't receive a mutable reference to the
    // state in the `Get` method.
    .on_msg_mut::<Add>(|_env: &mut EthEnv, state: &mut State, to_add| {
      state.current += to_add;
    })
    .on_msg::<Get>(|_env: &EthEnv, state: &State, ()| state.current.clone())
}
```

好多了，我想。起初它可能看起来很奇怪，因为它不再像在类型上定义函数，但我决定摆脱，嗯，方法有一个方法。首先，使用类型来表示方法可以更轻松地定义外部合约的惯用接口（请参阅const NAME下面的宏扩展中的 ）。如果外部合约使用与 Rust 不同的命名约定（如果您调用 Solidity 合约，它可能会这样做），您不必让暴露给 Rust 的方法名称直接与外部合约匹配。这样做也使我们能够更好地利用 Rust 的工具，我们Contract免费获得文档（以rustdoc)，而我们必须手动维护有关宏语法的文档。使用此 API，新功能可以由使用 trait 系统的库完成，而默认情况下不组合程序宏。例如，您可以实现一个库，用于将合约定义为状态机，而无需触及其pwasm自身的代码。

看待这一点的一种方法是，它与 Web 服务器的语义相匹配，例如express，您有不可变的GET请求和可变POST的请求，所有这些都具有明确定义的路径上的端点并具有明确定义的输入类型。您在 Web 服务器中访问的路径对应于合同中消息的概念。例如，在 Rust 中，您可以使用actix-webcrate 创建一个如下所示的服务器：

```rs
server::new(|| {
  App::new()
    .route(
      "/add/{a}/{b}",
      http::Method::GET,
      |info: Path<(u32, u32)>| {
        format!("{}", info.0 + info.1)
      }
    )
  }).bind("127.0.0.1:8080")
    .unwrap()
    .run();
```

看起来pwasm像这样：

```rs
messages! {
  Add(u32, u32) -> u32;
}

Contract::new()
  .on_msg::<Add>(|_, _, (a, b)| a + b)
```

它使用serde（Rust 的标准序列化和反序列化框架）而不是手动滚动我们自己的序列化。手动序列化对我们自己的类型来说很好，但这意味着如果我们想要允许用户序列化他们自己的类型，我们要么需要他们编写大量样板文件，要么我们需要serde_derive为我们自己的序列化方法编写等效的，复制工作。它使用 Rust 内置的可变性保证来强制我们不对不需要它的函数进行任何突变。\_mut默认情况下它是不可变的，如果您想要一个可变方法，则必须编写。这可以防止意外突变导致的错误，并且如果我们知道该方法无法对其进行突变，也可以让我们避免花费气体来保存状态。和contract宏messages只是macro_rules!宏，你可以很简单地编写没有它们的合约。它们只是扩展为以下内容：

```rs
// ...

#[no_mangle]
fn __pwasm_ethereum_call() {
  pwasm::ret(
    foo_contract().call(pwasm::read_state(), pwasm::input())
  );
}

#[no_mangle]
fn __pwasm_ethereum_deploy() {
  let state = foo_contract().construct(pwasm::input());
  pwasm::write_state(state);
}

fn foo_contract() -> impl ContractDef<State> {
  struct Add;
  struct Get;

  impl pwasm::Message for Add {
    type Input = U256;
    type Output = ();

    const NAME: &str = "Add";
  }

  impl pwasm::Message for Get {
    type Input = ();
    type Output = U256;

    const NAME: &str = "Get";
  }

  // ...
}
```

此外，合约的方法采用EthEnv代表区块链环境本身的参数，可用于（例如）获取当前区块头或向另一个合约发送消息。我们可以再次使用 Rust 的可变性保证来强制不可变方法不能调用可变方法等。我们还使用一个简单的类型级状态机来确保您编写的构造函数不超过一个（尽管在 Rust 的类型系统中还不可能阻止您为同一消息编写两个处理程序）。

如此多的常规 Rust 代码的另一个好处是，您可以转到Contract文档中的页面，并很容易地从中推断出如何编写合约。由于您需要通过env参数来访问区块链，因此只需转到EthEnv. 没有操纵全局状态的顶级函数。

最后，它无需任何不必要的开销即可编译为代码，因为它广泛使用了 Rust 的泛型。Plus constructor, on_msgandon_msg_mut都是const fns，所以合约是在编译时而不是运行时创建的（就像宏一样）。与编写使用现有 API 的详细版本相比，该 API 没有任何开销。

以这种抽象为基础，应该很容易在上面实现其他功能。例如，如果您需要一个键/值存储并且您不希望每次访问它时都必须序列化和反序列化整个事物（例如，如果您正在创建 ERC20 令牌并且您想要存储映射在地址和余额之间）你可以有一个Database在内部使用原始调用的类型，pwasm_ethereum::read但在pwasm_ethereum::write外部表现得更像 Rust 的 stdlibHashMap类型。这也将允许您做当前 API 中不可能做的事情，例如拥有两个可以具有重叠键的数据库（通过对Database类型的每个实例使用唯一的随机数）。这可以在我在这里描述的 API 之上实现为一个库，代码相对较少。

我应该说，你在这里看到的一切都不是一成不变的。我一直在考虑为智能合约编写一个新的 API，我想从真正的智能合约开发人员和对编写一些智能合约感兴趣的 Rust 程序员那里获得一些反馈。我自己不是智能合约开发人员，我来自我们的 Rust 团队，但我对让智能合约更安全、更容易编写感兴趣。我很想听听您对这里可以改进的任何东西的想法，尤其是在人体工程学方面。我不是想让它看起来像 Solidity 代码，我只是想让它安全且可读。如果您想了解如何实现它，可以在我的 GitHub 帐户上找到一个尚未在实际区块链上运行的简单工作但非常不完整的实现。
