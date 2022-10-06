# clean-code-python

[![Build Status](https://travis-ci.com/zedr/clean-code-python.svg?branch=master)](https://travis-ci.com/zedr/clean-code-python)
[![](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/download/releases/3.8.3/)

## 目录
  1. [介绍](#介绍)
  2. [变量](#变量)
  3. [函数](#函数)
  5. [类](#类)
     1. [S: 单一功能原则（SRP：Single responsibility principle）](#s单一功能原则)
     2. [O: 开闭原则（OCP：Open/Closed Principle）](#开闭原则)
     3. [L: 里氏替换原则 （LSP：Liskov Substitution Principle）](#里氏替换原则)
     4. [I: 接口隔离原则 （ISP：Interface Segregation Principle）](#接口隔离原则)
     5. [D: 依赖倒置原则 （DIP：Dependency Inversion Principle (DIP)](#依赖倒置原则)
  6. [项目里不要有重复代码 （DRY：Don't repeat yourself）](#项目里不要有重复代码 )
  7. [其它语言版本](#其它语言版本)

## **介绍**
软件工程需要遵守的一些原则。参考了 Robert C. Martin's 的
[*代码整洁之道*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)，原书编程语言使用的是用 javascript，这里将它改写提炼成了 python 版本。核心思想是一样的。这并不是一个介绍 python 代码风格的手册。它的核心目的是帮助大家用 python 写出：可读性高，易用，易于重构的项目。

这里介绍的原则并不需要每一条都严格执行，也许有些人只会赞同里面的一部分。但是这些原则都是*代码整洁之道*作者多年经验的总结。

请注意这里的示例代码只能在 Python3.7 及以上的版本运行。

## **变量**

### 使用有意义，完整的单词作为变量名字。少用拼音，缩略语。

**不好的例子:**

```python
import datetime

shijian = datetime.date.today().strftime("%y-%m-%d") # 不要用拼音
ymdstr = datetime.date.today().strftime("%y-%m-%d") # 不要用缩略语，除非这些缩率语大家一眼能认出来，也不需要变量里面加类型。
```
另外，不需要再变量名字里面加类型（str）。

**改进：**:

```python
import datetime


current_date: str = datetime.date.today().strftime("%y-%m-%d")
```

**[⬆ 回到顶部](#目录)**

### 同一个项目中，表示同一个东西的时候请使用相同的变量名

**不好的例子:**
这里我们用了三个不同的变量名（user，client，customer）表示了同一个东西：

```python
def get_user_info(): pass
def get_client_data(): pass
def get_customer_record(): pass
```

**改进1**:
如果你想表示同一个的东西，在函数名及任何引用到它的地方，名字都需要一致。

```python
def get_user_info(): pass
def get_user_data(): pass
def get_user_record(): pass
```

**改进2**
Python 是一个面向对象的编程语言。可以把上面的函数封装成一个类，然后这些函数的功能就可以用类属性，类 property 方法（使用@property 装饰器，可以像访问类属性一样访问这类方法），类方法来实现。从下面这个例子中我们可以发现，在给类方法起名字的时候，我们不需要在把类名放进去，get_user_record 简化成了 get_record。

```python
from typing import Union, Dict


class Record:
    pass


class User:
    info : str

    @property
    def data(self) -> Dict[str, str]:
        return {}

    def get_record(self) -> Union[Record, None]:
        return Record()
```

**[⬆ 回到顶部](#目录)**

### 使用便于搜索的命名方式
我们在工作中，阅读别人的代码量会比自己写的代码量要多很多。为了方便他人阅读我们自己写的代码，我们需要保证我们的代码写的可读性高，检索起来方便。如果我们直接把一些变量写死，读者不明白它的具体含义，代码可读性就会很差。常见的就是魔鬼数字（magic number）。


**不好的例子:**

```python
import time


# What is the number 86400 for again?
time.sleep(86400) #魔鬼数字
```

**改进**:

```python
import time


# 将它变成常量，在模块的全局域声明.
SECONDS_IN_A_DAY = 60 * 60 * 24
time.sleep(SECONDS_IN_A_DAY)
```
**[⬆ 回到顶部](#目录)**

### 变量名要有意义
**不好的例子:**

```python
import re


address = "One Infinite Loop, Cupertino 95014"
city_zip_code_regex = r"^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$"

matches = re.match(city_zip_code_regex, address)
if matches:
    print(f"{matches[1]}: {matches[2]}")
```

**改进1**:

这个比上面那个例子好一些，但是仍然强依赖正则表达式。

```python
import re


address = "One Infinite Loop, Cupertino 95014"
city_zip_code_regex = r"^[^,\\]+[,\\\s]+(.+?)\s*(\d{5})?$"
matches = re.match(city_zip_code_regex, address)

if matches:
    city, zip_code = matches.groups()
    print(f"{city}: {zip_code}")
```

**改进2**:

对正则表达式做一些改进，给匹配的部分起个别名。

```python
import re


address = "One Infinite Loop, Cupertino 95014"
city_zip_code_regex = r"^[^,\\]+[,\\\s]+(?P<city>.+?)\s*(?P<zip_code>\d{5})?$"

matches = re.match(city_zip_code_regex, address)
if matches:
    print(f"{matches['city']}, {matches['zip_code']}")
```
**[⬆ 回到顶部](#目录)**

### 起名不要想当然
不要让阅读你代码的人猜变量名表示的含义，直接显式的在变量名中表示它想表达的东西。

**不好的例子:**

```python
seq = ("Austin", "New York", "San Francisco")

for item in seq:
    #do_stuff()
    #do_some_other_stuff()

    # Wait, what's `item` again?
    print(item)
```

**改进**:

```python
locations = ("Austin", "New York", "San Francisco")

for location in locations:
    #do_stuff()
    #do_some_other_stuff()
    # ...
    print(location)
```
**[⬆ 回到顶部](#目录)**


### 不要在变量名中加一些不必要冗余的东西

如果你的类/对象已经告诉读者一些信息，这些信息就不需要在类属性名或者方法名里面出现了。

**不好的例子:**

```python
class Car:
    car_make: str
    car_model: str
    car_color: str
```

**改进**:

```python
class Car:
    make: str
    model: str
    color: str
```

**[⬆ 回到顶部](#目录)**

### 必要的话给参数赋一个默认值，同时也可以显示的说明参数的类型。

**不好的例子**

Why write:

```python
import hashlib


def create_micro_brewery(name):
    name = "Hipster Brew Co." if name is None else name
    slug = hashlib.sha1(name.encode()).hexdigest()
    # etc.
```

**改进1**:

```python
import hashlib


def create_micro_brewery(name: str = "Hipster Brew Co."):
    slug = hashlib.sha1(name.encode()).hexdigest()
    # etc.
```

**[⬆ 回到顶部](#目录)**
## **函数**
### 函数的参数 (参数个数小于等于两个)
对函数的参数个数做限制是非常重要的，因为它可以使你的函数测试起来更简单。如果函数的参数个数超过三个，那么测试用例将会成几何数增加。

一个参数都没有的函数是最理想的。一个或者两个参数也是可以接受的。三个及三个以上要尽量避免。如果参数过多，有可能是这个函数的功能太复杂，要做的事情太多。如果不是这种情况，大多数情况可以把这些参数用一个类对象来封装起来，然后再作为函数的参数。

**不好的例子:**

```python
def create_menu(title, body, button_text, cancellable):
    pass
```

**Java-esque 表示法**:

```python
class Menu:
    def __init__(self, config: dict):
        self.title = config["title"]
        self.body = config["body"]
        # ...

menu = Menu(
    {
        "title": "My Menu",
        "body": "Something about my menu",
        "button_text": "OK",
        "cancellable": False
    }
)
```

**改进版1**

```python
class MenuConfig:
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: str
    body: str
    button_text: str
    cancellable: bool = False


def create_menu(config: MenuConfig) -> None:
    title = config.title
    body = config.body
    # ...


config = MenuConfig()
config.title = "My delicious menu"
config.body = "A description of the various items on the menu"
config.button_text = "Order now!"
# The instance attribute overrides the default class attribute.
config.cancellable = True

create_menu(config)
```

**改进版2**

```python
from typing import NamedTuple


class MenuConfig(NamedTuple):
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: str
    body: str
    button_text: str
    cancellable: bool = False


def create_menu(config: MenuConfig):
    title, body, button_text, cancellable = config
    # ...


create_menu(
    MenuConfig(
        title="My delicious menu",
        body="A description of the various items on the menu",
        button_text="Order now!"
    )
)
```

**改进版3**

```python
from dataclasses import astuple, dataclass


@dataclass
class MenuConfig:
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: str
    body: str
    button_text: str
    cancellable: bool = False

def create_menu(config: MenuConfig):
    title, body, button_text, cancellable = astuple(config)
    # ...


create_menu(
    MenuConfig(
        title="My delicious menu",
        body="A description of the various items on the menu",
        button_text="Order now!"
    )
)
```

**改进版4, Python3.8以上**

```python
from typing import TypedDict


class MenuConfig(TypedDict):
    """A configuration for the Menu.

    Attributes:
        title: The title of the Menu.
        body: The body of the Menu.
        button_text: The text for the button label.
        cancellable: Can it be cancelled?
    """
    title: str
    body: str
    button_text: str
    cancellable: bool


def create_menu(config: MenuConfig):
    title = config["title"]
    # ...


create_menu(
    # You need to supply all the parameters
    MenuConfig(
        title="My delicious menu",
        body="A description of the various items on the menu",
        button_text="Order now!",
        cancellable=True
    )
)
```
**[⬆ 回到顶部](#目录)**

### 一个函数只需要负责一件事情
在软件工程中，这是一件非常重要的事。如果函数做的事情过多，把各种事情串起来就会很麻烦，读者很难直观的理解这个函数的功能，测试也会很复杂。如果你可以保证你写的函数只负责一件事情或者一个功能，那么你的代码重构起来就会很简单，而且别人读起来也会比较轻松。如果你阅读完这篇内容，只记住了这个要求，你就已经比很多其他的程序员要厉害了。

**不好的例子:**

```python
from typing import List


class Client:
    active: bool


def email(client: Client) -> None:
    pass


def email_clients(clients: List[Client]) -> None:
    """Filter active clients and send them an email.
    """
    for client in clients:
        if client.active:
            email(client)
```

**改进1**:

```python
from typing import List


class Client:
    active: bool


def email(client: Client) -> None:
    pass


def get_active_clients(clients: List[Client]) -> List[Client]:
    """Filter active clients.
    """
    return [client for client in clients if client.active]


def email_clients(clients: List[Client]) -> None:
    """Send an email to a given list of clients.
    """
    for client in get_active_clients(clients):
        email(client)
```

有没有发现上面这个例子里面可以使用生成器。

**改进2**

```python
from typing import Generator, Iterator


class Client:
    active: bool


def email(client: Client):
    pass


def active_clients(clients: Iterator[Client]) -> Generator[Client, None, None]:
    """Only active clients"""
    return (client for client in clients if client.active)


def email_client(clients: Iterator[Client]) -> None:
    """Send an email to a given list of clients.
    """
    for client in active_clients(clients):
        email(client)
```


**[⬆ 回到顶部](#目录)**

### 函数的名字需要表达它的功能

**坏的例子:**

```python
class Email:
    def handle(self) -> None:
        pass

message = Email()
# handle 到底是干啥用的?
message.handle()
```

**改进**

```python
class Email:
    def send(self) -> None:
        """Send this message"""

message = Email()
message.send()
```

**[⬆ 回到顶部](#目录)**

### 函数应该只有一层抽象（abstraction）
如果你的函数有不止一层抽象，你的函数就太复杂了。把函数拆一下提高它的可复用性，并让它更容易测试。译者注：abstraction 这个翻译的不太好，这里想表达的含义是函数要简单，如果该函数实现的一个功能依赖另一个功能。那把依赖的那个功能也写一个函数。不要层层嵌套写到一起。可以体会一下下面的例子（需要有一些编译原理基础）。

**不好的例子:**

```python
# type: ignore

def parse_better_js_alternative(code: str) -> None:
    regexes = [
        # ...
    ]

    statements = code.split('\n')
    tokens = []
    for regex in regexes:
        for statement in statements:
            pass

    ast = []
    for token in tokens:
        pass

    for node in ast:
        pass
```

**改进1**

```python
from typing import Tuple, List, Dict


REGEXES: Tuple = (
   # ...
)


def parse_better_js_alternative(code: str) -> None:
    tokens: List = tokenize(code)
    syntax_tree: List = parse(tokens)

    for node in syntax_tree:
        pass


def tokenize(code: str) -> List:
    statements = code.split()
    tokens: List[Dict] = []
    for regex in REGEXES:
        for statement in statements:
            pass

    return tokens


def parse(tokens: List) -> List:
    syntax_tree: List[Dict] = []
    for token in tokens:
        pass

    return syntax_tree
```

**[⬆ 回到顶部](#目录)**

### 函数的参数里面不要出现 True/False 标志位

如果函数的参数里面出现了标志位，那就说明函数需要实现不止一个功能。需要把函数根据标志位拆分一下。


**不好的例子:**

```python
from tempfile import gettempdir
from pathlib import Path


def create_file(name: str, temp: bool) -> None:
    if temp:
        (Path(gettempdir()) / name).touch()
    else:
        Path(name).touch()
```

**改进**

```python
from tempfile import gettempdir
from pathlib import Path


def create_file(name: str) -> None:
    Path(name).touch()


def create_temp_file(name: str) -> None:
    (Path(gettempdir()) / name).touch()
```

**[⬆ 回到顶部](#目录)**

### 函数要避免副作用（side effect）

这里的副作用不是贬义词。一个函数的功能一般是接受参数，返回一些值。如果它在做这些事情的同时还做了其它的事情，这些其它的事情就是副作用（side effect）。这些副作用可以是写一个文件，改全局变量，也有可能一不小心把你账户里的钱划入某个陌生的账户。

如果你确实需要做这些事情，比如说在上个例子中，你需要写入一个文件。在这些情况下，你需要标记你引进入副作用的地方。不要让不同的函数和类同时操作相同的文件，直接通过某个特定函数来专门操作这个文件。

需要避免一些常见的陷阱：在不同对象之间共享状态；使用可变数据类型（列表，字典等），导致任何函数或变量可以操作这些数据；使用某个类实例，但是不将产生副作用的地方集中。
如果你能做到这点，你会比大多数其他程序员更轻松。

**不好的例子:**

```python
# type: ignore

# fullname 是这个模块里的全局变量，是 string 类型.
fullname = "Ryan McDermott"

def split_into_first_and_last_name() -> None:
    # fullname 是定义在这个模块里的全局变量，下面这行代码会改变这个全局变量的状态
    # 并产生一个副作用。

    global fullname
    fullname = fullname.split()

split_into_first_and_last_name()

# 调用完这个函数后fullname的类型就变了 由string 变成了 List[str]。再调用这个函数就会报错
# 'Incompatible types in assignment: 
# (expression has type "List[str]", variable has type "str")'
print(fullname)  # ["Ryan", "McDermott"]

```

**改进1**

```python
from typing import List, AnyStr


def split_into_first_and_last_name(name: AnyStr) -> List[AnyStr]:
    return name.split()

fullname = "Ryan McDermott"
name, surname = split_into_first_and_last_name(fullname)

print(name, surname)  # => Ryan McDermott
```

**改进2**

```python
from dataclasses import dataclass


@dataclass
class Person:
    name: str

    @property
    def name_as_first_and_last(self) -> list:
        return self.name.split()


# 这个类的目的就是统一控制状态!
person = Person("Ryan McDermott")
print(person.name)  # => "Ryan McDermott"
print(person.name_as_first_and_last)  # => ["Ryan", "McDermott"]
```

**[⬆ 回到顶部](#目录)**

## **类**

### **单一功能原则**

Robert C. Martin writes:

> 就一个类而言，应该仅有一个引起它变化的原因.

“变化的原因” 对应着类或者函数负责的功能。所以上面这句话换个说法就是一个类只负责一个功能。
下面这个例子中，我们创建了一个 HTML 元素，这个元素是 HTML 里一个注释，注释里写入pip的版本号。


**不好的例子：**

```python
from importlib import metadata


class VersionCommentElement:
   """An element that renders an HTML comment with the program's version number
   """

   def get_version(self) -> str:
       """Get the package version"""
       return metadata.version("pip")
   
   def render(self) -> None:
       print(f'<!-- Version: {self.get_version()} -->')


VersionCommentElement().render()
```

这里的类有两个功能:

 - 获得 pip 的版本号。
 - 生成HTML的注释元素

这里面任意一个功能的改变都会影响另一个。我们可以把这两个功能拆一下。

**改进**

```python
from importlib import metadata


def get_version(pkg_name:str) -> str:
    """Retrieve the version of a given package"""
    return metadata.version(pkg_name)


class VersionCommentElement:
   """An element that renders an HTML comment with the program's version number
   """

   def __init__(self, version: str):
       self.version = version
   
   def render(self) -> None:
       print(f'<!-- Version: {self.version} -->')


VersionCommentElement(get_version("pip")).render()
```

这样写的话，这个类只需要关注生成HTML元素。在实例化的时候，版本号作为初始化参数传了进去（版本号通过 `get_version()` 获得）。类和函数都是隔离的，任何一个发生改变都不会对另一个产生影响。

另外，`get_version()`  是可以被重用的。

### **开闭原则**

> “加入新的特性应该通过扩展的方式而不是修改的方式。
> Uncle Bob.

软件中的对象（类，模块，函数等等）应该对于扩展是开放的，但是对于修改是封闭的。一个对象（比如说一个类）需要保证在不修改内部逻辑的情况下，增加一些新的功能（通过增加代码实现而不是修改原有的代码）。对象在设计之初就要保证它的扩展性。在下面这个例子中，我们会实现一个简单的web框架来处理 HTTP 请求作出响应（返回一些内容）。当 HTTP 服务器收到一个 GET 请求的时候，`View` 类`.get()`方法会被调用。

`View` 只会简单的返回`text/plain`。我们希望返回 HTML 内容。所以我们就继承 `View` 类，写了一个`TemplateView` 类。

**不好的例子：**

```python
from dataclasses import dataclass


@dataclass
class Response:
    """An HTTP response"""

    status: int
    content_type: str
    body: str


class View:
    """A simple view that returns plain text responses"""

    def get(self, request) -> Response:
        """Handle a GET request and return a message in the response"""
        return Response(
            status=200,
            content_type='text/plain',
            body="Welcome to my web site"
        )


class TemplateView(View):
    """A view that returns HTML responses based on a template file."""

    def get(self, request) -> Response:
        """Handle a GET request and return an HTML document in the response"""
        with open("index.html") as fd:
            return Response(
                status=200,
                content_type='text/html',
                body=fd.read()
            )

```

为了实现新的功能， `TemplateView`  类把父类的内容改了，重新写了一个 `.get()` 方法。如果父类的 `.get()`不改变还好，如果改变了，比如说加一些额外的检查，子类的`.get()` 也需要同步修改。如果这样的子类很多，难免会有错漏。

我们可以重新设计一下 `View`  类，让他变得可扩展，而不需要修改原有的类方法。

**改进1**

```python
from dataclasses import dataclass


@dataclass
class Response:
    """An HTTP response"""

    status: int
    content_type: str
    body: str


class View:
    """A simple view that returns plain text responses"""

    content_type = "text/plain"

    def render_body(self) -> str:
        """Render the message body of the response"""
        return "Welcome to my web site"

    def get(self, request) -> Response:
        """Handle a GET request and return a message in the response"""
        return Response(
            status=200,
            content_type=self.content_type,
            body=self.render_body()
        )


class TemplateView(View):
    """A view that returns HTML responses based on a template file."""

    content_type = "text/html"
    template_file = "index.html"

    def render_body(self) -> str:
        """Render the message body as HTML"""
        with open(self.template_file) as fd:
            return fd.read()


```
注意我们还是需要重写`render_body()`，这样才能改变响应的内容。但这个方法的职责就是然让子类来重写来扩展功能的。

另一个方式就是利用继承和聚合的优点，使用[Mixins](https://docs.djangoproject.com/en/4.1/topics/class-based-views/mixins/)。

Mixins 类是基础类，他们就是给其它相关的类使用的。它们和目标类通过多继承的方式结合可以改变目标类的功能。

一些使用规则：
 - Mixins 必须继承自 `object`
 - Mixins 继承顺序在目标类之前，例如`class Foo(MixinA, MixinB, TargetClass): ...`

**改进2**

```python
from dataclasses import dataclass, field
from typing import Protocol


@dataclass
class Response:
    """An HTTP response"""

    status: int
    content_type: str
    body: str
    headers: dict = field(default_factory=dict)


class View:
    """A simple view that returns plain text responses"""

    content_type = "text/plain"

    def render_body(self) -> str:
        """Render the message body of the response"""
        return "Welcome to my web site"

    def get(self, request) -> Response:
        """Handle a GET request and return a message in the response"""
        return Response(
            status=200,
            content_type=self.content_type,
            body=self.render_body()
        )


class TemplateRenderMixin:
    """A mixin class for views that render HTML documents using a template file

    Not to be used by itself!
    """
    template_file: str = ""

    def render_body(self) -> str:
        """Render the message body as HTML"""
        if not self.template_file:
            raise ValueError("The path to a template file must be given.")

        with open(self.template_file) as fd:
            return fd.read()


class ContentLengthMixin:
    """A mixin class for views that injects a Content-Length header in the
    response

    Not to be used by itself!
    """

    def get(self, request) -> Response:
        """Introspect and amend the response to inject the new header"""
        response = super().get(request) # type: ignore
        response.headers['Content-Length'] = len(response.body)
        return response


class TemplateView(TemplateRenderMixin, ContentLengthMixin, View):
    """A view that returns HTML responses based on a template file."""

    content_type = "text/html"
    template_file = "index.html"

```
正如你看到的，通过把相关的功能封装到一个可重用的类里面，Mixins是对象的聚合更加容易。这个封装的类也符合单一职责原则。类的扩展就通过继承这些Mixins类来实现。

Django 就用了很多Mixins 来聚合它的 view 类。

FIXME: 等`typing.Protocol` 的使用方式明确了，需要在上面那行代码给 Mixins 加入类型检查。

### **里氏替换原则**
函数如果使用了某个基类指针或引用，也可以同样的使用基类的派生类，而不用关心这些派生的具体实现。
> “函数如果使用了某个基类指针或引用，也可以同样的使用基类的派生类，而不用关心这些派生的具体实现。
> Uncle Bob.


我们用 Barbara Liskov 来命名这条原则，他和计算机科学家 Jeannette Wing 发表了论文
*"A behavioral notion of subtyping" (1994). 论文的一个核心是派生类方法必须保留它的父类的该方法相同的功能和行为。

如果某个函数的参数可以传一个父类对象，那么这个父类的所有派生类对象都可以传给这个函数，而且这个函数不需要修改。

*译者注：*如果对上面这段不是很理解的话只要记住一点。父类的所有方法子类必须要都实现，要么继承，要么重写。如果重写的话，它实现的功能必须和父类该方法的功能相似，参数尽量保持一致。

来看看下面的代码有哪些问题。

**不好的例子：**

```python
from dataclasses import dataclass


@dataclass
class Response:
    """An HTTP response"""

    status: int
    content_type: str
    body: str


class View:
    """A simple view that returns plain text responses"""

    content_type = "text/plain"

    def render_body(self) -> str:
        """Render the message body of the response"""
        return "Welcome to my web site"

    def get(self, request) -> Response:
        """Handle a GET request and return a message in the response"""
        return Response(
            status=200,
            content_type=self.content_type,
            body=self.render_body()
        )


class TemplateView(View):
    """A view that returns HTML responses based on a template file."""

    content_type = "text/html"

    def get(self, request, template_file: str) -> Response: # type: ignore
        """Render the message body as HTML"""
        with open(template_file) as fd:
            return Response(
                status=200,
                content_type=self.content_type,
                body=fd.read()
            )


def render(view: View, request) -> Response:
    """Render a View"""
    return view.get(request)

```

`render()` 方法应该可以和`View` 类以及它的子类`TemplateView`配合使用，但是`TemplateView`在继承的时候把`.get()`方法的签名（方法的输入输出）给改了。使用``render()` 的时候 TemplateView`会抛出一个错误。

如果我们希望`render()` 可以被`View` 和它的所有派生类来使用，我们要注意不能破坏对外的接口。但是我们怎么能知道某个给定类的构成呢？输入*mypy*，当遇到类似的问题是它会抛出一个错误：

```
error: Signature of "get" incompatible with supertype "View"
<string>:36: note:      Superclass:
<string>:36: note:          def get(self, request: Any) -> Response
<string>:36: note:      Subclass:
<string>:36: note:          def get(self, request: Any, template_file: str) -> Response
```

### **接口隔离原则**

接口要简洁，这样用户就不需要依赖他们不需要的东西
> “接口要简洁，这样用户就不需要依赖他们不需要的东西。”
> Uncle Bob.

一些比较著名的面向对象编程语言比如说 Java 和 Go，有一个接口（interface）概念。一个接口类定义了抽象方法和属性，这些方法不需要具体实现。当我们想定义一个函数的签名（定义函数的输入输出类型）但不想具体实现它，接口就会变得非常有用。我们可以说：“我们并不关心你传给我的对象的细节，我只关心我会用到的类方法或属性。”

Python 没有接口，但是它提供了抽象类，这和接口有一些不一样，但可以实现相同的功能。

**好的例子**

```python

from abc import ABCMeta, abstractmethod


# Define the Abstract Class for a generic Greeter object
class Greeter(metaclass=ABCMeta):
    """An object that can perform a greeting action."""

    @staticmethod
    @abstractmethod
    def greet(name: str) -> None:
        """Display a greeting for the user with the given name"""


class FriendlyActor(Greeter):
    """An actor that greets the user with a friendly salutation"""

    @staticmethod
    def greet(name: str) -> None:
        """Greet a person by name"""
        print(f"Hello {name}!")


def welcome_user(user_name: str, actor: Greeter):
    """Welcome a user with a given name using the provided actor"""
    actor.greet(user_name)


welcome_user("Barbara", FriendlyActor())
```
现在想象下面一个场景：我们有一些PDF文档，我们想提供给我们网站的用户。我们想使用一个python web 框架来设计一个类管理这些文档。所以我们给文档设计了一个抽象基类，这个基类大而全，把一些可能用到的功能都写进去了。


**不好的例子**

```python
import abc


class Persistable(metaclass=abc.ABCMeta):
    """Serialize a file to data and back"""

    @property
    @abc.abstractmethod
    def data(self) -> bytes:
        """The raw data of the file"""

    @classmethod
    @abc.abstractmethod
    def load(cls, name: str):
        """Load the file from disk"""

    @abc.abstractmethod
    def save(self) -> None:
        """Save the file to disk"""


# 针对 PDF 文档 我们只需要实现`.load()` 方法，并给`data` 赋对应的值。

class PDFDocument(Persistable):
    """A PDF document"""

    @property
    def data(self) -> bytes:
        """The raw bytes of the PDF document"""
        ... # Code goes here - omitted for brevity

    @classmethod
    def load(cls, name: str):
        """Load the file from the local filesystem"""
        ... # Code goes here - omitted for brevity


def view(request):
    """A web view that handles a GET request for a document"""
    requested_name = request.qs['name'] # We want to validate this!
    return PDFDocument.load(requested_name).data

```

但是我们不可以！如果我们没有实现 `.save()` 方法，会抛出一个异常：

```
Can't instantiate abstract class PDFDocument with abstract method save.
```

这很烦人。我们不需要真的在这里实现`.save()`。我们可以给这个方法赋一个空的内容 或者写 `NotImplementedError`, 但是这些无用的代码我们需要避免。

同时如果我们把`.save()`从抽象类里移除了，当用户提交他们的文档的时候，我们需要把它重新加进去，那我们就会遇到和前面一样的问题。

问题总结一下就是：是我们写了一个接口，这个接口里面有一些我们现在用不上的特性。

解决方式是把这个接口拆成更小的接口，每个新的接口负责一部分内容。


**改进**

```python
import abc


class DataCarrier(metaclass=abc.ABCMeta):
    """Carries a data payload"""
    @property
    def data(self):
        ...

class Loadable(DataCarrier):
    """Can load data from storage by name"""
    @classmethod
    @abc.abstractmethod
    def load(cls, name: str):
        ...

class Saveable(DataCarrier):
    """Can save data to storage"""
    @abc.abstractmethod
    def save(self) -> None:
        ...


class PDFDocument(Loadable):
    """A PDF document"""

    @property
    def data(self) -> bytes:
        """The raw bytes of the PDF document"""
        ... # Code goes here - omitted for brevity

    @classmethod
    def load(cls, name: str):
        """Load the file from the local filesystem"""
        ... # Code goes here - omitted for brevity


def view(request):
    """A web view that handles a GET request for a document"""
    requested_name = request.qs['name'] # We want to validate this!
    return PDFDocument.load(requested_name).data

```

### **依赖倒置原则**

> “依赖抽象，而不是具体的细节”,
> Uncle Bob.

想象我们想写一个web view 来返回HTTP响应。这个响应我们希望使用 csv 的数据格式来返回。我们可以使用标准库提供的 CSV writer。


**不好的例子**

```python
import csv
from io import StringIO


class StreamingHttpResponse:
    """A streaming HTTP response"""
    ... # implementation code goes here


def some_view(request):
   rows = (
       ['First row', 'Foo', 'Bar', 'Baz'],
       ['Second row', 'A', 'B', 'C', '"Testing"', "Here's a quote"]
   )
   # Define a generator to stream data directly to the client
   def stream():
       buffer_ = StringIO()
       writer = csv.writer(buffer_, delimiter=';', quotechar='"')
       for row in rows:
           writer.writerow(row)
           buffer_.seek(0)
           data = buffer_.read()
           buffer_.seek(0)
           buffer_.truncate()
           yield data

   # Create the streaming response  object with the appropriate CSV header.
   response = StreamingHttpResponse(stream(), content_type='text/csv')
   response['Content-Disposition'] = 'attachment; filename="somefilename.csv"'

   return response

```

我们的第一个实现使用了 CSV writer接口。通过操作`StringIO` 对象（类似一个文件）这些使用执行了一些底层操作以向writer中写入数据。这些操作比较繁杂而且不优雅。

更好的方式是明白 writer 只需要一个包含`.write()` 方法的对象。为什么不给它一个假的对象，这个对象可以及时的返回新的获得行数据。这样 `StreamingHttpResponse` 类可以及时的将它返回给发出请求的用户。


**改进**

```python
import csv


class Echo:
   """An object that implements just the write method of the file-like
   interface.
   """
   def write(self, value):
       """Write the value by returning it, instead of storing in a buffer."""
       return value

def some_streaming_csv_view(request):
   """A view that streams a large CSV file."""
   rows = (
       ['First row', 'Foo', 'Bar', 'Baz'],
       ['Second row', 'A', 'B', 'C', '"Testing"', "Here's a quote"]
   )
   writer = csv.writer(Echo(), delimiter=';', quotechar='"')
   return StreamingHttpResponse(
       (writer.writerow(row) for row in rows),
       content_type="text/csv",
       headers={'Content-Disposition': 'attachment; filename="somefilename.csv"'},
   )

```

这样实现就比前面的好很多，更加优雅。它的优点很明显：用更少的代码实现了相同的功能。我们利用了 writer 类只关心参数类里`.write()`这个抽象的方法，而不关心它内部的实现细节。
这个例子来源于
[a submission made to the Django documentation](https://code.djangoproject.com/ticket/21179)
.

**[⬆ 回到顶部](#目录)**

## **项目里不要有重复代码**

可以去[维基百科](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 看一下这个原则的介绍 
项目里尽量不要有重复代码。重复代码意味着：你修改某一处代码逻辑时，那些重复的地方也需要同步的进行修改。如果重复代码过多，修改的工作量会很大而且极易产生错漏。

想象一下，如果您经营一家餐馆，并盘点您的库存：
西红柿、洋葱、大蒜、香料等。如果您有多个清单
那么当你用了一个西红柿做了西红柿炒蛋。随后你必须更新所有的清单上的西红柿和鸡蛋的数据，。如果您只有一个列表，那么只需要更新一次。

写重复代码的原因主要是想要实现的功能仅有一两处不同，其它地方都是相同的。就是这些很少的不同，让你需要用多个具有重复代码的函数去实现，尽管这些函数的大部分代码都是共用的。想要去除重复代码，需要把共同的部分给抽象出来，然后用一个函数/模块/类来处理那些不同的地方。

具有好的抽象思维对一个程序员来说是至关重要的。不好的抽象带来的危害比重复代码更严重。如果你可以做一个好的抽象，一定要去做。不要写重复代码，否则你会发现想改一处逻辑的时候你的代码需要改动的地方会很多。
译者注：abstraction 这个含义比较难解释，百度了一下，含义如下：从众多的具体事物中，抽取共同的、本质的属性，舍弃个别的、非本质的属性，从而形成概念。

**不好的例子:**

```python
from typing import List, Dict
from dataclasses import dataclass

@dataclass
class Developer:
    def __init__(self, experience: float, github_link: str) -> None:
        self._experience = experience
        self._github_link = github_link
        
    @property
    def experience(self) -> float:
        return self._experience
    
    @property
    def github_link(self) -> str:
        return self._github_link
    
@dataclass
class Manager:
    def __init__(self, experience: float, github_link: str) -> None:
        self._experience = experience
        self._github_link = github_link
        
    @property
    def experience(self) -> float:
        return self._experience
    
    @property
    def github_link(self) -> str:
        return self._github_link
    

def get_developer_list(developers: List[Developer]) -> List[Dict]:
    developers_list = []
    for developer in developers:
        developers_list.append({
        'experience' : developer.experience,
        'github_link' : developer.github_link
            })
    return developers_list

def get_manager_list(managers: List[Manager]) -> List[Dict]:
    managers_list = []
    for manager in managers:
        managers_list.append({
        'experience' : manager.experience,
        'github_link' : manager.github_link
            })
    return managers_list

## create list objects of developers
company_developers = [
    Developer(experience=2.5, github_link='https://github.com/1'),
    Developer(experience=1.5, github_link='https://github.com/2')
]
company_developers_list = get_developer_list(developers=company_developers)

## create list objects of managers
company_managers = [
    Manager(experience=4.5, github_link='https://github.com/3'),
    Manager(experience=5.7, github_link='https://github.com/4')
]
company_managers_list = get_manager_list(managers=company_managers)
```

**改进**

```python
from typing import List, Dict
from dataclasses import dataclass

@dataclass
class Employee:
    def __init__(self, experience: float, github_link: str) -> None:
        self._experience = experience
        self._github_link = github_link
        
    @property
    def experience(self) -> float:
        return self._experience
    
    @property
    def github_link(self) -> str:
        return self._github_link
    


def get_employee_list(employees: List[Employee]) -> List[Dict]:
    employees_list = []
    for employee in employees:
        employees_list.append({
        'experience' : employee.experience,
        'github_link' : employee.github_link
            })
    return employees_list

## create list objects of developers
company_developers = [
    Employee(experience=2.5, github_link='https://github.com/1'),
    Employee(experience=1.5, github_link='https://github.com/2')
]
company_developers_list = get_employee_list(employees=company_developers)

## create list objects of managers
company_managers = [
    Employee(experience=4.5, github_link='https://github.com/3'),
    Employee(experience=5.7, github_link='https://github.com/4')
]
company_managers_list = get_employee_list(employees=company_managers)
```



**[⬆ 回到顶部](#目录)**

## **其它语言版本**

其它语言版本:

- :en: **English:** [https://github.com/zedr/clean-code-pythonn](https://github.com/zedr/clean-code-python)
- :pt: :br: **Portugese** [fredsonchaves07/clean-code-python](https://github.com/fredsonchaves07/clean-code-python)
- :iran: **Persian:** [https://github.com/SepehrRasouli/clean-code-python](https://github.com/SepehrRasouli/clean-code-python)


**[⬆ back to top](#table-of-contents)**
