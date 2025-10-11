# 第九章：测试 JavaScript 应用

要开始使用 JavaScript 进行测试，我们需要创建一个 ReactJS 应用，并使用 Mocha、Chai、Enzyme 和 Sinon 库来配置它进行测试。

这些步骤在第三章中已详细讨论，“设置 JavaScript 环境”，因此在这里，我们只需概述这些步骤，而不进行详细解释。

本章的目标是：

+   创建演讲者遇见 React 应用

+   讨论我们测试应用的行动计划：

    +   我们的方法是什么？

    +   我们甚至可以测试应用的部分是什么？

    +   我们从应用的哪个部分开始？

+   为应用编写测试并完成几个功能：

    +   演讲者列表

    +   演讲者详情

一旦这一章完成，你应该能够对任何基于 React 的应用进行单元测试。

# 创建 React 应用

对于本书中的应用，为了保持兼容性，您希望使用 Node.js 版本 8.5.0、NPM 版本 5.4.2 和 create-react-app 版本 1.4.0。

执行以下命令来安装和执行应用：

```cs
>npm install
>npm test
>npm start
```

所有三个命令都应该成功运行。运行`npm test`后，您需要通过按`<q>`退出测试运行。运行`npm start`后，您需要通过按*Ctrl* + *C*退出服务器。

# 推出应用

假设前面的步骤没有遇到任何问题，我们可以继续执行将 React 应用“推出”的操作。同样，正如在第三章中详细解释的，“设置 JavaScript 环境”，我们在这里只做简要回顾。

推出应用只有一个命令。推出后，我们将重新运行上一节中的命令，以确保应用仍然按预期工作。

执行以下命令来推出：

```cs
>npm run eject
```

# 配置 Mocha、Chai、Enzyme 和 Sinon

现在，我们准备添加我们希望用于此应用的测试设施。与之前一样，这些实用工具的添加已在之前的章节中详细说明。因此，我们只提供执行命令和要安装的包的版本。

执行以下命令来安装我们将要使用的库：

```cs
>npm install mocha@3.5.3
>npm install chai@4.1.2
>npm install enzyme@2.9.1
>npm install sinon@3.2.1
```

我们还将使用一些其他库作为我们 Redux 工作流程的一部分：

```cs
>npm install nock@9.0.1
>npm install react-router-dom@4.2.2
>npm install redux@3.7.2
>npm install redux-mock-store@1.3.0
>npm install redux-thunk@2.2.0
```

在安装命令中包含版本将确保您使用与我们相同的库版本，并将减少潜在问题的数量。

要使用我们刚刚安装的库，我们还需要安装一个额外的 babel 预设：

```cs
>npm install babel-preset-es2015@6.24.1
```

在`package.json`中更新您的 babel 配置，以删除 react-app 并包括`react`和`es2015`。

```cs
"babel": {
   "presets": [
     "react",
     "es2015"
   ]
 },
```

如第三章中所述，“设置 JavaScript 环境”，从`package.json`中删除测试配置部分。然后，更新测试脚本为：

```cs
"test": "mocha --require ./scripts/test.js --compilers babel-core/register ./src/**/*.spec.js"
```

并添加一个测试监视脚本：

```cs
"test:watch": "npm test -- -w"
```

我们现在可以更新脚本文件夹中的测试执行文件`test.js`，使其与 Mocha 兼容。将文件中的所有内容更改为：

```cs
'use strict'; 

import jsdom from 'jsdom'; 
global.document = jsdom.jsdom('<html><body></body></html>'); 
global.window = document.defaultView; 
global.navigator = window.navigator; 

function noop() { 
  return {}; 
}

// prevent mocha tests from breaking when trying to require a css file 
require.extensions['.css'] = noop; 
require.extensions['.svg'] = noop;
```

在我们能够使用新的测试库之前，最后一步是更新`App.test.js`文件以匹配与 Mocha 和 Chai 一起使用的约定。因此，将文件名更改为`App.spec.js`，并将内容更新为如下所示：

```cs
import React from 'react'; 
import ReactDOM from 'react-dom'; 
import { expect } from 'chai'; 

import App from './App'; 

describe('(Component) App', () => { 
  it('renders without crashing', () => { 
        const div = document.createElement('div'); 
        ReactDOM.render(<App />, div); 
    }); 
}); 
```

现在，就像之前一样，执行测试脚本并启动应用程序，以确保在转换到 Mocha 的过程中没有出现任何问题。

```cs
>npm test
>npm run test:watch
>npm start
```

这三个命令都应该能正常工作。如果你遇到问题，检查我们刚刚讨论的所有步骤，并查看第三章，*设置 JavaScript 环境*，以获取更详细的解释。

# 计划

现在我们测试配置已经更新并且运行正确，我们可以开始考虑测试驱动我们的第一个功能。

在前面的章节中，我们讨论了从哪里开始测试，并决定如果可能的话，一个从内到外的方法更受欢迎。为了保持这种方法，我们想要确定我们 React 应用程序的不同部分，这样我们就可以针对最纯粹的业务逻辑。

首先，无论其他架构选择如何，我们都可以识别出 React 组件和代表与我们的数据源通信的服务。我们计划在这个应用程序中使用 Redux，这使得它成为缺失的部分，并将我们的组件与我们的数据连接起来。

那么这些中的哪一个代表业务逻辑呢？从这些基本选项中，我们会测试什么呢？让我们更仔细地检查每一个，看看我们能测试什么，这将被视为单元测试。

# 考虑 React 组件

通常，我们想要避免对第三方库进行单元测试。因此，让我们将 React 组件的第三方方面与我们可能进行单元测试的部分分开。

第三方方面包括任何继承的功能和特性；这包括一定程度上的任何生命周期方法和 JSX。那么，剩下什么呢？这个问题的答案取决于所讨论的组件是呈现组件还是容器组件。

呈现组件几乎是纯 HTML 和视图机制。几乎没有传统上可进行单元测试的行为。当然，没有真正的业务逻辑。

容器组件是 React 应用程序中真正发生动作的地方。这些组件可以操作数据并做出业务决策，这些决策可以控制应用程序的流程。因此，让我们将容器组件保留在可能的单元测试起点列表中。

# 查看 Redux 的可测试性

Redux 是一个第三方库，它控制着整个应用中的数据流，并管理了我们可能想要进行单元测试的大量正常数据交换。因为它是第三方库，所以表面上似乎没有太多可以单元测试的。让我们更仔细地看看 Redux 数据流的各个方面，以确定是否真的没有什么可以测试的，或者我们是否仍然需要单元测试 Redux 的部分。

# Store

Redux store 是应用获取所有数据后数据所在的地方。通常，每个使用 Redux 的应用只有一个 store。store 几乎完全包含在 Redux 库中，我们与它的直接交互非常少。因此，似乎没有太多我们可以或必须为 store 进行测试的，它完全属于我们必须简单地信任的第三方代码领域。

# Actions

Redux 中的 Actions 代表携带数据包的事件。这个事件通常是一个命令，用于从数据源中检索或更新数据，这些数据应该由 store 反映出来。因为 actions 只是一个带有一些数据的键，所以在这里似乎没有太多可以测试的。

# Reducers

如果在 Redux 交互中有什么可以测试的，那很可能是在 reducers 中。Reducers 接收 actions，并根据请求的 actions 和作为这些 actions 一部分提供的数据来确定要做什么，如果需要的话。

通常，一旦确定了适当的 service 调用，reducer 将简单地调用 API 服务。有可能 reducer 也可能将接收到的数据映射到更适合必须进行的 service 调用的格式。

