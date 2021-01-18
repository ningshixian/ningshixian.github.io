---
title:      Rasa Open Source & Rasa X
subtitle:   Rasa Open Source & Rasa X
date:       2020-12-14
author:     NSX
catalog: true
tags:
    - Rasa
    - Rasa X

---

- Rasa Core
- Rasa X
- Rasa Action Server

 <!-- more -->



# Rasa Core

https://jiangdg.blog.csdn.net/article/details/105434136

Rasa框架提供的对话管理模块，即Dialog Management(DM)

它控制着人机对话的过程，是人机对话系统的重要组成部分。DM会根据NLU模块输出的语义表示执行对话状态的更新和追踪，并根据一定策略选择相应的候选动作。在对话过程中不断根据当前状态决定下一步应该采取的最优动作（如：提供结果，询问特定限制条件，澄清或确认需求等）

![img](https://flashgene.com/wp-content/uploads/2020/04/9d373f74b2cbb4925ff0cc2e7649db9e.jpg)

他有两个任务：

1. 对话状态维护（dialog state tracking, DST）

  对话状态是指记录了哪些槽位已经被填充、下一步该做什幺、填充什幺槽位，还是进行何种操作。用数学形式表达为，t+1 时刻的对话状态S(t+1)，依赖于之前时刻 t 的状态St，和之前时刻 t 的系统行为At，以及当前时刻 t+1 对应的用户行为O(t+1)。可以写成S(t+1)←St+At+O(t+1)。

2. 生成系统决策（dialog policy）

  根据 DST 中的对话状态（DS），产生系统行为（dialog act），决定下一步做什幺 dialog act 可以表示观测到的用户输入（用户输入 -> DA，就是 NLU 的过程），以及系统的反馈行为（DA -> 系统反馈，就是 NLG 的过程）

**对于1，Rasa实现很简单，就是简单地基于策略的槽状态替换。**

**对于2，Rasa使用基于LSTM的排序学习，大体上是将当前轮用户意图、上一轮系统行为、当前槽值状态向量化，然后与所有系统行为做相似度学习，以此决定当前轮次的一个或多个系统行为**



## Stories

Stories 是用于训练 Rasa Core的训练示例，包含3部分：

1. **用户输入（User Messages）**
   
   - \* greet 表示用户输入没有entity情况
   - \* inform{“people”: “six”} 表示用户输入包含entity情况，响应这类intent为普通action；
- \* request_weather 表示用户输入Message对应的intent为自定义表单动作 (form action)情况；
  
2. **动作（Actions）**

   Actions是机器针对用户输入的响应。Rasa中有3种actions：

   - default actions：默认的一组actions，无需定义. 

     包括以下三种action：

     1. action_listen：监听action，Rasa Core在会话过程中通常会自动调用该action；
     2. action_restart：重置状态，比初始化Slots(插槽)的值等；
     3. action_default_fallback：当Rasa Core得到的置信度低于设置的阈值时，默认执行该action；

   - utter actions：以utter_为开头，仅仅用于向用户发送一条消息作为反馈的一类actions

     在domain文件中添加utterance template，以`utter_`开头：

     ```
     responses:
       utter_my_message:
       - "this is what I want my action to say!"
     ```

   - custom actions：自定义action，允许开发者执行任何操作并反馈给用户

     - 自定义action需要我们在 domain.yml 文件中的actions部分先进行定义，然后在指定的webserver中实现它，其中，这个webserver的url地址在 endpoint.yml 文件中指定，并且这个webserver可以通过任何语言实现，当然这里首先推荐python来做，毕竟Rasa Core为我们封装好了一个rasa-core-sdk专门用来处理自定义action

       ```
       action_endpoint:
         url: "http://localhost:5055/webhook"
       ```

     - 另外，FormAction也是自定义actions，但是需要在domainl.yaml文件的forms字段声明

3. **事件（Events）**

   Events也是使用-开头，主要包含槽值设置(SlotSet)和激活/注销表单(Form)，它是是Story的一部分，并且必须显示的写出来。Slot Events和Form Events的作用如下：

   1. Slot Events：

      Slot Events的作用当我们在**自定义Action**中设置了某个槽值，那幺我们就需要在Story中Action执行之后显着的将这个SlotSet事件标注出来，格式为 `- slot{“slot_name”: “value”}` 。比如，我们在action_fetch_profile中设置了Slot名为account_type的值，代码如下：

      ```python
      from rasa_sdk.actions import Action
      from rasa_sdk.events import SlotSet
      import requests
      
      class FetchProfileAction(Action):
          def name(self):
              return "fetch_profile"
      
          def run(self, dispatcher, tracker, domain):
              url = "http://myprofileurl.com"
              data = requests.get(url).json
              return [SlotSet("account_type", data["account_type"])]
      ```

      那么，就需要在Story中执行`action_fetch_profile`之后，添加`- slot{"account_type" : "premium"}`。虽然，这么做看起来有点多余，但是Rasa规定这么做必须的，目的是提高训练时准确度。

      ```
      ## fetch_profile
      * fetch_profile
         - action_fetch_profile
         - slot{"account_type" : "premium"}
         - utter_welcome_premium
      ```

      当然，如果您的自定义Action中将槽值重置为None，则对应的事件为`-slot{"slot_name": null}`。

   2. Form Events：

      ```
      <!-- Form Action-->
      ## happy path 
      * request_weather
         - weather_form
         - form{"name": "weather_form"}
         - form{"name": null}
      ```

      在Story中主要存在三种形式的表单事件(Form Events)，它们可表述为：

      - Form Action事件：自定义Action的一种，用于一个表单操作。示例如下：

        ```
        - weather_form
        ```

      - Form activation事件：激活表单事件，当form action事件执行后，会立马执行该事件。示例如下：

        ```
        - form{"name": "weather_form"}
        ```

      - Form deactivation事件：注销表单事件，作用与form activation相反。示例如下：

        ```
        - form{"name": null}
        ```

总之，我们在构建Story时，可以说是多种多样的，因为设计的故事情节是多种多样的，这就意味着上述三种内容的组合也是非常灵活的。另外，在设计Story时Rasa还提供了[Checkpoints ](https://rasa.com/docs/rasa/core/stories/#id9)和[OR statements](https://rasa.com/docs/rasa/core/stories/#id9)两种功能，来提升构建Story的灵活度，但是需要注意的是，东西虽好，但是不要太贪了，过多的使用不仅增加了复杂度，同时也会拖慢训练的速度。



## Domain

- Domain，描述了对话机器人应知道的所有信息

  类似于“人的大脑”，存储了**意图intents、实体entities、插槽slots、动作actions、响应、表单等**信息。domain.yml文件组成结构如下：

  ![img](https://flashgene.com/wp-content/uploads/2020/04/e85b0bc1accb546be2a817523b5ee22c.png)

  1. intents

     意图的定义是在NLU样本中实现的，并且在每个意图下面我们需要枚举尽可多的样本用于训练，以达到Bot能够准确识别出我们输入的一句话到底想要干什幺

  2. session_config

     即会话配置，这部分的作用为配置一次会话(conversation session)是否有超时限制

  3. slots

     即插槽，它就像对话机器人的内存，它通过键值对的形式可用来收集存储用户输入的信息(实体)或者查询数据库的数据

  4. entities

     即实体，类似于输入文本中的关键字，需要在NLU样本中进行标注，然后Bot进行实体识别，并将其填充到Slot槽中，便于后续进行相关的业务操作

  5. actions

     当Rasa NLU识别到用户输入Message的意图后，Rasa Core对话管理模块就会对其作出回应，而完成这个回应的模块就是action

     ```
     actions:
     – utter_answer_affirm
     – utter_answer_deny
     – utter_answer_greet
     ```

  6. forms

     即表单，该部分列举了在NLU样本中自定义了哪些表单动作 (Form Actions)

     ```
   forms:
       - weather_form
     ```
  
  7. responses
  
     描述UtterActions具体的回复内容，并且每个UtterAction下可以定义多条信息，当用户发起一个意图，比如 “你好!”，就触发utter_answer_greet操作，Rasa Core会从该action的模板中自动选择其中的一条信息作为结果反馈给用户。除此之外，还可以在训练数据中存储Responses；或者自定义一个NLG服务来生成Responses。
     
     ```
     responses:
       utter_answer_greet:
         - text: "您好！请问我可以帮到您吗？"
         - text: "您好！很高兴为您服务。请说出您要查询的功能？"
         
       utter_ask_date_time:
         - text: "请问您要查询哪一天的天气？"
     ```

## Slots

- Slots

  - Slots Type

    1. Text：文本
    2. Boolean：布尔型
    3. Categorical：多个值中的一个
    4. Float：浮点数
    5. List：
    6. Unfeaturized：不影响对话的数据

  - slots 填充方式

    - 通过initial_value字段为当前slot提供一个初始值
    - Slots Set from NLU `\* greet{“name”: “Ali”}`
    - Slots Set By Clicking Buttons
    - Slots Set by Actions：在Custom Action中通过返回事件来填充Slots的值

  - Slots值的两种获取方式

    - Get Slot in responses

      在domain.yaml的responses部分，可以通过{slotname}的形式获取槽值

      ```
      responses:
      
      utter_greet:
      – text: “Hey, {name}. How are you?”
      ```

    - Get Slot in Custom Action

      通过Tracker，能够轻松获取整个对话信息，其中就包括Slot的值

      ```
      from rasa_sdk.actions import Action
      from rasa_sdk.events import SlotSet
      import requests
      
      class FetchProfileAction(Action):
          def name(self):
          	return “fetch_profile”
          def run(self, dispatcher, tracker, domain):
              # 获取slot account_type的值
              account_type = tracker.get_slot(‘account_type’)
              return []
      ```

## Form 

在Rasa Core中，当我们执行一个action需要同时填充多个slot时，可以使用 **FormAction** 来实现，因为FormAction会遍历监管的所有slot，当发现相关的slot未被填充时，就会向用户主动发起询问，直到所有slot被填充完毕，才会执行接下来的业务逻辑。使用步骤如下：

1. 构造story

   ```
   * request_weather
       - weather_form
       - form{"name": "weather_form"}  激活form
       - form{"name": null}  使form无效
   ```

2. 添加form字段到Domain

   ```
   intents:
     - request_weather
   
   forms:
     - weather_form
   ```

3. configs.yml新增FormPolicy策略

   ```
   policies:
     - name: FormPolicy
   ```

4. **Form Action实现**

   ```python
   class WeatherForm(FormAction):
   
       def name(self) -> Text:
           """Unique identifier of the form"""
   
           return "weather_form"
   
       @staticmethod
       def required_slots(tracker: Tracker) -> List[Text]:
           """A list of required slots that the form has to fill"""
   
           return ["date_time", "address"]
   
       def submit(
               self,
               dispatcher: CollectingDispatcher,
               tracker: Tracker,
               domain: Dict[Text, Any],
       ) -> List[Dict]:
           """Define what the form has to do
               after all required slots are filled"""
           address = tracker.get_slot('address')
           date_time = tracker.get_slot('date_time')
   
           return []
   ```

当form action第一被调用时，form就会被激活并进入FormPolicy策略模式。

**每次执行form action，required_slots会被调用，当发现某个还未被填充时，会主动去调用形式为uter_ask_{slotname}的模板(注：定义在domain.yml的templates字段中)；**

当所有slot被填充完毕，submit方法就会被调用，此时本次form操作完毕被取消激活。



## Policies 对话策略

Policies：使用合适的策略（Policy）来预测一次对话后要执行的行为（Actions）

在策略配置文件config.yml文件中，配置不同策略；

Rasa对提供的Policies进行了优先级排序，具体如下表：

![img](https://flashgene.com/wp-content/uploads/2020/04/68e68cbe6bf8a5e4f8bddb1f5eb5c2be.png)
1. Memoization Policy

   记忆策略只记录训练数据中的对话

   若训练数据存在这样的对话，则以置信度1.0预测下一个动作，否则以0.0预测

2. Keras Policy

   Keras框架实现神经网络 `LSTM+Dense+softmax`，来预测选择执行下一个action

3. Embedding Policy (TEDP)

   Rasa的最新技术，在多轮对话中表现比Keras Policy好

   与其他RNN机器学习算法不同，它利用transformer学习模式并进行预测。能更好处理不同语料库间的低纠缠，能更好地处理意外的用户输入，如闲聊

4. Form Policy

   表单策略，用于处理(form)表单的填充事项，收集指定信息，如性别年龄地址

   需要实现FormAction，在`domain.yml`中指定，在`stories.md`中使用

   当一个FormAction被调用时，FormPolicy将持续预测表单动作，直到表单中的所有槽都被填满，然后再执行对应的FormAction。

   ```python
   from rasa_sdk import Tracker
   from rasa_sdk.forms import FormAction
   from typing import Dict, Text, Any, List
   from rasa_sdk.executor import CollectingDispatcher
   
   
   class FacilityForm(FormAction):
       def name(self) -> Text:
           return "facility_form"
   
       @staticmethod
       def required_slots(tracker: Tracker) -> List[Text]:
           return ["facility_type", "location"]
   
       def slot_mappings(self) -> Dict[Text, Any]:
           return {"facility_type": self.from_entity(entity="facility_type", intent=["inform", "search_provider"]),
                   "location": self.from_entity(entity="location", intent=["inform", "search_provider"])}
   
       def submit(self, dispatcher: CollectingDispatcher, tracker: Tracker, domain: Dict[Text, Any]) -> List[Dict]:
           location = tracker.get_slot('location')
           facility_type = tracker.get_slot('facility_type')
           dispatcher.utter_button_message("Here is home health agency near you")
           return []
   ```

   ```
   forms:
   - facility_form
   ```

5. Mapping Policy

  直接将意图映射到要执行的action

  无视之前对话，一旦触发意图就操作

  映射是传递intent属性给`triggers`实现的，修改`domain.yml`

  ```
  intents:
    - ask_is_bot:
      triggers: action_is_bot
  ```

  ```
  intents:
  – greet: {triggers: utter_goodbye}
  ```

6. Fallback Policy

  回退策略，聊天机器人不可避免需要的回退情况，例如用户问了让机器人理解不了的东西时需要回退

  需要提供阈值

  ```
  policies:
    - name: FallbackPolicy
      nlu_threshold: 0.3
      ambiguity_threshold: 0.1
      core_threshold: 0.3
      fallback_action_name: 'action_default_fallback'
  ```



## Interactive Learning

虽然我们可以容易的人工构建story样本数据，但是往往会出现一些考虑不全，甚至出错等问题，基于此，Rasa Core框架为我们提供了一种交互式学习(Interactive Learning)来获得所需的样本数据。

1. 开启Action Server

```
python -m rasa run actions --port 5055 --actions actions --debug
```

2. 开启Interactive Learning

```
python -m rasa interactive -m models/20200313-101055.tar.gz –endpoints configs/endpoints.yml –config configs/config.yml

\# 或者（没有已训练模型情况）

\# rasa会先训练好模型，再开启交互式学习会话

python -m rasa interactive –data /data –domain configs/domain.yml –endpoints configs/endpoints.yml –config configs/config.yml
```

分别执行(1)、(2)命令后，我们可以预设一个交互场景根据终端的提示操作即可。如果一个交互场景所有流程执行完毕，按Ctrl+C结束并选择Start Fresh进入下一个场景即可。当然Rasa还提供了可视化界面，以帮助你了解每个Story样本构建的过程，网址：http://localhost:5005/visualization.html。



# Rasa X

- layers on top of Rasa Open Source and helps you build a better assistant

- is a free, closed source tool available to all developers
- can be deployed anywhere, so your training data stays secure and proprietary

Rasa X可以以本地模式安装，也可以使用Kubernetes / Openshift或Docker Compose安装在服务器上。请考虑以下哪种描述最能描述您的情况，并相应地选择一种安装方法。



## 本地模式

如果以下任何一种描述您的用例，则最好的方法是：

- 您正在开发自己的Rasa助手，并希望在用户界面中与其聊天并与他人共享
- 您是第一次浏览Rasa X，只想了解它在本地的工作方式

在以下情况下，您将需要使用其他安装选项之一：

- 您已经准备好部署助手并将其提供给最终用户

[本地模式安装指南](https://rasa.com/docs/rasa-x/installation-and-setup/install/local-mode)



## 服务器快速安装

这是在服务器或群集上安装Rasa X的最简单，最快的方法。如果您不确定选择哪种方法，请尝试这种方法。

如果以下任何一种描述您的用例，则最好的方法是：

- 您正在安装要在生产中使用的Rasa X，并且不需要太多自定义
- 您是第一次浏览Rasa X，并想了解它在服务器上的工作方式

在以下情况下，您将需要使用其他服务器安装选项之一：

- 您的系统不符合[要求](https://rasa.com/docs/rasa-x/installation-and-setup/install/quick-install-script#requirements)
- 除了[此方法可用的](https://rasa.com/docs/rasa-x/installation-and-setup/customize#server-quick-install)定制之外，您还需要其他定制
- 您期望机器人有大量流量（数百个并发用户），并且需要扩展
- 您正在安装Rasa Enterprise

[服务器快速安装指南](https://rasa.com/docs/rasa-x/installation-and-setup/install/quick-install-script)



## Docker撰写

如果满足以下条件，则使用Docker Compose在服务器上安装Rasa X是一个不错的选择：

- 您不能或不想使用Kubernetes / Openshift
- 您需要比快速安装脚本提供的更多自定义设置
- 您不会期望大量的用户流量（即数百个并发用户）

安装脚本可用于满足[硬件和操作系统要求的](https://rasa.com/docs/rasa-x/installation-and-setup/install/docker-compose#requirements)服务器 。对于其他操作系统，您可以通过Docker Compose手动安装RasaX。

[Docker Compose安装指南](https://rasa.com/docs/rasa-x/installation-and-setup/install/docker-compose)



## 注意❤❤❤

启动 `rasa x` 的过程中，windows下可能会遇到如下问题

```
rasa.shared.exceptions.FileIOException: Failed to read file 'C:\Users\NINGSH~1\AppData\Local\Temp\tmpmqoghnak', could not read the file using utf-8 to decode it. Please make sure the file is stored with this encoding.
```

解决：win10下需保证所有 XXX.yml 等配置文件不包含中文，或者是修改rasa\shared\utils\io.py代码：

```
def read_file(filename: Union[Text, Path], encoding: Text = DEFAULT_ENCODING) -> Any:
    """Read text from a file."""

    import codecs
    try:
        with codecs.open(filename, 'r', encoding=encoding) as f:
            return f.read()
    except FileNotFoundError:
        raise FileNotFoundException(
            f"Failed to read file, " f"'{os.path.abspath(filename)}' does not exist."
        )
    except UnicodeDecodeError:
        try:
            with codecs.open(filename, 'r', encoding='gb2312') as f:
                return f.read()
        except Exception:
            raise FileIOException(
                f"Failed to read file '{os.path.abspath(filename)}', "
                f"could not read the file using {encoding} to decode "
                f"it. Please make sure the file is stored with this "
                f"encoding."
            )
```



# Rasa Action Server

> 为Rasa开源对话助手运行[自定义操作](https://rasa.com/docs/rasa/next/custom-actions)

## 它是如何工作

当您的助手预测自定义动作时，Rasa服务器将`POST`使用json有效负载向动作服务器发送请求，其中包括预测动作的名称，对话ID，跟踪器的内容和域的内容。

当动作服务器完成自定义动作的运行时，它将返回[响应](https://rasa.com/docs/rasa/next/responses)和[事件](https://rasa.com/docs/action-server/events)的json有效负载。然后，Rasa服务器将响应返回给用户，并将事件添加到会话跟踪器。

## Running a Rasa SDK Action Server

```
rasa run actions
```

## 编写自定义Action

`Action Class`为任何自定义操作的基类。要定义自定义动作，请创建`Action`该类的子类并覆盖两个必需的方法`name`和`run`。当动作服务器`name`收到运行动作的请求时，它将根据其方法的返回值调用动作。

骨架自定义操作如下所示：

```
class MyCustomAction(Action):

    def name(self) -> Text:
        return "action_name"

    async def run(
        self, dispatcher, tracker: Tracker, domain: Dict[Text, Any],
    ) -> List[Dict[Text, Any]]:

        return []
```

### Action.name

定义动作的名称。该方法返回的名称是您的机器人域中使用的名称。

### Action.run 

```
async Action.run(dispatcher, tracker, domain)
```

**参数**

- **dispatcher** –用于将消息发送回用户的调度程序。

  - `utter_message`方法可用于将任何类型的响应返回给用户

    ```
    dispatcher.utter_message(text = "Hey there")
    dispatcher.utter_message(image = "")
    dispatcher.utter_message(json_message = {...})
    dispatcher.utter_message(template = "utter_greet")
    dispatcher.utter_message(template = "utter_greet_name", name = "Aimee")
    dispatcher.utter_messsage(buttons = [
                    {"payload": "/affirm", "title": "Yes"},
                    {"payload": "/deny", "title": "No"},
                ]
    ```

- [**tracker**](https://rasa.com/docs/action-server/sdk-tracker) –当前用户的状态跟踪器。可以通过`Tracker`属性和方法获取有关过去事件和会话当前状态的信息，如：访问插槽值 `tracker.get_slot(slot_name)`，最新的用户消息是`tracker.latest_message.text`以及任何其他 `rasa_sdk.Tracker`属性。请参阅[跟踪器](https://rasa.com/docs/action-server/sdk-tracker)的[文档](https://rasa.com/docs/action-server/sdk-tracker)。

- **domain**–机器人的域



# 遇到的坑

1. 训练数据JSON格式实体entity为中文，报错`UnicodeEncodeError: 'ascii' codec can't encode characters`，entity命名应使用英文
2. 根目录不要有文件`test.py`，启动API服务时会运行该文件，可能会报错
3. 项目名不要带中文，启动Rasa X时可能报错
4. YAML文件涉及字符串最好用`"`包围起来，特别为了调用Rasa API时。如`- payload: "/inform{\"gender\": \"男\"}"`
5. 一个Form激活时输入了无关数据报错`Failed to extract slot xxx with action xxx_form`，可在Stories中添加`action_deactivate_form`停止



# 参考

400 多行代码！超详细 Rasa 中文聊天机器人开发指南
https://flashgene.com/archives/110272.html

Rasa中文聊天机器人开发指南

https://jiangdg.blog.csdn.net/article/details/104328946

Rasa入门——AI助手和聊天机器人

https://blog.csdn.net/lly1122334/article/details/104041589