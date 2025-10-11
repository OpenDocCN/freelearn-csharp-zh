# 第十章：探索集成

在本章中，我们将探讨集成测试 Speaker Meet 应用程序。我们将测试并配置 React 前端应用程序以击中真实的后端 API，并测试.NET 应用程序以确保它从控制器到数据库都能正常工作。

在本章中，我们将涵盖：

+   实现一个真实的 API 服务

+   移除模拟 API 调用

+   端到端集成

+   集成测试

# 实现一个真实的 API 服务

现在是真正从服务器接收数据的时候了。我们当前的数据模型仍然不是 100%正确，但基础工作已经完成。当我们从服务器接收到正确的数据结构时，我们需要相应地更新我们的视图。我们将把这个部分留给你作为练习。

在本节中，我们将查看从我们创建的工厂中拉出模拟 API，并用真实 API 替换它。在我们的现有测试中，我们将使用 Sinon 来覆盖我们的 Ajax 组件的默认功能，使用我们的模拟 API 的功能。

最后，我们需要创建一个应用程序配置对象来管理 API 的基本路径，以确定开发和生产中的正确路径。

# 用真实 API 服务替换模拟 API

为了尽可能保持简单，我们将使用 fetch API 从服务器获取数据。我们将首先中断所有目前使用模拟 API 的测试。这是因为我们将创建一个实现与模拟 API 相同接口的存根类，但它不会做任何事情：

`src/services/fetchSpeakerService.js`

```cs
import * as errorTypes from '../reducers/errorTypes';

 export default class FetchSpeakerService {
   constructor() { }

   create(speaker) {
     return;
   }

   getAll() {
     return;
   }

   getById(id) {
     return;
   }
 }
```

现在，用基于 fetch 的服务创建来替换工厂创建的模拟服务：

```cs
import FetchSpeakerService from './fetchSpeakerService';

 export default class ServiceFactory {
   constructor() {
     this._speakerService = null;
   }

   static createSpeakerService() {
     return this._speakerService = 
       this._speakerService || new FetchSpeakerService();
   }
 }
```

幸运的是，只有四个测试因为这次更改而失败。查看失败的测试，其中三个失败是因为我们没有返回一个承诺。然而，有一个测试失败是因为我们不再返回模拟 API。我们将忽略由缺少承诺引起的失败测试，暂时排除它们。然后，我们将专注于检查特定实例的测试。

失败的测试是在服务工厂测试中。我们实际上不希望服务工厂返回一个`MockSpeakerService`。我们希望它返回一个`FetchSpeakerService`。更准确地说，我们希望返回任何`SpeakerService`的实现。让我们创建一个基类，它将表现得像 C#中的接口或抽象类：

`/src/services/speakerService.js`

```cs
export default class SpeakerService {
   create(speaker) {
     throw new Error("Not Implemented!")
   }

   getAll() {
     throw new Error("Not Implemented!")
   }

   getById(id) {
     throw new Error("Not Implemented!")
   }
 }
```

现在我们有一个抽象基类，我们需要在我们的现有服务类中继承这个基类：

```cs
import SpeakerService from './speakerService';

 export default class MockSpeakerService extends SpeakerService {
   constructor() {
     super();

     this._speakers = [];
  }
 …
```

```cs
import SpeakerService from './speakerService';

 export default class FetchSpeakerService extends SpeakerService {
   constructor() {
     super();
   }
 …
```

然后我们需要修改工厂测试，以期望基类的一个实例而不是派生类的一个实例：

```cs
it('returns a speaker service', () => {
   // act
   let result = factory.createSpeakerService();

   // assert
   expect(result).to.be.an.instanceOf(SpeakerService);
 });
```

# 使用 Sinon 模拟 Ajax 响应

现在，是时候解决我们忽略的三个测试了。它们期望从我们的服务中获得实际响应。目前，我们的服务是完全空的。记住，那些测试是为了编写单元测试而编写的，我们需要保护它们免受真实端点随时间变化响应的影响。为此，我们最终将引入 Sinon。

我们将使用 Sinon 从我们的模拟 API 返回结果，而不是使用真实 API。这将允许我们继续使用我们已经投入模拟 API 的工作。

在我们有现有的测试覆盖后，我们将通过使用 Sinon 模拟后端服务器来引入集成测试。以这种方式使用 Sinon 将允许我们测试驱动基于 fetch 的演讲者服务。

# 修复现有测试

首先，我们必须让现有的测试通过。在`speakerActions.spec.js`文件中，找到我们跳过的第一个测试，并移除跳过。这将导致该测试失败，原因如下：

`无法读取未定义的属性 'then'`

回到`beforeEach`方法，在那里我们创建演讲者服务，我们需要为服务方法创建一个新的 Sinon 模拟。查看测试，我们可以看到我们第一次服务调用是获取所有演讲者。所以，让我们从这里开始：