那么，如果实际上 reducer 只是将要调用服务，我们应该为 reducer 测试什么？除了确保使用适当的数据调用适当的 service 方法之外，似乎没有太多可以测试的。为了完整性，我们可能想要测试这些事情，但它们并不代表我们的业务逻辑的核心。

总之，Redux 中似乎没有太多可以测试的内容，而且可以测试的内容并不代表我们的业务逻辑的核心。

# 单元测试 API 服务

最后，让我们看看 API 服务。通常，前端应用中的服务表现得就像后端应用中的仓库一样。服务的主要功能是抽象与某些数据源的数据交互。这些交互不一定包含任何可定义的业务逻辑。如果有的话，服务的真实逻辑存在于服务器上，并且不需要作为前端应用的一部分进行测试。至少，它不需要以你可能会认为的方式进行测试。

那么，如果服务不包含任何业务逻辑，Redux 也不包含太多业务逻辑，组件也不包含太多业务逻辑，我们应该测试什么，以及如何进行单元测试呢？

简短的回答是，我们还没有摆脱测试的责任，但我们必须跳过一些障碍才能进行任何测试，因为从集成测试中抽身是很困难的。在典型的前端应用程序中，与 C#不同，我们的代码和他们的代码之间没有明确的界限。因此，我们将不得不做出一些妥协，并编写大量的代码来抽象第三方代码的部分，以便我们可以测试我们需要测试的内容。

那么，当我们谈论测试方向时，这又把我们带向何方呢？遗憾的是，似乎没有明确的胜者。为了这个应用程序的目的，我们将从数据源开始工作，这样在我们编写应用程序的用户界面方面时，我们可以清楚地了解我们可用的数据操作。

# 演讲者列表

按照我们的 C#后端的功能，我们首先将测试可用的演讲者列表。我们还没有准备好连接到后端，并且对于我们将要在这里编写的任何测试（作为单元测试），我们需要模拟后端通常会呈现的行为。

目前，我们不会关心任何类型的身份验证。因此，我们将要实现的重要功能是，当没有演讲者存在时，我们应该让用户知道，当演讲者存在时，我们应该列出他们。

我们将产生这两种情况的方式是通过模拟 API。虽然这听起来可能很奇怪，但我们的大部分业务逻辑将位于模拟 API 中。因为它对于我们将要编写的所有其他测试都至关重要，我们必须像对待生产代码一样对模拟 API 进行单元测试。

# 模拟 API 服务

为了开始测试模拟 API 服务，让我们创建一个新的服务文件夹，并添加一个`mockSpeakerService.spec.js`文件。

在该文件中，我们需要导入我们的断言库，创建我们的初始 describe，并编写一个存在性测试。

```cs
import { expect } from 'chai';

describe('Mock Speaker Service', () => {
  it('exits', () => {
    expect(MockSpeakerService).to.exist;
  });
});
```

使用带有 watch 的 npm 测试脚本开始。我们刚刚编写的测试应该失败。为了使测试通过，我们必须创建一个`MockSpeakerService`对象。让我们扮演一下魔鬼的代言人，在这个文件中创建一个对象，但只创建足够使测试通过的对象。

```cs
let MockSpeakerService = {};
```

这行代码通过了目前失败的测试，但显然不是我们想要的。然而，它确实迫使我们编写更健壮的测试。我们可以编写的下一个测试是证明`MockSpeakerService`可以被构造。这个测试应该确保我们已经将`MockSpeakerService`定义为一个类。

```cs
it('can be constructed', () => {
  // arrange
  let service = new MockSpeakerService();

  // assert
  expect(service).to.be.an.instanceof(MockSpeakerService);
});
```

这个测试失败了，显示`MockSpeakerService`不是一个构造函数。解决这个问题的方式是将`MockSpeakerService`改为一个类。

```cs
class MockSpeakerService {
}
```

现在我们有一个可以实例化的类，接下来我们编写的测试可以开始测试实际的功能。那么，我们将测试哪些功能呢？查看需求，我们可以看到第一个场景涉及请求所有演讲者并接收没有演讲者。这是一个相对简单的测试场景。我们在`MockSpeakerService`中会称什么函数为获取所有演讲者的函数呢？因为我们试图获取所有演讲者，一个简单且不重复且符合我们在 C#后端讨论的存储库模式的名称是简单地`getAll`。让我们创建一个嵌套的 describe 和一个`getAll`类方法的存活性测试。

```cs
describe('Get All', () => {
  it('exists', () => {
    // arrange
    let service = new MockSpeakerService();

    // assert
    expect(service.getAll).to.exist;
  });
});
```

如同往常，这个测试应该失败，并且应该以`expected undefined to exist`失败。让这个测试通过相对简单，只需在`MockSpeakerService`类中添加一个`getAll`方法。

```cs
class MockSpeakerService {
  getAll() {
  }
}
```

下一步我们需要决定的是当没有演讲者时我们应该期望什么结果。回顾后端，当没有演讲者时，我们应该收到一个空数组。查看需求，系统应该显示`NO_SPEAKERS_AVAILABLE`消息。服务应该负责显示这个消息吗？在这种情况下，答案是不了。当到达代码的这一部分时，react 组件应该负责显示`NO_SPEAKERS_AVAILABLE`消息。现在，我们应该期望，当没有演讲者存在时，接收一个空的数据集。

因为我们在扩展测试上下文，让我们为这个上下文扩展创建另一个 describe。

```cs
describe('No Speakers Exist', () => {
  it('returns an empty array', () => {
    // arrange
    let service = new MockSpeakerService();

    // act
    let promise = service.getAll();

    // assert
    return promise.then((result) => {
      expect(result).to.have.lengthOf(0);
    });      
  });
});
```

注意我们在这个测试中使用的语法。我们在`then`函数中返回承诺并做出断言。这是因为我们希望我们的测试在服务中的异步代码上操作。大多数后端操作将需要异步执行，处理这种异步性的一个惯例是使用承诺。在 Mocha 中，异步测试，即处理承诺的测试，要求承诺必须从测试中返回，这样 Mocha 就可以知道在关闭测试之前等待承诺解析。

现在为了让测试通过，我们只需要从`getAll`方法返回一个解析为空数组的承诺。在这里我们将使用一个零延迟的`setTimeout`，这样我们就可以为开发目的稍后实现某种延迟。我们想要延迟的原因是，这样我们就可以测试在慢速网络响应事件中 UI 的操作。

```cs
getAll() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(Object.assign([], this._speakers));
    }, 0);
  });
}
```

现在我们已经通过了第一个场景，并且有足够的代码可以进行重构。我们在多个地方声明了服务变量，并且我们没有表示该变量基线实例化的上下文。让我们创建一个 describe 来包裹所有后实例化测试，并添加一个`beforeEach`来初始化一个作用域在 describe 内的服务变量，并且使其对所有该 describe 内的测试可用。

这里是重构后的测试：

```cs
describe('Mock Speaker Service', () => {
   it('exits', () => {
     expect(MockSpeakerService).to.exist;
   });

   it('can be constructed', () => {
     // arrange
     let service = new MockSpeakerService();

     // assert
     expect(service).to.be.an.instanceof(MockSpeakerService);
   });

   describe('After Initialization', () => {
     let service = null;

     beforeEach(() => {
       service = new MockSpeakerService();
     });

     describe('Get All', () => {
       it('exists', () => {
         // assert
         expect(service.getAll).to.exist;
       });

       describe('No Speakers Exist', () => {
         it('returns an empty array', () => {
           // act
           let promise = service.getAll();

           // assert
           return promise.then((result) => {
             expect(result).to.have.lengthOf(0);
           });
         });
       });
     });
   });
 });
```