```cs
beforeEach(() => {
   let service = factory.createSpeakerService();
   let mockService = new MockSpeakerService();

   getAll = sinon.stub(service, "getAll");
   getAll.callsFake(mockService.getAll.bind(mockService));

   mockStore = configureMockStore(middleware);
 });
```

看着这段代码，我们所做的是创建一个新的 Sinon 模拟，并将对服务`getAll`方法的调用重定向到`mockService getAll`方法。最后，我们将`mockService`调用绑定到`mockService`以保留对私有变量的访问。

再次运行测试，我们得到一个新的错误：

`尝试包装已包装的 getAll`

这个错误告诉我们，我们已经为我们要模拟的方法创建了一个模拟。起初，这个错误可能没有意义。但是，如果你看，我们在`beforeEach`中这样做。Sinon 是一个单例，我们在`beforeEach`中运行我们的模拟命令，所以当第二个测试准备运行时，它已经注册了一个`getAll`模拟。我们必须做的是在我们再次注册之前移除该注册。另一种说法是，我们必须在每个测试运行后移除注册。让我们添加一个`afterEach`方法并在那里移除注册：

```cs
afterEach(() => {
   getAll.restore();
 });
```

这修复了我们遇到的第一个失败的测试，现在让我们修复其他两个。过程将大致相同，让我们开始。

从下一个测试中移除跳过。测试失败。我们在这次测试中调用`getSpeaker`动作，如果我们查看演讲者动作，我们可以看到它使用了`getById`服务方法。和之前一样，我们将在`beforeEach`中模拟这个方法。

`getById = sinon.stub(service, "getById");`

`getById.callsFake(mockService.getById.bind(mockService));`

如前所述，我们现在正在获取已经包装的消息：

`尝试包装已包装的 getById`

我们可以通过与修复上一个错误相同的方式修复这个问题，即在`afterEach`函数中移除模拟。

`getById.restore();`

我们现在回到了所有通过测试，只有一个跳过的测试。最后一个测试是相同的过程。以下是当我们完成时完整的`beforeEach`和`afterEach`函数：

```cs
beforeEach(() => {
   let service = factory.createSpeakerService();
   let mockService = new MockSpeakerService();

   getAll = sinon.stub(service, "getAll");
   getAll.callsFake(mockService.getAll.bind(mockService));

   getById = sinon.stub(service, "getById");
   getById.callsFake(mockService.getById.bind(mockService));

   create = sinon.stub(service, "create");
   create.callsFake(mockService.create.bind(mockService));

   mockStore = configureMockStore(middleware);
 });

 afterEach(() => {
   create.restore();
   getAll.restore();
   getById.restore();
 });
```

不要忘记从最后一个测试中移除跳过。当一切完成时，你应该有 42 个通过测试和 0 个跳过测试。

# 模拟服务器

现在我们已经修复了现有的测试，我们准备好开始编写对真实服务`fetchSpeakerService`的测试。让我们从查看我们用于模拟服务的测试开始。测试将大致相同，因为我们试图实现相同的功能模式。

首先，我们将想要创建测试文件`fetchSpeakerService.spec.js`。一旦文件创建完成，我们就可以添加标准的存在性测试：

```cs
describe('Fetch Speaker Service', () => {
   it('exits', () => {
     expect(FetchSpeakerService).to.exist;
   });
 });
```

由于我们之前模拟了 fetch 演讲者服务，这个测试在添加适当的导入后应该直接通过。

在模拟演讲者服务测试之后，下一个测试是一个构造和类型验证测试：

```cs
it('can be constructed', () => {
   // arrange
   let service = new FetchSpeakerService();

   // assert
   expect(service).to.be.an.instanceof(FetchSpeakerService);
 });
```

这个测试也应该立即通过，因为我们模拟了 fetch 服务时，我们将其创建为一个类。继续跟随模拟服务测试的进展，我们有一个`After Initialization`部分，其中包含一个`Create`部分。*Create*部分中唯一的测试是对`Create`方法的`exists`测试。编写这个测试时，它应该通过：

```cs
describe('After Initialization', () => {
   let service = null;

   beforeEach(() => {
     service = new FetchSpeakerService();
   });

   describe('Create', () => {
     it('exists', () => {
       expect(service.create).to.exist;
     });
   });
 });
```

由于我们正在从模拟服务测试中复制流程，我们已经将服务提取到`beforeEach`实例化中。

在下一节中，我们的测试将开始变得有趣，而不会立即通过。在我们继续之前，为了验证测试是否正在执行它们应该执行的操作，注释掉 fetch 服务的一部分是一个好主意，以查看适当的测试是否通过。

接下来是`Get All`部分，仍然在`After Initialization`部分内，我们有一个检查`getAll 方法`的存在性测试：

```cs
describe('Get All', () => {
   it('exists', () => {
     // assert
     expect(service.getAll).to.exist;
   });
 });
```

与迄今为止的其他测试一样，要使这个测试失败，你将不得不在 fetch 服务中注释掉`getAll`方法以查看其失败。紧接着这个测试的是两个更多部分：`No Speakers Exist`和`Speaker Listing`。我们将逐个添加它们，从`No Speakers Exist`开始：

```cs
describe.skip('No Speakers Exist', () => {
   it('returns an empty array', () => {
     // act
     let promise = service.getAll();

     // assert
     return promise.then((result) => {
       expect(result).to.have.lengthOf(0);
     });
   });
 });
```

最后，我们有一个失败的测试。失败的原因是它看起来我们没有返回一个 promise。让我们开始正确实现 fetch 服务，并在测试中使用 Sinon 来模拟后端。在 fetch 服务中，添加以下内容：

```cs
constructor(baseUrl) {
   super();

   this.baseUrl = baseUrl;
 }

 getAll() {
   return fetch(`${this.baseUrl}/speakers`).then(r => {
     return r.json();
   });
 }
```

这是一个非常基本的 fetch 调用。我们使用 HTTP 动词`GET`，所以没有必要在 fetch 上调用方法；默认情况下，它将使用`GET`。

在我们的测试中，我们现在得到了一个有意义的结果。`fetch is not defined`。这个结果是因为 fetch 在我们的测试设置中还不存在。我们需要导入一个新的 NPM 包来处理测试中的 fetch 调用。我们想要导入的包是`fetch-ponyfill`。

```cs
>npm install fetch-ponyfill
```

安装了`ponyfill`库之后，我们必须修改我们的测试设置文件`scripts/test.js`：

```cs
import { JSDOM } from'jsdom';
import Enzyme from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';
import fetchPonyfill from 'fetch-ponyfill';
const { fetch } = fetchPonyfill(); 
const jsdom = new JSDOM('<!doctype html><html><body></body></html>');
const { window } = jsdom;
window.fetch = window.fetch || fetch; 
…

global.window = window;
global.document = window.document;
global.fetch = window.fetch;
```

在这些修改之后，我们必须重新启动我们的测试以使更改生效。我们现在得到了一个测试失败，告诉我们只支持绝对 URL。我们得到这个消息是因为当我们实例化我们的 fetch 服务时，我们没有传递一个 baseURL。对于测试来说，URL 是什么并不重要，所以让我们就使用 localhost：

```cs
beforeEach(() => {
   service = new FetchSpeakerService('http://localhost');
 });
```

在进行这个更改后，我们将错误向前移动，现在我们得到了一个 fetch 错误，表明 localhost 拒绝了一个连接。我们现在准备好用 Sinon 替换后端。我们将从`beforeEach`和`afterEach`开始：

```cs
let fetch = null;

 beforeEach(() => {
   fetch = sinon.stub(global, 'fetch');
   service = new FetchSpeakerService('http://localhost');
 });

 afterEach(() => {
   fetch.restore();
 });
```

在测试中，我们需要`fetch-ponyfill`包中的某些项目，所以在我们接近文件顶部的时候，让我们添加导入语句。

```cs
import fetchPonyfill from 'fetch-ponyfill';
 const {
   Response,
   Headers
 } = fetchPonyfill();
```

现在在测试中，我们需要配置来自服务器的响应：

```cs
it('returns an empty array', () => {
   // arrange
   fetch.returns(new Promise((resolve, reject) => {
     let response = new Response();
     response.headers = new Headers({
       'Content-Type': 'application/json'
     });
     response.ok = true;
     response.status = 200;
     response.statusText = 'OK';
     response.body = JSON.stringify([]);

     resolve(response);
   }));

   // act
   let promise = service.getAll();

   // assert
   return promise.then((result) => {
     expect(result).to.have.lengthOf(0);
   });
 });
```

这样就完成了“无演讲者存在”的场景。一旦我们有了更好的数据变化想法，我们将重构服务器响应。

我们现在准备好进行演讲者列表场景了。和之前一样，我们首先从模拟服务测试中复制测试。从模拟服务测试中移除 arrange，并复制我们之前的测试中的 arrange。

在添加了无演讲者测试的 arrange 之后，我们得到了一个期望长度为 1 而不是 0 的消息。这是一个简单的修复，为了这个测试的目的，我们可以在响应的主体数组中简单地添加一个空对象。以下是测试通过后应该看起来像的样子：

```cs
describe('Speaker Listing', () => {
     it('returns speakers', () => {
       // arrange
       fetch.returns(new Promise((resolve, reject) => {
         let response = new Response();
         response.headers = new Headers({
           'Content-Type': 'application/json'
         });
         response.ok = true;
         response.status = 200;
         response.statusText = 'OK';
         response.body = JSON.stringify([{}]);

         resolve(response);
       }));

       // act
       let promise = service.getAll();

       // assert
       return promise.then((result) => {
         expect(result).to.have.lengthOf(1);
       });
     });
   });
 });
```