下一个场景，演讲者列表，是在演讲者存在时的情况。这个场景的第一个测试需要至少向模拟 API 添加一个演讲者。让我们在 `GetAll` 内创建一个新的 `describe`，但与 `No Speakers Exist` 分开。

```cs
describe('Speaker Listing', () => {
   it('returns speakers', () => {
     // arrange
     service.create({});

     // act
     let promise = service.getAll();

     // assert
     return promise.then((result) => {
       expect(result).to.have.lengthOf(1);
     });
   });
 });
```

作为这个测试设置的一部分，我们已经添加了对一个 `Create` 方法的引用。这个方法目前还不存在，我们的测试没有它无法通过。因此，我们需要暂时忽略这个测试，并编写 `Create` 的测试。我们可以通过跳过它来忽略这个测试。

```cs
it.skip('returns speakers', () => {
```

现在，我们可以在 `After Initialization` 块内为 `Create` 编写一个新的 `describe` 块。

```cs
describe('Create', () => {
   it('exists', () => {
     expect(service.create).to.exist;
   });
 });
```

为了使测试通过，我们向模拟服务类中添加了 `Create` 方法。

```cs
class MockSpeakerService {
   create() {

   }
 …
```

从这个点开始，我们可以编写一些测试来向 `Create` 方法添加验证逻辑。然而，我们目前没有任何引用 API 上 `Create` 方法的场景。由于这个方法仅用于测试目的，我们将让它保持原样，只进行存在性测试。让我们回到我们的场景测试。

现在 `Create` 存在了，我们应该收到测试预期的失败，即我们期望长度为 1，但实际长度为 0。从测试中移除 `skip` 并验证。

为了使这个测试通过，我们实际上必须实现创建的基本逻辑并对 `getAll` 进行修改。

```cs
class MockSpeakerService {
   constructor() {
     this._speakers = [];
   }

   create(speaker) {
     this._speakers.push(speaker);
   }

   getAll() {
     return new Promise((resolve, reject) => {
       setTimeout(() => {
         resolve(Object.assign([], this._speakers));
       }, 0);
     });
   }
 }
```

我们可以认为当前的测试足够了，可以开始测试我们的数据流。

# 获取所有演讲者的动作

要开始使用 Redux 进行测试，我们可以从几个测试入口点开始。我们可以从测试动作、reducer 或甚至与存储的交互开始。存储测试将更多是集成测试，而我们希望在这个章节中专注于单元测试。这留下了动作和 reducer。两者都是良好的起点，但我们将从动作开始，因为它们在测试概念上非常简单且不复杂。

我们现在需要的动作是请求检索演讲者信息的一个动作；本质上，就是一个获取所有演讲者的动作。如前所述，动作可以非常简单；然而，我们有一个问题，那就是我们的获取所有演讲者的服务调用是异步的。动作并不是真正设计来处理异步调用的。因此，让我们从一个稍微简单一点的东西开始，等我们了解了如何测试正常动作后，再回来解决这个问题。

# 测试标准动作

我们需要在演讲者加载完毕后通知 Redux 我们已经获取了演讲者。我们没有理由不能从这里开始。所以，让我们编写一个测试来验证成功检索演讲者。

```cs
import { expect } from 'chai';

 describe('Speaker Actions', () => {
   describe('Sync Actions', () => {
     describe('Get Speakers Success', () => {
       it('exists', () => {
         expect(getSpeakersSuccess).to.exist;
       });
     });
   });
 });
```

运行这个测试应该会失败。为了使测试通过，定义一个名为 `getSpeakersSuccess` 的函数。

```cs
function getSpeakersSuccess() {
 }
```

由于典型动作的简单性，我们的下一个测试实际上将测试动作的功能。我们可以将其分解成多个测试，但我们实际上只是在断言返回数据的结构。关于单一断言规则，我们仍然只断言了一件事情。

```cs
it('is created with correct data', () => {
   // arrange
   const speakers = [{
     id: 'test-speaker',
     firstName: 'Test',
     lastName: 'Speaker'
   }];

   // act
   const result = getSpeakersSuccess(speakers);

   // assert
   expect(result.type).to.equal(GET_SPEAKERS_SUCCESS);
   expect(result.speakers).to.have.lengthOf(1);
   expect(result.speakers).to.deep.equal(speakers);
 });
```

为了使这个测试通过，我们需要对我们当前实现的 `getSpeakersSuccess` 函数进行重大修改。

```cs
const GET_SPEAKERS_SUCCESS = 'GET_SPEAKERS_SUCCESS';

 function getSpeakersSuccess(speakers) {
   return { type: GET_SPEAKERS_SUCCESS, speakers: speakers };
 }
```

在 Redux 中，动作有一个预期的格式。它们必须包含一个类型属性，通常包含某些数据结构。在 `getSpeakersSuccess` 的情况下，我们的类型是一个常量，`GET_SPEAKERS_SUCCESS`，数据是传递给动作的演讲者数组。为了让它们对应用程序可用，让我们将动作和常量移动到它们自己的文件中。我们需要一个 `speakerActions` 文件和一个 `actionTypes` 文件，

`src/actions/speakerActions.js`:

```cs
import * as types from '../reducers/actionTypes';

 export function getSpeakersSuccess(speakers) {
   return { type: types.GET_SPEAKERS_SUCCESS, speakers: speakers };
 }
```

`src/reducers/actionTypes.js`:

```cs
export const GET_SPEAKERS_SUCCESS = 'GET_SPEAKERS_SUCCESS';
```

在测试中添加导入语句，并且所有测试都应该通过。对于一个典型的动作，这是测试的格式。在 `reducers` 文件夹中放置动作类型是为了依赖反转的原因。从 SOLID 的角度来看，`reducers` 定义了一个交互合同，这由动作类型表示。动作是履行这个合同的。

# 测试 `thunk`

由于 `getSpeakersSuccess` 动作旨在表示成功服务调用的结果，我们需要一种特殊类型的动作来表示服务调用本身。如前所述，Redux 并不固有地支持异步动作。因此，我们需要其他方式与后端进行通信。幸运的是，Redux 支持中间件，许多中间件已被设计为向 Redux 添加异步能力。我们将为了简单起见使用 `redux-thunk`。

要开始下一个测试，我们首先需要将 `redux-thunk` 和 `redux-mock-store` 导入到我们的演讲者动作测试中。

```cs
import thunk from 'redux-thunk';
 import configureMockStore from 'redux-mock-store';
```

然后我们可以测试获取演讲者。

```cs
describe('Async Actions', () => {
   describe('Get Speakers', () => {
     it('exists', () => {
       expect(speakerActions.getSpeakers).to.exist;
     });
   });
 });
```

如同往常，我们从存在性测试开始。而且，如同往常，使这个测试通过相当容易。在演讲者动作文件中，添加 `getSpeakers` 函数的定义并将其导出。

```cs
export function getSpeakers() {
 }
```

下一个测试比我们之前工作的测试稍微复杂一些，所以我们将更详细地解释它。

我们首先需要做的是配置一个模拟存储库并添加 `thunk` 中间件。我们需要这样做，因为为了正确测试 `thunk`，我们必须假装 Redux 正在运行，这样我们才能派发我们的新动作并检索结果。所以，让我们将我们的模拟存储库配置添加到 `Async Actions` 的 `describe` 中：