现在我们基本上两次使用了相同的 arrange，是时候重构我们的测试了。唯一真正改变的是主体。让我们提取一个`okResponse`函数来使用：

```cs
function okResponse(body) {
   return new Promise((resolve, reject) => {
     let response = new Response();
     response.headers = new Headers({
       'Content-Type': 'application/json'
     });
     response.ok = true;
     response.status = 200;
     response.statusText = 'OK';
     response.body = JSON.stringify(body);

     resolve(response);
   });
 }
```

我们已经将这个辅助函数放在了`After Initialization` `describe`的顶部。现在在每个测试中，用对这个函数的调用替换 arrange，传递特定于该测试的主体。

现在所有演讲者的获取功能都由测试覆盖了。让我们继续通过 ID 获取特定演讲者的测试。从模拟服务测试中复制`getById`的测试，并应用一个 skip 到 describes。现在，从最外层的 describe 中移除 skip。这应该启用存在测试，它应该通过。

下一个测试是当找不到演讲者时；从那个测试中移除 skip 会导致出现一个消息，表明我们没有返回一个 promise。让我们进入`getById`函数的主体，并使用 fetch 获取一个演讲者：

```cs
getById(id) {
   return fetch(`${this.baseUrl}/speakers/${id}`);
 }
```

给我们的函数添加 fetch 应该已经修复了错误，但实际上并没有。记住，我们正在模拟 fetch 的响应，如果我们不设置响应，那么 fetch 将不会返回任何内容。让我们配置模拟响应。在这种情况下，我们期望服务器返回 404 错误，所以让我们配置这个响应：

```cs
// arrange
 fetch.returns(new Promise((resolve, reject) => {
   let response = new Response();
   response.headers = new Headers({
     'Content-Type': 'application/json'
   });
   response.ok = false;
   response.status = 404;
   response.statusText = 'NOT FOUND';

   resolve(response);
 }));
```

这样我们的测试就通过了，但这并不是正确的理由。让我们在断言中添加一个`then`子句来证明这是一个假阳性：

```cs
// assert
 return promise}).then(() => {
   throw { type: 'Error not returned' };
 }).catch((error) => {
   expect(error.type).to.equal(errorTypes.SPEAKER_NOT_FOUND);
 });
```

现在我们的测试将失败，预期的`'Error not returned'`将等于`'SPEAKER_NOT_FOUND'`。为什么是这样？不应该是一个 404 导致 promise 的拒绝吗？对于 fetch 来说，唯一会导致 promise 拒绝的是网络连接错误。因此，当我们模拟服务器响应时我们没有拒绝。我们需要做的是在服务中检查那个条件，并在那一侧引起 promise 拒绝。最简单的方法是将 fetch 调用包裹在我们的自己的 promise 中。一旦包裹，我们就可以检查适当的条件并拒绝我们的 promise：

```cs
getById(id) {
   return new Promise((resolve, reject) => {
     fetch(`${this.baseUrl}/speakers/${id}`).then((response) => {
       if (!response.ok) {
         reject({
           type: errorTypes.SPEAKER_NOT_FOUND
         });
       }
     });
   });
 }
```

这应该就完成了这个测试。我们现在准备进行最后一个测试。在我们继续之前，让我们快速重构这个测试中的 arrange 部分，以缩短测试并使其对未来的读者更有意义。当我们这样做的时候，我们将重构现有的响应函数以减少重复并强制执行一些默认值：

```cs
function baseResponse() {
   let response = new Response();
   response.headers = new Headers({
     'Content-Type': 'application/json'
   });
   response.ok = true;
   response.status = 200;
   response.statusText = 'OK';

   return response;
 }

 function okResponse(body) {
   return new Promise((resolve, reject) => {
     let response = baseResponse();       
     response.body = JSON.stringify(body);

     resolve(response);
   });
 }

 function notFoundResponse() {
   return new Promise((resolve, reject) => {
     let response = baseResponse();
     response.ok = false;
     response.status = 404;
     response.statusText = 'NOT FOUND';

     resolve(response);
   })
 }
```

在测试中使用`notFoundResponse`函数，就像我们使用`okResponse`函数一样。接下来，我们将进行当前获取服务功能的最后一个测试，移除下一个 describe 中的 skip，我们将开始查看生成的错误并做出必要的更改以使测试通过。

在我们已经做了让模拟响应更容易的工作之后，这个最后的测试相当简单。我们需要获取调用返回一个带有演讲者作为主体的`ok`响应：