```cs
const middleware = [thunk];
 let mockStore;

 beforeEach(() => {
   mockStore = configureMockStore(middleware);
 });
```

现在我们有了可用的存储库，我们就可以开始编写测试了。

```cs
it('creates GET_SPEAKERS_SUCCESS when loading speakers', () => {
  // arrange
  const speaker = {
    id: 'test-speaker',
    firstName: 'Test',
    lastName: 'Speaker'
  };

  const expectedActions = speakerActions.getSpeakersSuccess([speaker]);
  const store = mockStore({
    speakers: []
  });
```

在安排阶段，我们配置了一个最基础的演讲者。然后，我们调用之前测试过的动作来构建合适的数据结构。最后，我们定义了一个模拟存储库及其初始状态。

```cs
  // act
   return store.dispatch(speakerActions.getSpeakers()).then(() => {
     const actions = store.getActions();

     // assert
     expect(actions[0].type).to.equal(types.GET_SPEAKERS_SUCCESS);
   });
 });
```

现在，当在 Mocha 中进行异步测试时，我们可以返回一个 promise，Mocha 将自动知道这个测试是异步的。对于异步测试，我们的断言放在 promise 的 resolve 或 reject 函数中。在获取演讲者动作的情况下，我们将假设服务器交互成功，并测试解析的 promise。

因为我们从`getSpeakers`动作中没有返回任何内容，`mockStore`抛出一个错误，指出动作可能不是 undefined。为了使测试继续进行，我们必须返回一些内容。为了朝着使用`thunk`的方向前进，我们需要返回一个函数。

```cs
export function getSpeakers() {
   return function(dispatch) {
   };
 };
```

添加一个什么也不做的函数的返回值将测试失败信息向前推进，现在呈现的是无法读取 undefined 的`then`属性。所以，现在我们需要从我们的动作中返回一个 promise。我们已经在模拟 API 服务中构建了服务端点，所以现在让我们调用它。

```cs
export function getSpeakers() {
   return function(dispatch) {
     return new MockSpeakerService().getAll().then(speakers => {
       dispatch(getSpeakersSuccess(speakers))
     }).catch(err => {
       throw(err);
     });
   };
 }
```

现在测试通过了，我们已经编写了第一个处理 thunks 的测试。如您所见，测试和通过测试的代码都相当容易编写。

# 获取所有演讲者的 reducer

现在我们已经测试了与获取所有演讲者相关的动作，是时候转向测试 reducers 了。像往常一样，让我们从一个存在性测试开始。

```cs
describe('Speaker Reducers', () => {
   describe('Speakers Reducer', () => {
     it('exists', () => {
       expect(speakersReducer).to.exist;
     });
   });
 });
```

为了使这个测试通过，我们只需要定义一个名为`speakersReducer`的函数。

```cs
function speakersReducer() {
 }
```

我们下一个测试将检查 reducer 的功能。

```cs
it('Loads Speakers', () => {
   // arrange
   const initialState = [];

   const speaker = {
     id: 'test-speaker',
     firstName: 'Test',
     lastName: 'Speaker'
   };
   const action = actions.getSpeakersSuccess([speaker]);

   // act
   const newState = speakersReducer(initialState, action);

   // assert
   expect(newState).to.have.lengthOf(1);
   expect(newState[0]).to.deep.equal(speaker);
 });
```

这个测试比我们通常喜欢的要大，所以让我们来分析一下。在安排阶段，我们配置初始状态并创建一个由单个演讲者组成的动作结果数组。当一个 reducer 被调用时，应用的前一个状态和动作的结果会被传递给它。在这种情况下，我们从一个空数组开始，修改是添加一个单个演讲者。

接下来，在测试的 *Act* 部分，我们调用 reducer，传入`initialState`和我们的动作调用结果。reducer 返回一个新的状态，供我们在应用中使用。

最后，在断言中，我们期望新状态由单个演讲者组成，并且演讲者具有与我们为动作创建的演讲者相同的数据。

为了使测试通过，我们需要处理传递给 reducer 的动作。

```cs
function speakersReducer(state = [], action) {
   switch(action.type) {
     case types.GET_SPEAKERS_SUCCESS:
       return action.speakers;
     default:
       return state;
   }
 }
```

因为在使用 Redux 的应用中，对于每个动作都会调用 reducer，我们需要确定对于任何不是我们想要处理的动作，我们应该做什么。在这些情况下，适当的响应是简单地返回未修改的状态。

对于我们想要处理的动作类型，在这种情况下，我们返回动作的演讲者数组。在其他 reducers 中，我们可能会将初始状态与动作结果合并，但对于获取演讲者成功，我们想要用我们接收的值替换状态。

最后一步，现在所有测试都通过了，我们需要将演讲者 reducer 从测试文件中提取出来，并将其移动到`speakerReducer.js`

# 演讲者列表组件

我们还可以测试应用的其他部分，即组件。在典型的 React + Redux 应用中，有两种类型的组件。我们有容器组件和表示性组件。

容器组件通常不包含任何真实的 HTML。容器组件的渲染函数只是引用单个表示性组件。

表示性组件通常不包含任何业务逻辑。它们接收属性并显示这些属性。

在我们从后端到前端的过程中，我们已经涵盖了数据的检索和更新。接下来，让我们看看将使用这些数据的容器组件。

我们的容器组件将是一个简单的组件。让我们从典型的存在性测试开始。

```cs
import { expect } from 'chai';
import { SpeakersPage } from './SpeakersPage';

 describe('Speakers Page', () => {
   it('exists', () => {
     expect(SpeakersPage).to.exist;
   });
 });
```

简单直接；现在来让它通过。

```cs
export class SpeakersPage {
 }
```

接下来是组件的渲染函数。

```cs
import React from 'react';
import Enzyme, { mount, shallow } from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';
import { SpeakersPage } from './SpeakersPage';

 describe('Render', () => {
   beforeEach(() => {
      Enzyme.configure({ adapter: new Adapter() });
   });

   it('renders', () => {
     // arrange
     const props = {
       speakers: [{ 
         id: 'test-speaker', 
         firstName:'Test', 
         lastName:'Speaker' 
       }]
     };

     // act  
     const component = shallow(<SpeakersPage { ...props } />);

     // assert
     expect(component.find('SpeakerList')).to.exist;                 
     expect(component.find('SpeakerList').props().speakers)
       .to.deep.equal(props.speakers);
   });
 });
```

这个测试引入了一些新的概念。从测试的 act 部分开始。我们使用 Enzyme 的浅渲染。浅渲染会渲染 React 组件，但不会渲染组件的子组件。在这种情况下，我们期望存在一个`SpeakerList`组件，并且这个组件正在渲染它。这里显示了 Enzyme 适配器的配置，但也可以在测试通过后将它移动到`test.js`文件中。

我们还在检查属性，以确保我们将演讲者传递给表示性组件。为了让这个测试通过，我们必须修改`SpeakersPage`组件，但我们也必须创建一个`SpeakerList`组件。现在让我们来做这件事。

```cs
export class SpeakersPage extends Component {
   render() {
     return (
       <SpeakerList speakers={this.props.speakers} />
     );
   }
 }
```

然后在新的文件中，我们需要添加`SpeakerList`。

```cs
export const SpeakerList = ({speakers}) => {}
```

你可能已经注意到我们的容器组件没有任何逻辑。事实上，它所做的只是渲染`SpeakerList`组件。如果它只做这些，为什么它是一个容器组件呢？原因在于这个组件将是一个 Redux 连接组件。我们希望将 Redux 代码保留在我们的业务逻辑中，而不是在我们的显示组件中。因此，我们将它视为一个高阶组件，只是用它来将数据传递给表示性组件。稍后，当我们到达演讲者详情组件时，你会看到一个带有少量业务逻辑的容器组件。