```cs
describe('Speaker Exists', () => {
   it('returns the speaker', () => {
     // arrange
     const speaker = {
       id: 'test-speaker'
     };
     fetch.returns(okResponse(speaker));

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

现在，我们遇到了超时错误。这是因为我们的服务实际上没有处理演讲者存在的情况。现在让我们添加这个功能：

```cs
getById(id) {
   return new Promise((resolve, reject) => {
     fetch(`${this.baseUrl}/speakers/${id}`).then((response) => {
       if (response.ok) {
         resolve(response.json());
       } else {
         reject({
           type: errorTypes.SPEAKER_NOT_FOUND
         });
       }
     });
   });
 }
```

现在我们所有的测试都通过了，我们已经验证了系统的所有预期行为。我们还可以做几件事情，一些开发者可能会选择去做。我们将讨论其中的一些，但不会提供示例。

# 应用程序配置

现在所有测试都通过了，但在应用程序可以使用之前，还有一些应用程序配置需要处理。

在服务工厂中，我们必须为运行中的应用程序设置一个用于获取服务的基 URL。这可以通过多种方式完成，具体方式由你决定。最简单但最不灵活的方式是将一个字符串值硬编码为构建服务的基 URL。然而，你也可以让它变得非常复杂，比如有一个动态类，根据应用程序和运行环境来设置值。再次，这个决定留给你。

# 端到端集成测试

本章最后要讨论的主题是端到端集成测试。这些测试涉及实际调用服务器并检查真实响应。

# 优势

那么，测试实际客户端服务器连接的好处是什么？最有价值的优势是，你知道你的应用程序将在部署环境中工作。有时应用程序会被部署但无法工作，因为网络或数据库连接配置不正确，这将对部署造成破坏。

此外，这还将帮助验证系统是否正常工作。在部署后，可以执行一系列烟雾测试，以确保部署成功。

# 损害

端到端测试通常由于以下两个原因之一而被跳过。第一个原因是它们很难编写。你需要进行很多额外的设置才能运行这些测试，包括一个与通常用于单元测试完全不同的测试运行器。如果不是不同的运行器，它们至少需要是单独的测试运行，而不是包含在你的正常单元测试中。

第二个问题是端到端测试很脆弱。系统中的任何更改都会导致这些测试失败。它们不像单元测试那样经常运行，因此，直到它们在生产环境中运行时，才会注意到代码已损坏。

由于这些原因，我们通常不会编写很多端到端测试，如果不是一个都不写的话。

# 你应该进行多少端到端测试？

如果你选择进行端到端测试，你希望尽可能少做。这些是最高级别的测试，应该是你系统中数量最少的测试类型。建议只编写与你的应用程序第三方连接一样多的测试，也就是说，每个你必须与之通信的后端服务器一个测试。此外，使用最简单、最基本的情况，这种情况下预计不会发生变化。

这样就完成了前端集成测试。还有一些事情可以做。我们将把它们留给你作为练习。你可能已经注意到，前端和后端在传递回和 forth 的模型上并不完全一致。作为练习，添加或删除并完善两个系统都使用的模型，以便它们在格式上达成一致。

另一个任务是设置获取服务的基 URL 并在本地运行两个应用程序以验证互操作性。

# 配置 API 项目

现在 React 项目已配置为调用真实 API，是时候将我们的注意力转向 .NET 解决方案了。为了验证一切是否正确连接，你需要编写一系列集成测试，以确保整个系统正常工作。

# 集成测试项目

在现有解决方案中创建一个新的 xUnit 项目，名为 `SpeakerMeet.Api.IntegrationTest`。这将是在其中创建 .NET 集成测试的地方。你可能希望根据你的偏好和/或团队编码标准将它们分离出来，但这可以稍后再说。现在，一个单独的集成测试项目就足够了。

对于我们的目的，我们将测试系统是否从 API 入口到数据库，然后再返回的功能。然而，最好从小测试单个集成点开始，然后逐步扩展。

# 从哪里开始？

你当然可以从创建一个调用 API 端点的测试开始。为了实现这一点，需要向控制器发出一个 HTTP 请求。然后控制器将调用业务层中的服务，该服务随后将调用仓库，最终向数据库发送命令。这感觉有很多移动部件。也许有一个更好的起点。

为了将问题分解成更小、更易于管理的部分，也许从测试应用程序的持久层开始测试是最好的。

# 验证仓库是否调用 DB 上下文

一个好的开始是验证系统是否完全集成；让我们首先测试仓库是否可以访问数据库。在集成测试项目中创建一个名为`RepositoryTests`的文件夹，并创建一个名为`GetAll`的新测试文件。这里将创建仓库`GetAll`方法的集成测试。

你可以创建一个测试来验证仓库可以被创建，如下所示：

```cs
[Fact]
public void ItExists()
{
  var options = new DbContextOptions<SpeakerMeetContext>();
  var context = new SpeakerMeetContext(options);

  var repository = new Repository.Repository<Speaker>(context);
}
```

然而，这并不能通过。如果你运行测试，你将收到以下错误：

`System.InvalidOperationException: No database provider has been configured for this DbContext`。

这很容易解决，只需配置一个合适的提供程序。

# 内存数据库

对 SQL Server 进行测试既耗时又容易出错，可能还相当昂贵。建立数据库连接需要时间，记住，你希望你的测试套件能够快速运行。如果数据库被其他人使用，无论是开发环境中的，还是质量保证工程师等，这也可能是一个问题。你当然不希望对你的生产数据库运行集成测试。此外，对托管在云中的数据库（例如 AWS、Azure 等）进行测试可能会在带宽和处理方面产生成本。

幸运的是，配置一个使用 Entity Framework 的解决方案以使用`InMemory`数据库相当简单。

首先，安装一个用于`InMemory`数据库的`NuGet`包。

`Microsoft.EntityFrameworkCore.InMemory`

现在，修改你之前创建的测试，以便在内存中创建数据库上下文：

```cs
[Fact]
public void ItExists()
{
  var options = new DbContextOptionsBuilder<SpeakerMeetContext>()
      .UseInMemoryDatabase("SpeakerMeetInMemory")
      .Options;

  var context = new SpeakerMeetContext(options);

  var repository = new Repository.Repository<Speaker>(context);
}
```

测试现在应该通过，因为上下文现在是在内存中创建的。

接下来，创建一个测试来验证当调用`GetAll`方法时，会返回一个演讲者实体集合：

```cs
[Fact]
public void GivenSpeakersThenQueryableSpeakersReturned()
{
  using (var context = new SpeakerMeetContext(_options))
  {
    // Arrange
    var repository = new Repository.Repository<Speaker>(context);

    // Act
    var speakers = repository.GetAll();

    // Assert
    Assert.NotNull(speakers);
    Assert.IsAssignableFrom<IQueryable<Speaker>>(speakers);
  }
}
```

现在，将注意力转向仓库中的`Get`方法。创建一个新的测试方法来验证当找不到具有给定 ID 的演讲者时，会返回一个 null 的演讲者实体：

```cs
[Fact]
public void GivenSpeakerNotFoundThenSpeakerNull()
{
  using (var context = new SpeakerMeetContext(_options))
  {
    // Arrange
    var repository = new Repository.Repository<Speaker>(context);

    // Act
    var speaker = repository.Get(-1);

    // Assert
    Assert.Null(speaker);
  }
}
```

这应该立即通过。现在，创建一个测试来验证当存在具有提供的 ID 的演讲者时，会返回一个演讲者实体：

```cs
[Fact]
public void GivenSpeakerFoundThenSpeakerReturned()
{
  using (var context = new SpeakerMeetContext(_options))
  {
    // Arrange
    var repository = new Repository.Repository<Speaker>(context);

    // Act
    var speaker = repository.Get(1);

    // Assert
    Assert.NotNull(speaker);
    Assert.IsAssignableFrom<Speaker>(speaker);
  }
}
```

这个测试现在还不能通过。无论 ID 为 1 的演讲者是否存在于您的开发数据库中，`InMemory`数据库中的演讲者表目前是空的。向`InMemory`数据库添加数据非常简单。

# 向`InMemory`数据库添加演讲者

为了测试仓库在查询数据库时是否会返回特定的`Speaker`实体，您首先必须将`Speaker`添加到数据库中。为了做到这一点，在您的测试文件中添加几行代码：

```cs
using (var context = new SpeakerMeetContext(_options))
{
  context.Speakers.Add(new Speaker { Id = 1, Name = "Test"... });
  context.SaveChanges();
}
```

随意添加尽可能多的演讲者，以及您认为必要的尽可能多的细节。现在，您的测试应该可以通过。可以创建更多测试，并且随着系统的功能复杂度增长，应该继续添加测试。大部分的逻辑应该在单元测试中已经测试过了，但验证整个系统是否正常工作同样重要。

# 验证服务是否通过仓库调用数据库

接下来，转向业务层，您应该验证每个服务是否可以通过仓库从`InMemory`数据库中检索数据。

首先，在集成测试项目中创建一个名为`ServiceTests`的新文件夹。在该文件夹中，创建一个名为`SpeakerServiceTests`的文件夹。这个文件夹是创建针对`SpeakerService`的特定测试的地方。

创建一个名为`GetAll`的新测试文件。添加一个测试方法来验证服务是否可以创建：

```cs
[Fact]
public void ItExists()
{
  var options = new DbContextOptionsBuilder<SpeakerMeetContext>()
      .UseInMemoryDatabase("SpeakerMeetInMemory")
      .Options;

  var context = new SpeakerMeetContext(options);

  var repository = new Repository<Speaker>(context);
  var gravatarService = new GravatarService();

  var speakerService = new SpeakerService(repository, gravatarService);
}
```

# ContextFixture

这里有很多设置代码，以及从前面的测试中复制的大量代码。幸运的是，您可以使用所谓的*测试固定装置*。

测试固定装置只是一些用于配置待测系统的代码。在我们的情况下，创建一个*上下文固定装置*来设置`InMemory`数据库。

创建一个名为`ContextFixture`的新类，其中将发生所有`InMemory`数据库的创建：

```cs
public class ContextFixture : IDisposable
{
  public SpeakerMeetContext Context { get; }

  public ContextFixture()
  {
    var options = new DbContextOptionsBuilder<SpeakerMeetContext>()
        .UseInMemoryDatabase("SpeakerMeetContext")
        .Options;

    Context = new SpeakerMeetContext(options);

    if (!Context.Speakers.Any())
    {
      Context.Speakers.Add(new Speaker {Id = 1, Name = "Test"...});
      Context.SaveChanges();
    }
  }

  public void Dispose()
  {
    Context.Dispose();
  }
}
```

现在，修改测试类以使用新的`ContextFixture`类：

```cs
[Collection("Service")]
[Trait("Category", "Integration")]
public class GetAll : IClassFixture<ContextFixture>
{
  private readonly IRepository<Speaker> _repository;
  private readonly IGravatarService _gravatarService;

  public GetAll(ContextFixture fixture)
  {
    _repository = new Repository<Speaker>(fixture.Context);
    _gravatarService = new GravatarService();
  }

  [Fact]
  public void ItExists()
  {
    var speakerService = new SpeakerService(_repository, _gravatarService);
  }
}
```

这要干净得多。现在，创建一个新的测试来确保当调用`SpeakerService`的`GetAll`方法时，会返回一个`SpeakerSummary`对象的集合：

```cs
[Fact]
public void ItReturnsCollectionOfSpeakerSummary()
{
  // Arrange
  var speakerService = new SpeakerService(_repository, _gravatarService);

  // Act
  var speakers = speakerService.GetAll();

  // Assert
  Assert.NotNull(speakers);
  Assert.IsAssignableFrom<IEnumerable<SpeakerSummary>>(speakers);
}
```

接下来，为`SpeakerService`的`Get`方法创建一个新的测试类。第一个测试应该验证当提供的 ID 不存在时，会抛出异常：

```cs
[Fact]
public void GivenSpeakerNotFoundThenSpeakerNotFoundException()
{
  // Arrange
  var speakerService = new SpeakerService(_repository, _gravatarService);

  // Act
  var exception = Record.Exception(() => speakerService.Get(-1));

  // Assert
  Assert.IsAssignableFrom<SpeakerNotFoundException>(exception);
}
```

您可以重用之前创建的`ContextFixture`：

```cs
[Fact]
public void GivenSpeakerFoundThenSpeakerDetailReturned()
{
  // Arrange
  var speakerService = new SpeakerService(_repository, _gravatarService);

  // Act
  var speaker = speakerService.Get(1);

  // Assert
  Assert.NotNull(speaker);
  Assert.IsAssignableFrom<SpeakerDetail>(speaker);
}
```

# 验证对服务的 API 调用

现在，将您的注意力转向 Web API 控制器。如前一章所述，您可以简单地创建一个控制器的新实例并调用测试中的方法。然而，那样并不能测试整个系统。

用 HTTP 请求调用方法会更好。部署到 Web 服务器会非常耗时。

# TestServer

ASP.NET Core 有配置测试目的主机的能力。从`NuGet`安装`TestServer`：

`Microsoft.AspNetCore.TestHost`

这里有一些设置工作。首先，你将添加一个`TestServer`的实例。创建一个新的`WebHostBuilder`并使用 Web API 项目的现有`Startup`类。这将连接之前设置的依赖注入容器。现在，配置服务以设置一个新的`InMemory`数据库。

看看这里的测试以了解所需的设置：

```cs
[Fact]
public async void ItShouldCallGetSpeakers()
{
  // Arrange
  var server = new TestServer(new WebHostBuilder()
      .UseStartup<Startup>()
      .ConfigureServices(services =>
      {
        services.AddDbContext<SpeakerMeetContext>(o =>
          o.UseInMemoryDatabase("SpeakerMeetInMemory"));
      }));

  var client = server.CreateClient();

  // Act
  var response = await client.GetAsync("/api/speaker");

  // Assert
  Assert.NotNull(response);
}
```

# ServerFixture

为了将设置从控制器测试中移除，再次使用测试固定装置。这次，创建一个名为`ServerFixture`的新类。这将是在控制器测试中设置的地方：

```cs
public class ServerFixture : IDisposable
{
  public TestServer Server { get; }
  public HttpClient Client { get; }