目前，我们的`SpeakerList`组件看起来有点瘦弱，并不能真正作为一个 React Redux 应用的一部分工作。是时候测试我们的表示性组件了。

```cs
describe('Speaker List', () => {
   it('exists', () => {
     expect(SpeakerList).to.exist;
   });
 });
```

由于上一个测试，这个测试将自动通过。通常情况下，如果我们遵循我们刚才所做的进度，我们不会编写这个测试。实际上，我们应该忽略之前的测试，创建这个测试，然后创建`SpeakerList`组件。之后，我们可以重新启用之前的测试并让它通过。

下一步是测试当演讲者数组为空时，是否渲染了没有可用演讲者的消息。

```cs
function setup(props) {
   const componentProps = {
     speakers: props.speakers || []
   };

   return shallow(<SpeakerList {...componentProps} />);
 }

 describe('On Render', () => {
   describe('No Speakers Exist', () => {
     it('renders no speakers message', () => {
       // arrange
       const speakers = [];

       // arrange
       const component = setup({speakers});

       // assert
       expect(component.find('#no-speakers').text())
         .to.equal('No Speakers Available.');
     });
   });
 });
```

对于这次测试，我们创建了一个辅助函数来初始化组件，并传入我们需要的属性。为了让测试通过，我们只需返回一个包含正确文本的`div`。

```cs
export const SpeakerList = ({speakers}) => {
   return (
     <div>
     <h1>Speakers</h1>
     <div id="no-speakers">No Speakers Available.</div>
     </div>
   );
 };
```

当我们只测试 `no-speakers` `div` 时，我们可以有装饰，但我们决定不测试它。在这种情况下，我们希望在页面上有一个标题。我们的测试应该通过。

因此，现在我们准备测试当演讲者存在的情况。

```cs
describe('Speakers Exist', () => {
   it('renders a table when speakers exist', () => {
     // arrange
     const speakers = [{
       id: 'test-speaker-1',
       firstName: 'Test',
       lastName: 'Speaker 1'
     }, {
       id: 'test-speaker-2',
       firstName: 'Test',
       lastName: 'Speaker 2'
     }];

     // act
     const component = setup({speakers});

     // assert        
     expect(component.find('.speakers')
       .children()).to.have.lengthOf(2);
     expect(component.find('.speakers')
       .childAt(0).type().name).to.equal('SpeakerListRow');
   });
 });
```

在这个测试中，我们检查了两件事。我们希望显示正确的演讲者行数，并且希望它们由新的 `SpeakerListRow` 组件渲染。

```cs
export const SpeakerList = ({speakers}) => {
   let contents = <div>Error!</div>;

   if(speakers.length === 0) {
     contents = <div id="no-speakers">No Speakers Available.</div>;
   } else {
     contents = (
       <table className="table">
         <thead>
           <tr>
             <th>Name</th>
             <th></th>
           </tr>
         </thead>
         <tbody className="speakers">
           { 
             speakers.map(speaker => 
               <SpeakerListRow key={speaker.id} speaker={speaker} />) 
           }
         </tbody>
       </table>
     );
   }

   return (
     <div>
     <h1>Speakers</h1>
     { contents }
     </div>
   );
 };
```

由于我们最新的测试，组件代码已经发生了显著变化。我们不得不添加一些逻辑，并且我们还添加了一个默认错误情况，以防内容在没有分配的情况下通过。

为了使这一部分的代码正确工作，我们还需要一个组件。虽然我们不会在这个书中测试该组件，但该组件内部没有逻辑，留作你的练习来创建。

为了创建该组件，如果应用程序能够运行会更好。目前，我们没有连接 Redux，所以应用程序不会渲染任何内容。让我们回顾一下我们现在使用的 Redux 配置。

在 `index.js` 中，我们需要添加一些项目以让 Redux 工作。你的索引应该看起来像这样：

```cs
 import React from 'react';
 import ReactDOM from 'react-dom';
 import {BrowserRouter} from 'react-router-dom';
 import {Provider} from 'react-redux';
 import registerServiceWorker from './registerServiceWorker';
 import configureStore from './store/configureStore';
 import { getSpeakers } from './actions/speakerActions';
 import 'bootstrap/dist/css/bootstrap.min.css';
 import './index.css';
 import App from './components/App.js';

 const store = configureStore();
 store.dispatch(getSpeakers());

 ReactDOM.render(
   <Provider store={store}>
     <BrowserRouter>
       <App/>
     </BrowserRouter>
   </Provider>,
   document.getElementById('root')
 );

 registerServiceWorker();
```

我们添加的两个部分是 Redux 存储，包括一个初始调用以分发加载演讲者的动作，以及添加 Redux 提供者的标记。

在定义其他路由的地方，你需要为演讲者部分添加路由。我们将路由放在 `App.js` 中。

```cs
<Route exact path='/speakers/:id' component={SpeakerDetailPage}/>
<Route exact path='/speakers' component={SpeakersPage}/>
```

最后，我们必须将我们的组件转换为 Redux 组件。将以下行添加到演讲者页面组件文件底部。

```cs
import { connect } from 'react-redux';

function mapStateToProps(state) {
   return {
     speakers: state.speakers
   };
 }

 function mapDispatchToProps(dispatch) {
   return bindActionCreators(speakerActions, dispatch);
 }

 export default connect(mapStateToProps, mapDispatchToProps)(SpeakersPage);
```

从代码示例的底部开始，`connect` 函数由 Redux 提供，它将所有 Redux 功能连接到我们的组件中。传入的两个函数 `mapStateToProps` 和 `mapDispatchToProps` 是作为填充状态和为我们的组件提供执行动作的方式传入的。

在 `mapDispatchToProps` 中，我们调用 `bindActionCreators`；这是另一个 Redux 提供的函数，将给我们一个包含所有动作的对象。通过直接从 `mapDispatchToProps` 返回该对象，我们将动作直接添加到 props 中。我们也可以创建一个包含动作属性的对象，然后将 `bindActionCreators` 的结果分配给该属性。

在应用程序内部任何引用 `SpeakersPage` 的地方，现在都可以改为仅 `SpeakersPage`，这将获取我们新的默认导出。不要在测试中做这个更改。在测试中，我们仍然想要命名导入。

完成这些操作后，我们应该能够运行应用程序并导航到演讲者路由。如果你还没有添加演讲者路由的链接，现在是一个好时机，这样你就不必每次都直接在 URL 中输入路由了。

当你到达演讲者路由时，你应该看到没有演讲者，并且我们收到了我们的消息。我们需要某种方式来填充演讲者，这样我们就可以测试列表。我们将在下一节中介绍填充演讲者的方法。现在，在模拟 API 中修改构造函数以包含几个演讲者。以这种方式修改服务将导致一些测试失败，所以在你视觉上验证了一切看起来都很好之后，务必删除或至少注释掉你添加的代码。

# 演讲者详情

现在我们有了演讲者列表，能够查看更多关于特定演讲者的信息会很好。让我们看看涉及检索和查看演讲者详细信息的测试。

# 添加到模拟 API 服务

在模拟 API 中，我们需要添加一个调用以获取特定演讲者的详细信息。我们可以假设演讲者有一个 ID 字段，我们可以用它来收集这些信息。像往常一样，让我们从简单的存在性检查开始我们的测试。我们需要在`After Initialization`描述中添加一个新的 describe 来通过 ID 获取演讲者。

```cs
describe('Get Speaker By Id', () => {
   it('exists', () => {
     // assert
     expect(service.getById).to.exist;
   });
 });
```

要使这个测试通过，我们需要向模拟 API 添加一个方法。

```cs
getById() {
 }
```

现在，我们可以编写一个测试来验证当找不到匹配的演讲者时我们期望的功能。在这种情况下，我们想要的功能是在我们到达用户界面时显示`SPEAKER_NOT_FOUND`消息。在模拟 API 级别，我们可以假设服务器会发送一个 404。我们可以从模拟 API 中响应一个包含`SPEAKER_NOT_FOUND`类型的错误。这类似于使用操作的方式。

让我们为演讲者未找到的场景创建另一个 describe。

```cs
describe('Speaker Does Not Exist', () => {
   it('SPEAKER_NOT_FOUND error is generated', () => {
     // act
     const promise = service.getById('fake-speaker');

     // assert
     return promise.catch((error) => {
       expect(error.type).to.equal(errorTypes.SPEAKER_NOT_FOUND);
     });
   });
 });
```

你可能已经注意到我们悄悄地加入了`errorTypes`。`errorTypes`有自己的文件夹，但构建方式与`actionTypes`完全相同。

要使这个测试通过，我们必须向我们的模拟 API 添加一个拒绝的承诺。

```cs
getById(id) {
   return new Promise((resolve, reject) => {
     setTimeout(() => {
         reject({ type: errorTypes.SPEAKER_NOT_FOUND });
     }, 0);
   });
 }
```

我们没有测试来强制该方法返回一个积极的结果，所以现在我们可以每次都拒绝。

这就带我们到了下一个测试。如果找到演讲者会发生什么？理想情况下，演讲者和所有演讲者详情都应该返回给调用者。现在让我们编写这个测试。

```cs
describe('Speaker Exists', () => {
   it('returns the speaker', () => {
     // arrange
     const speaker = { id: 'test-speaker' };
     service.create(speaker);

     // act
     let promise = service.getById('test-speaker');

     // assert
     return promise.then((speaker) => {
       expect(speaker).to.not.be.null;
       expect(speaker.id).to.equal('test-speaker');
     });
   });
 });
```

要通过这个测试，我们将在生产代码中添加一些逻辑。

```cs
getById(id) {
   return new Promise((resolve, reject) => {
     setTimeout(() => {
       let speaker = this._speakers.find(x => x.id === id);
       if(speaker) {
         resolve(Object.assign({}, speaker));
       } else {
         reject({ type: errorTypes.SPEAKER_NOT_FOUND });
       }
     }, 0);
   });
 }
```

要使测试通过，我们首先需要检查演讲者是否存在。如果演讲者存在，我们返回该演讲者。如果演讲者不存在，我们拒绝承诺并提供我们的错误结果。

# 获取演讲者操作

我们现在有一个模拟 API 可以调用，它的行为方式符合我们的预期。接下来在我们的列表中是创建将处理模拟 API 结果的操作。对于获取演讲者的过程，我们需要两个操作。其中一个操作将通知应用程序成功找到并提供了找到的演讲者给 reducers。另一个操作将通知应用程序请求的演讲者找不到。

让我们编写一个测试来确认其存在。这个测试应该在演讲者动作测试的同步测试部分中。我们还将为获取演讲者成功动作创建一个新的 describe。

```cs
describe('Find Speaker Success', () => {
   it('exists', () => {
     expect(speakerActions.getSpeakerSuccess).to.exist;
   });
 });
```

要使这个测试通过，我们只需创建动作函数。

```cs
export function getSpeakerSuccess() {
 }
```

现在我们需要验证动作的返回值。就像我们的获取所有演讲者的成功动作一样，获取演讲者成功动作将接收找到的演讲者并返回一个包含类型和演讲者数据的对象。现在让我们为这个动作编写测试。

```cs
it('is created with correct data', () => {
   // arrange
   const speaker = {
     id: 'test-speaker',
     firstName: 'Test',
     lastName: 'Speaker'
   };

   // act
   const result = speakerActions.getSpeakerSuccess(speaker);

   // assert
   expect(result.type).to.equal(types.GET_SPEAKER_SUCCESS);
   expect(result.speaker).to.deep.equal(speaker);
 });
```

这个测试相当直接，所以让我们看看生产代码来通过它。

```cs
export function getSpeakerSuccess(speaker) {
   return { type: types.GET_SPEAKER_SUCCESS, speaker: speaker };
 }
```

再次，这段代码很简单。接下来，让我们处理失败动作。我们还需要为这个测试创建一个新的 describe。

```cs
describe('Get Speaker Failure', () => {
   it('exists', () => {
     expect(speakerActions.getSpeakerFailure).to.exist;
   });
 });
```

这里没有新的内容，你现在应该开始对流程有感觉了。让我们继续并使这个测试通过。

```cs
export function getSpeakerFailure() {
 }
```

在检索演讲者失败时，我们应该得到的是`SPEAKER_NOT_FOUND`错误类型。在我们的下一个测试中，我们将收到这个错误并从中创建动作类型。

```cs
it('is created with correct data', () => {
   // arrange
   const error = {
     type: errorTypes.SPEAKER_NOT_FOUND
   };

   // act
   const result = speakerActions.getSpeakerFailure(error);

   // assert
   expect(result.type).to.equal(types.GET_SPEAKER_FAILURE);
   expect(result.error).to.deep.equal(error);
 });
```

使这个测试通过的过程与其它同步动作的实现非常相似。

```cs
export function getSpeakerFailure(error) {
   return { type: types.GET_SPEAKER_FAILURE, error: error }
 }
```

看着代码，有一个重要的区别。这段代码没有演讲者数据。原因是这个动作需要由不同的 reducer，即错误 reducer 来处理。我们将很快创建错误 reducer 和错误组件。但首先，我们需要创建一个异步动作，该动作将调用模拟 API。

在测试获取演讲者的异步动作时，我们应该从失败情况开始。在这种情况下，失败情况是`GET_SPEAKER_FAILURE`。这里有一个测试来确保触发了正确的次要动作。

```cs
it('creates GET_SPEAKER_FAILURE when speaker is not found', () => {
   // arrange
   const speakerId= 'not-found-speaker';
   const store = mockStore({
     speaker: {}
   });

   // act
   return (
     store.dispatch(speakerActions.getSpeaker(speakerId)).then(() => {
       const actions = store.getActions();

       // assert
       console.log(actions);
       expect(actions[0].type).to.equal(types.GET_SPEAKER_FAILURE);
     })
   );
 });
```

使这个测试通过的代码与我们获取所有演讲者的代码相似。

```cs
export function getSpeaker(speakerId) {
   return function(dispatch) {
     return new MockSpeakerService().getById(speakerId).catch(err => {
       dispatch(getSpeakerFailure(err));
     });
   };
 }
```

在这里，我们已经调用了模拟 API，并期望它拒绝承诺，从而导致`getSpeakerFailure`动作的派发。

我们接下来的测试是成功检索一个特定的演讲者。不过，我们确实有一个问题。你可能已经注意到，我们为每个异步动作创建一个新的`MockSpeakerService`。这是有问题的，因为它阻止我们预先在我们的模拟 API 中填充测试值。在应用程序的开发后期，后端将准备就绪，我们希望将前端代码指向一个真实后端。只要我们直接引用并创建模拟 API 服务，我们就无法做到这一点。