  public ServerFixture()
  {
    Server = new TestServer(new WebHostBuilder()
             .UseStartup<Startup>()
             .ConfigureServices(services =>
             {
               services.AddDbContext<SpeakerMeetContext>(o =>
                 o.UseInMemoryDatabase("SpeakerMeetContext"));
             }));

    if (Server.Host.Services.GetService(typeof(SpeakerMeetContext)) is SpeakerMeetContext context)
    {
      context.Speakers.Add(new Speaker {Id = 1, Name = "Test"...});
      context.SaveChanges();
    }

    Client = Server.CreateClient();
  }

  public void Dispose()
  {
```

```cs
    Server.Dispose();
    Client.Dispose();
  }
}
```

现在，回到之前的测试。修改测试类以使用`ServerFixture`：

```cs
[Collection("Controllers")]
[Trait("Category", "Integration")]
public class GetAll : IClassFixture<ServerFixture>
{
  private readonly HttpClient _client;

  public GetAll(ServerFixture fixture)
  {
    _client = fixture.Client;
  }

  [Fact]
  public async void ItShouldCallGetSpeakers()
  {
    // Act
    var response = await _client.GetAsync("/api/speaker");

    Assert.NotNull(response);
  }
}
```

现在，通过创建一个新的测试来验证响应返回了一个`OK`状态码：

```cs
[Fact]
public async void ItShouldReturnSuccess()
{
  // Act
  var response = await _client.GetAsync("/api/speaker/");
  response.EnsureSuccessStatusCode();

  // Assert
  Assert.Equal(HttpStatusCode.OK, response.StatusCode);
}
```

最后，确保返回了正确的演讲者：

```cs
[Fact]
public async void ItShouldReturnSpeakers()
{
  // Act
  var response = await _client.GetAsync("/api/speaker");
  response.EnsureSuccessStatusCode();

  var responseString = await response.Content.ReadAsStringAsync();
  var speakers = JsonConvert.DeserializeObject<List<SpeakerSummary>>(responseString);

  // Assert
  Assert.Equal(1, speakers[0].Id);
}
```

记住，你想要确保你的测试套件干净且维护良好。为了稍微清理这个测试，你可能想要考虑创建一个`ReadAsJsonAsync`扩展。这可能看起来是这样的：

```cs
public static class Extensions
{
  public static async Task<T> ReadAsJsonAsync<T>(this HttpContent content)
  {
    var json = await content.ReadAsStringAsync();

    return JsonConvert.DeserializeObject<T>(json);
  }
}
```

现在，修改测试以使用新的扩展方法：

```cs
[Fact]
public async void ItShouldReturnSpeakers()
{
  // Act
  var response = await _client.GetAsync("/api/speaker");
  response.EnsureSuccessStatusCode();

  var speakers = await response.Content.ReadAsJsonAsync<List<SpeakerSummary>>();

  // Assert
  Assert.Equal(1, speakers[0].Id);
}
```

这样就更好了。现在这个扩展可以被反复使用，并且它的第一次使用已经在`ItShouldReturnSpeakers`测试中进行了文档记录。

现在，继续测试单个演讲者端点是否可以被调用。创建一个名为`ItShouldCallGetSpeaker`的测试，并确保返回了一个响应：

```cs
[Fact]
public async void ItShouldCallGetSpeaker()
{
  // Act
  var response = await _client.GetAsync("/api/speaker/-1");

  Assert.NotNull(response);
}
```

现在，测试如果给定的 ID 不存在，是否返回了正确的响应码：

```cs
[Fact]
public async void ItShouldReturnError()
{
  // Act
  var response = await _client.GetAsync("/api/speaker/-1");

  // Assert
  Assert.Equal(HttpStatusCode.NotFound, response.StatusCode);
}
```

现在，验证当提供的 ID 存在时，返回了一个`OK`状态码：

```cs
[Fact]
public async void ItShouldReturnSuccess()
{
  // Act
  var response = await _client.GetAsync("/api/speaker/1");
  response.EnsureSuccessStatusCode();

  // Assert
  Assert.Equal(HttpStatusCode.OK, response.StatusCode);
}
```

最后，确认返回的演讲者正是预期的那个。注意，这里可以使用`ReadAsJsonAsync`：

```cs
[Fact]
public async void ItShouldReturnSpeaker()
{
  // Act
  var response = await _client.GetAsync("/api/speaker/1");
  response.EnsureSuccessStatusCode();

  var speakerSummary = await response.Content.ReadAsJsonAsync<SpeakerDetail>();

  // Assert
  Assert.Equal(1, speakerSummary.Id);
}
```

在前面的页面中，只有演讲者的`Get`和`GetAll`方法被测试过。你可以自由地添加对`Search`方法的测试，以扩展你的集成测试套件。

# 摘要

你现在应该对集成测试、其优势和劣势有了牢固的理解。模拟 API 调用已被移除，并实现了真实的 API 服务。已经创建了集成测试，并确保应用程序的各个部分能够良好地协同工作。

变化是不可避免的，尤其是在软件开发中。在下一章中，我们将讨论如何处理需求的变化。这些变化可能包括新功能、修复缺陷或更改现有逻辑，通过 TDD 这些都可以轻松管理。