我们面临的问题有很多解决方案。我们将探索创建一个工厂来决定为我们提供哪个后端。工厂还将允许我们将模拟 API 作为单例处理。将服务作为单例处理将允许我们在测试设置中预先填充服务。

在服务文件夹中，让我们为创建工厂类和功能创建一组新的测试。

```cs
import { expect } from 'chai';
import { ServiceFactory as factory } from './serviceFactory';

describe('Service Factory', () => {
  it('exits', () => {
    expect(factory).to.exist;
  });
});
```

我们通过定义一个类就能使这个测试通过。

```cs
export class ServiceFactory {
}
```

现在我们需要一个创建演讲者服务的方法。在工厂测试中添加一个新的 describe。

```cs
describe('Create Speaker Service', () => {
   it('exists', () => {
     expect(factory.createSpeakerService).to.exist;
   });
 });
```

注意我们使用工厂的方式，我们并没有初始化它。我们希望工厂成为一个具有静态方法的类。拥有静态函数将赋予我们想要的单例能力。

```cs
static createSpeakerService() {
 }
```

接下来，我们想要确保`createSpeakerService`工厂方法能为我们提供一个模拟 API 的实例。

```cs
it('returns a speaker service', () => {
   // act
   let result = factory.createSpeakerService();

   // assert
   expect(result).to.be.an.instanceof(MockSpeakerService);
 });
```

使这个测试通过很简单，只需从工厂方法返回一个新的模拟演讲者服务。

```cs
static createSpeakerService() {
   return new MockSpeakerService();
 }
```

然而，这并不是一个单例。因此，我们在这里还有一些更多的工作要做。在我们替换应用中所有服务调用为工厂调用之前，让我们在工厂中再写一个测试。为了验证某个东西是单例，我们必须确保它在整个应用中都是相同的。我们可以通过在连续调用上进行引用比较来实现这一点。另一个选项是创建演讲者服务，向其中添加一个演讲者，创建一个新的演讲者服务，并尝试从第二个服务中拉取演讲者。如果我们正确地做了事情，第二个选项是最彻底的。我们将在这里执行第一个选项，但自己尝试第二个选项将是一个很好的练习。

```cs
it('returns the same speaker service', () => {
   // act
   let service1 = factory.createSpeakerService();
   let service2 = factory.createSpeakerService();

   // assert
   expect(service1).to.equal(service2);
 });
```

为了通过测试，我们必须确保每次都返回相同的演讲者服务实例。

```cs
export default class ServiceFactory {
   constructor() {
     this._speakerService = null;
   }

   static createSpeakerService() {
     return this._speakerService = this._speakerService || 
                                   new MockSpeakerService();
   }
 }
```

现在，工厂将返回当前值，如果当前值为`null`，则创建一个新的演讲者服务。

下一步是前往每个直接实例化模拟演讲者服务的地方，并用工厂调用替换它。我们将把这个作为你的练习，但要知道，向前推进时，我们将假设它已经被完成。

现在我们已经替换了工厂，并且它正在生成单例，我们可以编写下一个动作测试。我们想要测试成功检索一个演讲者。

```cs
it('creates GET_SPEAKER_SUCCESS when speaker is found', () => {
   // arrange
   const speaker = {
     id: 'test-speaker',
     firstName: 'Test',
     lastName: 'Speaker'
   };
   const store = mockStore({ speaker: {} });
   const expectedActions = [
     speakerActions.getSpeakerSuccess([speaker.id])
   ];
   let service = factory.createSpeakerService();
   service.create(speaker);

   // act
   return store.dispatch(
     speakerActions.getSpeaker('test-speaker')).then(() => {
       const actions = store.getActions();

       // assert
       expect(actions[0].type).to.equal(types.GET_SPEAKER_SUCCESS);
       expect(actions[0].speaker.id).to.equal('test-speaker');
       expect(actions[0].speaker.firstName).to.equal('Test');
       expect(actions[0].speaker.lastName).to.equal('Speaker');
     });
 });
```

这个测试中有很多事情在进行；让我们一步步来看。首先在 arrange 阶段，我们创建一个演讲者对象，将其放入服务中，并用于断言。接下来，仍然在 arrange 阶段，我们创建并配置模拟存储。最后，在 arrange 阶段，我们创建演讲者服务，并使用该服务创建我们的测试演讲者。

接着，在 act 中，我们派发一个调用以获取测试演讲者。记住，这个调用是异步的。因此，我们必须订阅 then。

当承诺解决时，我们将动作存储在一个变量中，并断言第一个动作具有正确的类型和有效载荷。

现在为了让这个测试通过，我们需要对服务上的`getById`方法进行一些修改。

```cs
export function getSpeaker(speakerId) {
   return function(dispatch) {
     return factory.createSpeakerService().getById(speakerId).then(
       speaker => {
         dispatch(getSpeakerSuccess(speaker));
       }).catch(err => {
         dispatch(getSpeakerFailure(err));
       });
   };
 }
```

我们真正做的只是添加了一个 then 来处理承诺的解析。现在，从所有当前目的来看，我们有一个工作的演讲者服务。让我们继续创建处理获取演讲者动作的 reducers。

# 获取演讲者 reducer

要处理与获取演讲者相关的动作，我们必须创建两个还原器。第一个还原器与为我们创建的获取演讲者动作的还原器非常相似。第二个将需要稍微不同，用于处理错误情况。

让我们从两个中最简单的一个开始，创建演讲者还原器。

```cs
describe('Speaker Reducer', () => {
     it('exists', () => {
       expect(speakerReducer).to.exist;
     });
 });
```

我们的典型存在测试很容易通过。

```cs
export function speakerReducer() {
 }
```

下一个测试确保还原器正确更新状态，并将结束这个还原器所需的测试。

```cs
it('gets a speaker', () => {
   // arrange
   const initialState = { id: '', firstName: '', lastName: '' };
   const speaker = { id: 'test-speaker', firstName: 'Test', lastName: 'Speaker'};
   const action = actions.getSpeakerSuccess(speaker);

   // act
   const newState = speakerReducer(initialState, action);

   // assert
   expect(newState).to.deep.equal(speaker);
 });
```

这个测试的变化是还原器的输入和状态的输出。让我们通过模仿演讲者还原器来使这个测试通过。

```cs
export function speakerReducer(state = { 
   id: '', 
   firstName: '', 
   lastName: ''
 }, action) {
   switch(action.type) {
     case types.GET_SPEAKER_SUCCESS:
       return action.speaker;
     default:
       return state;
   }
 }
```

与演讲者还原器类似，这个还原器只是检查动作类型是否为`GET_SPEAKER_SUCCESS`，如果找到，则返回动作附加的演讲者作为新状态。否则，我们只返回我们收到的状态对象。

接下来，我们需要一个错误还原器。

```cs
describe('Error Reducer', () => {
   it('exists', () => {
     expect(errorReducer).to.exist;
   });
 });
```

通过这个测试就像通过所有其他存在测试一样简单。

```cs
import * as types from './actionTypes';
import * as errors from './errorTypes';

export function errorReducer() {
}
```

错误还原器将具有一些有趣的功能。在接收到错误的情况下，我们希望错误堆叠起来，这样我们就不会替换状态。相反，我们将克隆并添加到状态中。然而，当我们接收到不是错误的动作时，我们希望清除错误并允许正常程序执行继续。我们还想忽略重复的错误。首先，我们将处理我们已知的错误。

```cs
it('returns error state', () => {
   // arrange
   const initialState = [];
   const error = { type: errorTypes.SPEAKER_NOT_FOUND };
   const action = actions.getSpeakerFailure(error);

   // act
   const newState = errorReducer(initialState, action);

   // assert
   expect(newState).to.deep.equal([error]);
 });
```

我们的测试与之前的还原器测试略有不同。主要区别在于我们正在将预期的值包裹在一个数组中。我们这样做是为了满足可能堆叠多个错误并显示给用户的需求。

要使这个测试通过，我们遵循熟悉的还原器模式，我们已经在使用。

```cs
export function errorReducer(state = [], action) {
   switch(action.type) {
     case types.GET_SPEAKER_FAILURE:
       return [...state, action.error];
     default:
       return state;
   }
 }
```

与之前所述的原因相同，注意我们如何使用其余的参数语法将现有状态展开到新数组中，从而有效地克隆状态。

我们还有两个针对错误还原器的测试；第一个是确保不添加重复的错误。第二个测试将在调用非错误动作时清除错误。

```cs
it('ignores duplicate errors', () => {
   // arrange
   const error = { type: errorTypes.SPEAKER_NOT_FOUND };
   const initialState = [error];
   const action = actions.getSpeakerFailure(error);

   // act
   const newState = errorReducer(initialState, action);

   // assert
   expect(newState).to.deep.equal([error]);
 });
```

在测试中，为了设置具有预填充状态的条件，我们只需修改`initialState`参数。

```cs
export function errorReducer(state = [], action) {
   switch(action.type) {
     case types.GET_SPEAKER_FAILURE:
       let newState = [...state];

       if(newState.every(x => x.type !== action.error.type)) {
         newState.push(action.error);
       }

       return newState;
     default:
       return state;
   }
 }
```

要使这个测试通过，我们只需确保错误类型尚未存在于状态数组中。有许多方法可以做到这一点；我们选择使用`every`函数作为一个检查，以确保现有的错误中没有匹配项。这种方法可能不是非常高效，但由于错误数量最多只有几个，因此不应该成为性能问题。

下一个测试是在接收到非错误时清除错误状态。

```cs
it('clears when a non-error action is received', () => {
   // arrange
   const error = { type: errorTypes.SPEAKER_NOT_FOUND };
   const initialState = [error];
   const action = { type: 'ANY_NON_ERROR' };

   // act
   const newState = errorReducer(initialState, action);

   // assert
   expect(newState).to.deep.equal([]);
 });
```

使这个测试通过极其简单。我们只需替换默认功能，即返回现有状态。

```cs
default:
   return [];
```

# 说话者详情组件

现在我们已经准备好创建我们的`SpeakerDetailPage`。这个组件没有太多内容。它将需要成为一个容器组件，以便它可以使用获取演讲者动作。因为它是一个容器组件，所以我们将不会在这个组件中直接放置任何标记。对我们来说，好消息是这意味着我们的测试将会简短且简单。

为了启动测试，创建一个存在性测试。

```cs
describe('Speaker Detail Page', () => {
   it('exists', () => {
     expect(SpeakerDetailPage).to.exist;
   });
 });
```

创建一个`SpeakerDetailPage`文件，并向其中添加一个组件。

```cs
export class SpeakerDetailPage extends Component {
   render() {
     return (<div></div>);
   }
 }
```

我们接下来想要测试的，在没有直接指定设计的情况下唯一可以测试的另一件事是，模型被接收并且以某种方式显示在屏幕上。现在我们只需要测试模型的一个属性。我们将编写一个测试来显示演讲者的名字被显示出来。

```cs
describe('Render', () => {
   it('renders', () => {
     // arrange
     const props = {
       match: { params: { id: 'test-speaker' } },
       actions: { getSpeaker: (id) => { return Promise.resolve(); } },
       speaker: { firstName: 'Test' } 
     };

     // act
     const component = mount(<SpeakerDetailPage { ...props } />);

     // assert
     expect(component.find('first-name').text()).to.contain('Test');
   });
 });
```

如果你注意到了，你可能想知道为什么获取演讲者动作只是返回一个空的已解析的承诺。我们并没有绑定到 Redux，所以启动动作不会触发 reducer，这不会更新 store，也不会触发组件状态的刷新。尽管如此，我们仍然想在测试设置中完成组件的合约，这个组件将调用那个函数。我们可以省略这一行，但一旦我们连接了 Redux，我们就会将其添加回来。

为了使测试通过，我们需要在`SpeakerDetailPage`组件中做一些简单的修改，并创建一个全新的组件。以下是该组件的修改，但创建下一个组件将是你的练习。它仅用于显示，我们正在测试它是否在这里被填充，所以你只需要编写一个表现组件。

```cs
export class SpeakerDetailPage extends Component {
   constructor(state, context) {
     super(state, context);

     this.state = {
       speaker: Object.assign({}, this.props.speaker)
     };
   }

   render() {
     return (
       <SpeakerDetail speaker={this.state.speaker} />
     );
   }
 }
```

之前的代码将使测试通过，但现在我们必须将组件连接到 Redux。我们将添加对`getSpeaker`动作的调用，绑定到`componentWillReceiveProps`生命周期事件，并使用 connect 函数映射属性和分发。以下是最终的`SpeakerDetailPage`组件。

```cs
export class SpeakerDetailPage extends Component {
   constructor(state, context) {
     super(state, context);

     this.state = {
       speaker: Object.assign({}, this.props.speaker)
     };

     this.props.actions.getSpeaker(this.props.match.params.id)
   }

   componentWillReceiveProps(nextProps) {
     if(this.props.speaker.id !== nextProps.speaker.id) {
       this.setState({ speaker: Object.assign({}, nextProps.speaker) });
     }
   }

   render() {
     return (
       <SpeakerDetail speaker={this.state.speaker} />
     );
   }
 }

 function mapStateToProps(state, ownProps) {
   let speaker = { id: '', firstName: '', lastName: '' }

   return {
     speaker: state.speaker || speaker
   };
 }

 function mapDispatchToProps(dispatch) {
   return {
     actions: bindActionCreators(speakerActions, dispatch)
   }
 }

 export default  connect(
   mapStateToProps,
   mapDispatchToProps
 )(SpeakerDetailPage);
```

现在一切测试都通过了，我们还需要完成最后一件事，才能正确地进一步开发。之前我们用对工厂的调用替换了模拟 API。我们这样做是为了让测试能够影响动作中模拟 API 的状态。同样的修改使得我们可以为我们的应用程序配置一个起点。在`index.js`文件中，在配置完 store 之后添加以下代码；现在，当你运行应用程序时，你将会有可用的演讲者来测试 UI。

```cs
const speakers = [{
   id: 'clayton-hunt',
   firstName: 'Clayton',
   lastName: 'Hunt'
 }, {
   id: 'john-callaway',
   firstName: 'John',
   lastName: 'Callaway'
 }];

 let service = factory.createSpeakerService();
 speakers.forEach((speaker) => {
   service.create(speaker);
 });
```

# 摘要

目前为止，我们已经完成了对 React 应用的单元测试。我们还没有一个测试某种输入的例子。尝试测试并实现一个`CreateSpeakerPage`。从 React 的角度来看，你需要做些什么？Redux 会要求你做些什么？在本章中，我们像对待组件一样对待了 React 组件。对于仅用于显示的组件，这正是这些组件所做的工作，这种方法可能是更好的。然而，对于具有一些实际功能的组件，你可能希望在将其附加到 React 之前，先尝试以普通的 JavaScript 类测试其功能。我们也在本章中为你留下了相当多的工作要做。如果你在填写空白以完成代码时迷失方向或需要提示，不要害羞地去查看与本章相关的源代码。
