# Prompt-Engineering-Guide-zh
# Prompt工程指南中文版

本文翻译改编自**[Dair-ai/Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide)**，中文原帖地址为**[大型语言模型Prompt书写指南](https://www.acceleratori.com/d/10615/5)**，为了方便理解以及补充缺少的知识点，内容有所添改。除非另有说明，本文中所有的例子都是使用text-davinci-003（GPT-3）（使用OpenAI的Playground）测试的。案例使用默认的配置，例如temperatur和top-p参数分别为0.7与1。本文所有的输入与输出都为英文，中文翻译仅用于辅助理解，并非实际输入输出，因此在出现输入的位置，本文将保留英语原文作为参考。

本文主要分为以下几个板块

Prompt工程-**介绍**

Prompt工程-**基础Prompt搭建**

Prompt工程-**进阶Prompt搭建**

Prompt工程-**对抗性Prompt搭建**

Prompt工程-**其他主题**

# **1. Prompt工程-介绍**

本指南涵盖了Prompt的基础知识，提供关于如何使用提示来互动和指导大型语言模型（LLM）的粗略概念。

本章节主题包括：**基础Prompt有关大型语言模型的参数设置标准PromptPrompt所包含的元素**

# **基础Prompt**

你可以通过大型语言模型生成很多类型的内容，但是生成内容的质量取决于你提供给他多少信息。一个提示可以包含你给模型的指令或问题等，也可以包括更多细节如输入（指引输出的文字）或例子。

下面是一个例子：输入：

```
大海是
The sky is
```

输出：

```
大海是指由大量咸水组成的水体，具体来说是海水。广义上的海洋是指由地球海洋咸水水域组成的部分，一般把它看作一整个世界洋，或是由若干个大洋组成的水体。
blue
The sky is blue on a clear day. On a cloudy day, the sky may be gray or white.
```

正如你所看到的，语言模型输出了一个字符串，该输出是语言模型根据上下文预测得到的。这一输出通常是合理的，就像上面这个例子一样，但有可能是出乎我们意料的，或者与我们想要完成的任务相去甚远。这例子也说明了提供更多背景知识或说明我们具体想要实现什么是十分必要的。因此我们对上面这个输入Prompt做一些提升：输入：

```
请将这个句子补充完整：大海是
Complete the sentence:  The sky is
```

输出：

```
大海是我的故乡，让我感到心旷神怡。
The sky is  so  beautiful today.
```

是不是与之前的结果相去甚远？我们通过告诉模型要将这个句子补充完整，成功让其产生的结果更加精准，因为它通过联系上下文后遵循了我们的指令（"将这个句子补充完整"）。这种指示模型执行任务的方法就是所谓的Prompt工程。

这里有一些提示，对于你的Prompt工程有帮助。

- 你可以尝试其他简单的任务，使用简单的命令来指示模型，如 "写"、"分类"、"总结"、"翻译"、"排序 "等等。
- 请记住，你还需要进行大量的实验，以便看到什么是最有效的。用不同的关键词、背景知识和数据尝试不同的指令，看看什么对你的特定用例和任务最有效。通常情况下，上下文与你要执行的任务越具体、越相关，就越好。我们将在接下来的教程中涉及采样和添加更多背景知识的重要性。

我们将在本指南中涵盖更多这些能力，但也涵盖一些其他你可能感兴趣的领域，如进阶Prompt技术和围绕Prompt工程的研究课题。

# **简单介绍大型语言模型的参数设置**

在处理提示时，你将通过API或直接与LLM进行交互。你可以配置一些参数来让你的Prompt获得不同的结果。

- Temperature - 简而言之，Temperature越低，生成结果就越确定，即LLM总是会挑选可能性最高的下一个词（token）。提高Temperature可能会导致更多的随机性，鼓励更多不同或创造性的输出，调高这个参数本质上是在增加其他可能的词的权重，让LLM有更大的概率生成其它词。在应用方面，当我们想生成一些基于事实的高质量内容时可以使用较低的Temperature，以鼓励更多基于事实的简明的结果。对于诗歌生成或其他创造性任务，提高Temperature可能产生更加优秀的结果。
- Top_p - 同样地， top_p，这种带有temperature的采样技术称为核采样nucleus sampling，可以控制模型在生成内容时的确定性。如果你正在寻找准确的、符合事实的答案，那就把这个参数调低。如果你在寻找更多样化的反应，就增加到一个更高的值。
- Top_k - 有些大型语言模型中提供这个参数，与Top_p类似

一般的建议是对于上述3个参数，一次性最多改变其中的一个。

- Frequency penalty - 根据目前同一词（token）在文中出现频率惩罚相同的词（token）再次生成的力度。这个值越高，生成词句的变化越多，降低相同词出现的概率。
- Presence penalty - 与Frequency penalty类似，但是是根据目前同一词（token）在文中是否出现过来惩罚相同的词（token）再次生成的力度。这个值越高，越可能生成新的话题。

在开始讲一些基本的例子之前，请注意你生成的结果可能会因你使用的LLM的种类版本而与本文有所不同。

# **标准Prompt**

我们已经在上文尝试了一个非常简单的Prompt。一个标准的Prompt应该具有以下的格式：`<问题>？`你可以在此之上为它添加一个名为“QA格式”的格式，这是很多问答类数据集的格式，添加后如下：

```
Q: <问题>？
A:
```

一种流行的、有效的Prompt技术被称为 "few-shot prompting"，这种技术通过给LLM输入几组样例，并提示LLM模仿学习样例，从而输出我们想要的目标内容。基于上述的标准格式，在这里我们提供一个案例来演示这一种Prompt技术。Few-shot prompt可以采用以下格式。

```
<问题>？
<答案>

<问题>？
<答案>

<问题>？
<答案>

<问题>？
```

当你使用QA格式时，他们会看起来像这样：

```
Q: <问题>？
A: <答案>

Q: <问题>？
A: <答案>

Q: <问题>？
A: <答案>

Q: <问题>？
A:
```

请记住，这种技术并没有强制要求使用QA格式。格式应该取决于目标任务的类型。例如，你可以通过给出示如下范任务案例，来让他执行一个简单的分类任务，如下所示。

```
这真是太棒了！// 积极的
这很糟糕！// 负面的
哇，那部电影真不错！// 积极的
多么可怕的表演！//
```

输出：`负面的`

```
This is awesome! // Positive
This is bad! // Negative
Wow that movie was rad! // Positive
What a horrible show! //
```

输出：`Negative`

通过少量的提示就使语境学习成为可能，这是语言模型学习任务的能力，只需要给几个例子，它就能完成类似的任务。我们可以在接下来的教程中看到更多类似的行为。

# **Prompt所包含的元素**

随着我们涵盖越来越多的例子和Prompt工程可能的应用，你会注意到，有一些元素构成了提示。

一个提示可以包含以下任何组成部分：

- 指令 - 你希望模型执行的具体任务或指令
- 背景 - 可以涉及外部信息或额外的背景，可以引导模型做出更好的反应
- 输入数据 - 是我们感兴趣的输入或问题，希望为其寻找回答
- 输出指标 - 表示输出的类型或格式

并非所有的组成部分都是Prompt必需的，格式取决于任务目标。我们将在接下来的教程中谈到更多的具体例子。

# **2. 基础Prompting**

在前面的指南中，我们介绍并给出了一个提示的基本例子。

在本指南中，我们将提供更多关于如何使用提示的例子，并介绍许多对下一章高级Prompting很重要的关键概念。

通常情况下，学习概念的最好方法是通过观察例子。下面我们介绍几个例子，说明精心设计的提示语如何用于执行各种类的任务。

本章主题：

- 文本归纳
- 信息提取
- 问题回答
- 文本分类
- 对话
- 代码生成
- 推理

# **文本归纳**

自然语言生成中的一个标准任务是文本的归纳总结。文本总结可以覆盖许多不同的领域。事实上，语言模型最有前途的应用之一是能够将文章和概念总结成快速和易于阅读的摘要。让我们尝试一下使用提示语的基本总结任务。

假设我对了解抗生素感兴趣，我可以尝试这样的prompt。Prompt:

```
解释抗生素
Explain antibiotics
A:
```

输出:

```
抗生素是一种用于治疗细菌感染的药物。它们的作用是杀死细菌或防止它们繁殖，使身体的免疫系统能够抵御感染。抗生素通常以药片、胶囊或液体溶液的形式口服，有时也可通过静脉注射。它们对病毒感染无效，而且不适当地使用它们会导致抗生素耐药。
Antibiotics are a type of medication used to treat bacterial infections. They work by either killing the bacteria or preventing them from reproducing, allowing the body’s immune system to fight off the infection. Antibiotics are usually taken orally in the form of pills, capsules, or liquid solutions, or sometimes administered intravenously. They are not effective against viral infections, and using them inappropriately can lead to antibiotic resistance.
```

“A”: 是一种明确的提示格式，在想让LLM回答问题时使用。我在这里用它来告诉模型，有一个我们预期的下一步。在这个例子中，这个A的作用并不是很明显，但我们可以把这个问题留给后面的例子。让我们假设这个回答信息太多，想进一步让LLM为我们总结一下。我们可以把刚才输出的结果再输入给它，并指示模型将其总结为一句话，像这样：Prompt：

```
抗生素是一种用于治疗细菌感染的药物。它们的作用是杀死细菌或防止它们繁殖，使身体的免疫系统能够抵御感染。抗生素通常以药片、胶囊或液体溶液的形式口服，有时也可通过静脉注射。它们对病毒感染无效，而且不适当地使用它们会导致抗生素耐药。
把上面这段话用一句话说明清楚：
Antibiotics are a type of medication used to treat bacterial infections. They work by either killing the bacteria or preventing them from reproducing, allowing the body’s immune system to fight off the infection. Antibiotics are usually taken orally in the form of pills, capsules, or liquid solutions, or sometimes administered intravenously. They are not effective against viral infections, and using them inappropriately can lead to antibiotic resistance.
Explain the above in one sentence:
```

输出：

```
抗生素是用于治疗细菌感染的药物，通过杀死细菌或阻止它们繁殖，但它们对病毒无效，而且过度使用会导致抗生素耐药性。
Antibiotics are medications used to treat bacterial infections by either killing the bacteria or stopping them from reproducing, but they are not effective against viruses and overuse can lead to antibiotic resistance.
```

在你输入上述指令后，模型尝试只使用一句话来概括这段话，你不用太关心上面这段话的准确性的，这个问题我们将在以后的教程中涉及。你还可以对上面Prompt进行一些巧妙的处理，但我们把这个问题留给后面的章节。你也可以自己去试试各种不同的Prompt，看看你是否能得到更好的结果。

# **信息提取**

虽然语言模型被训练用来执行自然语言生成和相关任务，但它也能够以很高的水准执行分类和一系列其他自然语言处理（NLP）任务。

下面是一个让语言模型从给定段落中提取信息的例子。Prompt：

```
研究论文中的作者贡献声明和致谢应清楚而具体地说明作者在准备稿件和分析时是否使用了ChatGPT等人工智能技术，以及使用的程度如何。他们还应该指出使用了哪些LLMs。这将提醒编辑和审稿人更仔细地审查稿件，以发现潜在的偏见、不准确和不当的来源。同样地，科学期刊在使用LLM方面也应该透明，例如在选择提交的稿件时。
上面一段中提到的基于大型语言模型的产品是：
Author-contribution statements and acknowledgements in research papers should state clearly and specifically whether, and to what extent, the authors used AI technologies such as ChatGPT in the preparation of their manuscript and analysis. They should also indicate which LLMs were used. This will alert editors and reviewers to scrutinize manuscripts more carefully for potential biases, inaccuracies and improper source crediting. Likewise, scientific journals should be transparent about their use of LLMs, for example when selecting submitted manuscripts.
Mention the large language model based product mentioned in the paragraph above:
```

输出：

```
上面一段中提到的基于大型语言模型的产品是ChatGPT。
The large language model based product mentioned in the paragraph above is ChatGPT.
```

我们有很多方法可以改进上述结果，但这样的回答已经非常有用了。

现在你应该能很明显地发现，只要给模型一些简单的指示就能要求它执行不同的任务。这是一种强大的能力，许多组织已经在使用这种能力来建造强大的产品和体验。

输入的段落来源：**[ChatGPT：研究的五个重点](https://www.nature.com/articles/d41586-023-00288-7)**

# **问题回答**

让模型回答更加具体的答案的最好方法之一是改进Prompt的格式。如之前所述，一个Prompt可以结合指示、背景、输入和输出这些元素来改进模型的输出。虽然不是必须的组成部分，但它成为一个很好的做法，因为你的指示越具体，你将得到更好的结果。下面是一个例子，说明在给出一个更有条理的提示之后，能够获得怎么样的输出。Prompt：

```
根据以下背景信息回答问题。回答要简短扼要。如果对答案不确定，请回答 "不确定"。
背景信息：泰普利珠单抗的历史可以追溯到新泽西州一家名为Ortho Pharmaceutical的药物公司。在那里，科学家们产生了一种早期版本的抗体，被称为OKT3。该分子最初来自于小鼠，能够与T细胞表面结合并限制其细胞杀伤潜力。1986年，它被批准用于帮助防止肾脏移植后的器官排斥，使其成为第一个允许人类使用的治疗性抗体。
问题：OKT3最初的来源是什么？
答案：
Answer the question based on the context below. Keep the answer short and concise. Respond "Unsure about answer" if not sure about the answer.
Context: Teplizumab traces its roots to a New Jersey drug company called Ortho Pharmaceutical. There, scientists generated an early version of the antibody, dubbed OKT3. Originally sourced from mice, the molecule was able to bind to the surface of T cells and limit their cell-killing potential. In 1986, it was approved to help prevent organ rejection after kidney transplants, making it the first therapeutic antibody allowed for human use.
Question: What was OKT3 originally sourced from?
Answer:
```

输出:

```
老鼠。
Mice。
```

文本分类到目前为止，我们已经使用了简单的指令来让大型语言模型执行一些任务。作为一名提示工程师，你需要提供很好的指示，但这还不是全部！你还会发现，在一些难度较大的场景下，仅仅给模型提供指示是不够的。这时，你需要考虑给模型一些上下文或在提示中使用更多不同的类型的信息，如输入数据或例子。

让我们试着通过提供一个文本分类的例子来证明这一点。Prompt：

```
分辨下文中表达的情绪是中性的、负面的还是积极的。

文本：我认为食物还可以。
情绪：
Classify the text into neutral, negative or positive.

Text: I think the food was okay.
Sentiment:
```

输出：

```
中性
Neutral
```

我们给出了对文本进行分类的指令，模型的回答是 "**中性Neutral**"，这是正确的。这没有什么不对，但是我们给出的指示是将文字中的情绪分为**中性的neutral**、**负面的**和**积极的**三种，我们需要他的答案与我们给出的三个标签一模一样。因此，我们希望它返回**中性的neutral**，而不是**中性Neutral**。那我们要如何实现这一点呢？有不同的方法来做到这一点，但我们在这部分教程中关心的是Prompt的具体性，所以我们将提供更多的信息来做到这一点。现在我们将提供一个具体的例子来展示这一方法，让我们再试一次。Prompt：

```
分辨下文中表达的情绪是中性的、负面的还是积极的。

文本：我认为这个假期还不错。
情绪：中性的

文本：我认为食物还可以。我认为食物还可以。
情绪：
Classify the text into neutral, negative or positive.

Text: I think the vacation is okay.
Sentiment: neutral

Text: I think the food was okay.
Sentiment:
```

输出：

```
中性的
neutral
```

这一次，模型返回了**中性的neutral**，这正是我需要的完全一致的标签。说明提示中提供的例子帮助模型输出更贴合我们需求的答案。为了强调为什么有时让你的需求变得具体是非常重要的，请你查看下面例子并找出问题。Prompt：

```
分辨下文中表达的情绪是营养的、负面的还是积极的。

文本：我认为这个假期还不错。
情绪：
Classify the text into nutral, negative or positive.

Text: I think the vacation is okay.
Sentiment:
```

输出：

```
中性
Neutral
```

这个例子有什么问题？(如果你要求的任务是不常见的、不符合模型预测的，它会直接生成完全不同的、符合它预测的内容)

# **对话**

也许你可以通过Prompt工程实现的更有趣的事情之一是告诉模型它应该如何表现，其意图和身份是什么。这在你建立对话系统时特别有用。

例如，让我们创建一个对话系统，能够对问题做出更多技术性的和科学的回答。注意我们是如何通过指令明确地告诉它如何行为的。Prompt：

```
以下是与一位人工智能研究助理的对话。助理的回答是技术性且科学的。

人类：你好，你是谁？
人工智能：你好! 我是一名人工智能研究助理。今天我能为您提供什么帮助？
人类：你能告诉我黑洞是如何产生的吗？
人工智能：
The following is a conversation with an AI research assistant. The assistant tone is technical and scientific.

Human: Hello, who are you?
AI: Greeting! I am an AI research assistant. How can I help you today?
Human: Can you tell me about the creation of blackholes?
AI:
```

回答：

```
当然！黑洞是时空的区域，其引力如此之强，以至于没有任何东西，甚至是光，可以从其中逃脱。它们是在一个非常大的恒星死亡后，其核心向自身塌陷，形成一个密度无限的奇点时产生的。奇点的强大引力将其周围的所有物质和辐射拉入其中，形成黑洞。
Sure! Black holes are regions of spacetime where the gravitational force is so strong that nothing, not even light, can escape from it. They are created when a very massive star dies and its core collapses in on itself, forming a singularity of infinite density. The intense gravity of the singularity pulls in all the matter and radiation around it, creating the black hole.
```

我想我们已经取得了一些进展，你也可以继续改进它。我相信如果你增加更多的例子，你可能会得到更好的结果。

# **代码生成**

LLMs相当有价值的一个应用是代码生成，比如Copilot就是这方面一个很好的例子。有大量的代码生成任务，你可以通过巧妙的提示来完成。让我们看一下下面的几个例子。

首先，让我们试试一个简单的程序，向用户问好。Prompt：

```
/*
询问用户的姓名并说 "你好"
*/
/*
Ask the user for their name and say "Hello"
*/
```

输出：

```
let name = prompt("What is your name?");
console.log(`Hello, ${name}!`);
```

你可以看到，我们甚至不需要指定要使用的语言。

那么接下来让我们增加一些难度。我想向你展示，只要在设计提示语上多下点功夫，LLM就能变得多么强大。Prompt：

```
"""
Table departments, columns = [DepartmentId, DepartmentName]
Table students, columns = [DepartmentId, StudentId, StudentName]
为计算机科学系的所有学生创建一个MySQL查询
"""
"""
Table departments, columns = [DepartmentId, DepartmentName]
Table students, columns = [DepartmentId, StudentId, StudentName]
Create a MySQL query for all students in the Computer Science Department
"""
```

输出：

```
SELECT StudentId, StudentName
FROM students
WHERE DepartmentId IN (SELECT DepartmentId FROM departments WHERE DepartmentName = 'Computer Science');
```

这是非常令人印象深刻的。在这种情况下，我们提供了关于数据库模式的数据，并要求它生成一个有效的MySQL查询。

# **推理**

对于今天的LLM来说，最困难的任务之一也许是需要某种形式的推理。推理是我最兴奋的领域之一，因为这能帮助LLM完成的复杂应用类型。

目前LLM在涉及数学能力的任务上已经有了一些改进。尽管如此，你仍需要注意，目前的LLM在执行推理任务时很吃力，所以这需要更加先进的Prompt技术。我们将在下一篇教程中介绍这些高级技术。现在，我们只会介绍几个基本的例子来展示算术能力。Prompt：

```
9,000*9,000是多少？
What is 9,000 * 9,000?
```

输出：`81,000,000`

让我们试着提升一些难度Prompt：

```
这一组数字中的奇数相加是一个偶数：15，32，5，13，82，7，1。
A:
The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1.
A:
```

输出：

```
不，这一组数字中的奇数相加是一个奇数：119。
No, the odd numbers in this group add up to an odd number: 119.
```

很明显，模型在这里出错了，答案应该是41。让我们试着提升一下我们的Prompt。Prompt：

```
这一组数字中的奇数相加是一个偶数：15，32，5，13，82，7，1。
把这个问题分成几个步骤来解决。首先，找出这组数字中的奇数，然后将它们相加，并指出结果是奇数还是偶数。
The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1.
Solve by breaking the problem into steps. First, identify the odd numbers, add them, and indicate whether the result is odd or even.
```

输出：

```
奇数：15, 5, 13, 7, 1
相加的和：41
41是一个奇数。
Odd numbers: 15, 5, 13, 7, 1
Sum: 41
41 is an odd number.
```

这就好多了对吧？顺便说一下，我试了好几次，发现模型有时还是会出错。如果你提供一个更好的指导，结合实例，可能可以获得更准确的结果。

在下一节教程中，我们将涵盖更多的高级Prompt工程概念，以提高本章提到的任务，以及更困难的任务的性能。

# 3. Prompt工程-进阶

看到现在，你已经可以很明显的感受到改进prompt可以帮助在不同的任务上获得更好的结果。这就是prompt工程背后的整个想法。

虽然这些例子很有趣，但在我们深入了解更高级概念之前，让我们先介绍一些概念。

主题：

- Zero-shot Prompt
- Few-shot Prompt
- Chain-of-Thought Prompt
- Zero-shot CoT（思维链）
- Self-Consistency
- 生成知识的Prompt
- 自动Prompt工程师

---

## Zero-Shot（零样本） Prompts

今天的LLMs接受了大量数据的训练，并调整了Prompt，使它们能够进行zero-shot任务。我们实际上在前一节中尝试了一些zero-shot示例。以下是我们之前使用过的一个示例：

*Prompt:*

```jsx
分辨下文中表达的情绪是中性的、负面的还是积极的。

文本：我认为这个假期还不错。
情绪：
Classify the text into neutral, negative or positive. 

Text: I think the vacation is okay.
Sentiment:
```

输出：

```jsx
中性
Neutral
```

请注意，上面的提示中我们没有为模型提供任何例子——这就是零样本能力在发挥作用。当零样本无法发挥作用时，建议在提示中提供演示或示例。以下我们将讨论被称为few-shot提示的方法。

## Few-Shot（少样本） Prompts

在深入学习更高级的概念之前，我们先回顾一下使用few-shot prompts的示例。

还记得我们先前的示例吗：

```jsx
这一组数字中的奇数相加是一个偶数：15，32，5，13，82，7，1。
A:

The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1. 
A:
```

如果我们再试一次，模型会输出以下内容：

```jsx
是的，这一组的奇数加起来是107，这是一个偶数。
Yes, the odd numbers in this group add up to 107, which is an even number.
```

同样，这不是正确的回答，这不仅突出了这些系统的局限性，而且说明了需要更先进的提示工程。

让我们试着添加一些例子，看看这是否能改善结果。

*Prompt:*

```jsx
这一组的奇数加起来是一个偶数：4，8，9，15，12，2，1。
答：答案是错的。

这一组的奇数加起来是一个偶数：17，10，19，4，8，12，24。
答：答案为正确的。

这一组的奇数加起来是一个偶数：16，11，14，4，8，13，24。
答：答案是正确的。

这一组的奇数加起来是一个偶数：17，9，10，12，13，4，2。
答：答案是错的。

这一组的奇数加起来是一个偶数：15，32，5，13，82，7，1。
A:

The odd numbers in this group add up to an even number: 4, 8, 9, 15, 12, 2, 1.
A: The answer is False.

The odd numbers in this group add up to an even number: 17,  10, 19, 4, 8, 12, 24.
A: The answer is True.

The odd numbers in this group add up to an even number: 16,  11, 14, 4, 8, 13, 24.
A: The answer is True.

The odd numbers in this group add up to an even number: 17,  9, 10, 12, 13, 4, 2.
A: The answer is False.

The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1. 
A:
```

*Output:*

```jsx
答案是正确的
The answer is True.
```

这并不奏效。看来最基本的标准Prompt不足以让这种类型的推理问题得到可靠的回应。上面的例子提供了任务的基本信息，甚至还有例子。如果你仔细看一下这个任务，它确实涉及到更多的推理步骤。

根据[Min et al. (2022)](https://arxiv.org/abs/2202.12837)的研究结果（https://arxiv.org/abs/2202.12837），在进行少样本学习时，以下是一些关于示例的技巧：

- "标签空间和演示所指定的输入文本的分布都是关键（不管标签对个别输入是否正确）”
    - 解释：示范的贴标方式以及示范中的输入分布与目标任务的相似性是few-shot learning中的重要因素。即使演示中的标签对每个输入都不正确，拥有一个与目标任务相似的标签空间也能帮助模型更好地学习。
- 你使用的格式对性能也起着关键作用，即使你只是使用随机标签，这也比没有标签好得多。
    - 在演示中使用某种形式的标签比完全没有标签要好，即使标签是随机分配的。这是因为标签为任务目标提供了一些指导。
- 额外的结果显示，从标签的真实分布（而不是均匀分布）中选择随机标签也有帮助。
    - 从标签的真实分布中随机选择标签（而不是使用统一分布）可以进一步提高few-shot learning的性能。这意味着演示中的标签应该从与目标任务中的标签分布相似的分布中取样，因为这可以帮助模型学习更好的表征，提高其对新输入的概括能力。
- "the label space and the distribution of the input text specified by the demonstrations are both key (regardless of whether the labels are correct for individual inputs)"
- the format you use also plays a key role in performance, even if you just use random labels, this is much better than no labels at all.
- additional results show that selecting random labels from a true distribution of labels (instead of a uniform distribution) also helps.

> 上面三条有关技巧的内容比较难以翻译和理解，因此附上我写的详细说明以及英语原文
> 

让我们看几个例子。首先我们来尝试一个随机标签的例子（意思是`负面的`和`积极的`这两个标签是随机分配给输入的）：

*Prompt:*

```jsx
这真是太棒了! // 负面的
这很糟糕! // 积极的
哇，这电影真不错! // 积极的
多么可怕的表演! //
This is awesome! // Negative
This is bad! // Positive
Wow that movie was rad! // Positive
What a horrible show! //
```

*Output:*

```jsx
负面的
Negative
```

我们仍然得到了正确的答案，尽管标签已经被随机化了。请注意，我们还保留了格式，这也有帮助。事实上，随着进一步的实验，我们正在试验的较新的GPT模型似乎在处理随机的格式时也表现得非常不错，例子如下：

*Prompt:*

```jsx
积极的 这真是太棒了! 
这很糟糕! 负面的
哇，这电影真不错!
积极的
多么可怕的表演! --
Positive This is awesome! 
This is bad! Negative
Wow that movie was rad!
Positive
What a horrible show! --
```

*Output:*

```jsx
负面的
Negative
```

上面的格式并不一致，但模型仍然预测了正确的标签。我们必须进行更彻底的分析，以确认这对不同的、更复杂的任务，包括不同的Prompt变化是否成立。

总的来说，提供例子似乎是有用的。当zero-shot prompt和few-shot prompt都不能引导模型完成任务时，这可能意味着模型学到的任何东西都不足以在当前任务中做得更好。在这种情况下，建议你可以考虑微调你自己的模型来完成这个任务。

最近，思维链（CoT）的Prompting方法已经被普及，以解决更复杂的算术、常识和符号推理任务。所以我们接下来谈谈CoT，看看我们是否能解决上述任务之一。

## Chain-of-Thought Prompting

这个概念在[Wei et al. (2022)](https://arxiv.org/abs/2201.11903)的文章中首次出现，思维链（CoT）Prompt通过中间推理步骤实现复杂推理能力。你可以把它与few-shot prompt结合起来，在需要先推理再回答的更复杂的任务上获得更好的结果。

*Prompt:*

```jsx
这一组中的奇数加起来是一个偶数：4，8，9，15，12，2，1。
答：将所有的奇数（9，15，1）相加得到25。答案是错误的。

这一组的奇数加起来是一个偶数：17，10，19，4，8，12，24。
答：将所有的奇数（17，19）相加得到36。答案是正确的。

这一组的奇数加起来是一个偶数：16，11，14，4，8，13，24。
答：将所有的奇数（11，13）相加得出24。答案是正确的。

这一组的奇数加起来是一个偶数：17，9，10，12，13，4，2。
答：将所有的奇数（17，9，13）相加得到39。答案是错误的。

这一组的奇数加起来是一个偶数：15，32，5，13，82，7，1。
A:

The odd numbers in this group add up to an even number: 4, 8, 9, 15, 12, 2, 1.
A: Adding all the odd numbers (9, 15, 1) gives 25. The answer is False.

The odd numbers in this group add up to an even number: 17,  10, 19, 4, 8, 12, 24.
A: Adding all the odd numbers (17, 19) gives 36. The answer is True.

The odd numbers in this group add up to an even number: 16,  11, 14, 4, 8, 13, 24.
A: Adding all the odd numbers (11, 13) gives 24. The answer is True.

The odd numbers in this group add up to an even number: 17,  9, 10, 12, 13, 4, 2.
A: Adding all the odd numbers (17, 9, 13) gives 39. The answer is False.

The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1. 
A:
```

*Output:*

```jsx
将所有的奇数（15，5，13，7，1）相加得出41。答案是错误的。
Adding all the odd numbers (15, 5, 13, 7, 1) gives 41. The answer is False.
```

我们可以看到，当我们提供了推理步骤后，就有了完美的结果。事实上，我们可以通过提供更少的例子来解决这个任务，也就是说，在这个问题上只要一个例子似乎就足够了：

*Prompt:*

```jsx
The odd numbers in this group add up to an even number: 4, 8, 9, 15, 12, 2, 1.
A: Adding all the odd numbers (9, 15, 1) gives 25. The answer is False.

The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1. 
A:
```

*Output:*

```jsx
Adding all the odd numbers (15, 5, 13, 7, 1) gives 41. The answer is False.
```

请注意，这篇文章的作者声称这是一种只有在足够大的语言模型中才会出现的能力。

## Zero-Shot CoT

最近出现的一个想法是zero-shot CoT [(Kojima et al. 2022)](https://arxiv.org/abs/2205.11916)，它基本上是在原来普通的Prompt中加入 "让我们一步一步地思考/Let's think step by step"这句话。让我们尝试一个简单的问题，看看模型的表现如何。

*Prompt:*

```jsx
我去了市场，买了10个苹果。我把两个苹果给了邻居，两个给了修理工。然后我又去买了5个苹果，吃了一个。我还剩下多少个苹果？
I went to the market and bought 10 apples. I gave 2 apples to the neighbor and 2 to the repairman. I then went and bought 5 more apples and ate 1. How many apples did I remain with?
```

*Output:*

```jsx
11个苹果
11 apples
```

答案是不正确的! 现在让我们用刚才提到的Prompting方法来试试。

*Prompt:*

```jsx
我去了市场，买了10个苹果。我把两个苹果给了邻居，两个给了修理工。然后我又去买了5个苹果，吃了一个。我还剩下多少个苹果？

让我们一步一步地思考。
I went to the market and bought 10 apples. I gave 2 apples to the neighbor and 2 to the repairman. I then went and bought 5 more apples and ate 1. How many apples did I remain with?

Let's think step by step.
```

*Output:*

```jsx
首先，你开始有10个苹果。
你送了2个苹果给邻居，2个给修理工，所以你还有6个苹果。
然后你又买了5个苹果，所以现在你有11个苹果。
最后，你吃了一个苹果，所以你还剩下10个苹果。
First, you started with 10 apples.
You gave away 2 apples to the neighbor and 2 to the repairman, so you had 6 apples left.
Then you bought 5 more apples, so now you had 11 apples.
Finally, you ate 1 apple, so you would remain with 10 apples.
```

如此简单的提示却能有效地完成这项任务，让人感到十分印象深刻。在你没有太多的例子可以在Prompt中使用时，这个方法尤其有用。

## Self-Consistency

有一种也许是更先进的Prompt工程技术是Self-Consistency。由[[Wang et al. (2022)]](https://arxiv.org/pdf/2203.11171.pdf)提出，Self-Consistency的目标是“替换链式思考提示中使用的天真贪婪的解码”。其思想是通过少量样本的链式思考提示来采样多条不同的推理路径，并使用生成结果来选择最一致的答案。这有助于提高在算术和常识推理等任务中使用CoT Prompt的性能。

让我们试试下面这个算术推理的例子。

*Prompt:*

```jsx
当我6岁的时候，我的妹妹只有我一半的年龄。现在
我已经70岁了，我妹妹多大了？
When I was 6 my sister was half my age. Now
I’m 70 how old is my sister?
```

*Output:*

```jsx
35
```

输出结果显然是错误的。我们可能如何用Self-Consistency来改善这个问题呢？让我们来试试吧。我们将使用[[Wang et al. (2022)]](https://arxiv.org/pdf/2203.11171.pdf)文章中的few-shot示例（表17）。

*Prompt:*

```jsx
问：小树林里有15棵树。树林工人今天将在树林里种树。他们完成后。
将有21棵树。林场工人今天种了多少棵树？
答：我们开始有15棵树。后来我们有21棵树。差额一定是他们种的树的数量。
所以，他们一定是种了21-15=6棵树。答案是6。

问：如果停车场有3辆车，又有2辆车到达，那么停车场有多少辆车？
答：停车场里已经有3辆汽车。又有2辆到达。现在有3+2=5辆车。答案是5。

问：利亚有32块巧克力，她姐姐有42块。如果她们吃了35块，她们总共还剩下多少块？
答：Leah有32块巧克力，Leah的姐姐有42块。这意味着原来有32+42=74块
巧克力。35块已经被吃掉了。所以他们总共还有74-35=39块巧克力。答案是39。

问：杰森有20根棒棒糖。他给了丹尼一些棒棒糖。现在杰森有12根棒棒糖。杰森给了丹尼多少个棒棒糖？
赠给了丹尼？
答：杰森有20根棒棒糖。因为他现在只有12个，所以他肯定把剩下的给了丹尼。他给丹尼的棒棒糖的数量是
他给丹尼的棒棒糖数量一定是20-12=8个棒棒糖。答案是8。

问：肖恩有五个玩具。圣诞节时，他从他的妈妈和爸爸那里各得到两个玩具。他现在有多少个玩具？
他现在有多少个玩具？
答：他有5个玩具。他从妈妈那里得到了2个，所以之后他有5+2=7个玩具。然后他又从爸爸那里得到了2个，所以
所以他总共有7+2=9个玩具。答案是9。

问：机房里有九台电脑。从星期一到星期四，每天都有五台电脑被安装。
星期一到星期四。现在机房里有多少台电脑？
答：从星期一到星期四有4天。每天增加5台电脑。这意味着总共4*5=
增加了20台电脑。一开始有9台电脑，所以现在有9+20=29台电脑。
答案是29。

问：迈克尔有58个高尔夫球。在星期二，他丢了23个高尔夫球。星期三，他又丢了两个。他最后有多少个
他在周三结束时有多少个高尔夫球？
答：迈克尔最初有58个球。周二他丢了23个，所以之后他有58-23=35个球。在
周三他又丢了2个，所以现在他有35-2=33个球。答案是33。

问：奥利维亚有23美元，她买了五个面包圈，每个3美元。她还剩下多少钱？
答：她买了5个面包圈，每个3美元。这意味着她花了5

问：当我6岁时，我的姐姐是我年龄的一半。现在我70岁了，我姐姐多大了？
A:

Q: There are 15 trees in the grove. Grove workers will plant trees in the grove today. After they are done,
there will be 21 trees. How many trees did the grove workers plant today?
A: We start with 15 trees. Later we have 21 trees. The difference must be the number of trees they planted.
So, they must have planted 21 - 15 = 6 trees. The answer is 6.

Q: If there are 3 cars in the parking lot and 2 more cars arrive, how many cars are in the parking lot?
A: There are 3 cars in the parking lot already. 2 more arrive. Now there are 3 + 2 = 5 cars. The answer is 5.

Q: Leah had 32 chocolates and her sister had 42. If they ate 35, how many pieces do they have left in total?
A: Leah had 32 chocolates and Leah’s sister had 42. That means there were originally 32 + 42 = 74
chocolates. 35 have been eaten. So in total they still have 74 - 35 = 39 chocolates. The answer is 39.

Q: Jason had 20 lollipops. He gave Denny some lollipops. Now Jason has 12 lollipops. How many lollipops
did Jason give to Denny?
A: Jason had 20 lollipops. Since he only has 12 now, he must have given the rest to Denny. The number of
lollipops he has given to Denny must have been 20 - 12 = 8 lollipops. The answer is 8.

Q: Shawn has five toys. For Christmas, he got two toys each from his mom and dad. How many toys does
he have now?
A: He has 5 toys. He got 2 from mom, so after that he has 5 + 2 = 7 toys. Then he got 2 more from dad, so
in total he has 7 + 2 = 9 toys. The answer is 9.

Q: There were nine computers in the server room. Five more computers were installed each day, from
monday to thursday. How many computers are now in the server room?
A: There are 4 days from monday to thursday. 5 computers were added each day. That means in total 4 * 5 =
20 computers were added. There were 9 computers in the beginning, so now there are 9 + 20 = 29 computers.
The answer is 29.

Q: Michael had 58 golf balls. On tuesday, he lost 23 golf balls. On wednesday, he lost 2 more. How many
golf balls did he have at the end of wednesday?
A: Michael initially had 58 balls. He lost 23 on Tuesday, so after that he has 58 - 23 = 35 balls. On
Wednesday he lost 2 more so now he has 35 - 2 = 33 balls. The answer is 33.

Q: Olivia has $23. She bought five bagels for $3 each. How much money does she have left?
A: She bought 5 bagels for $3 each. This means she spent 5

Q: When I was 6 my sister was half my age. Now I’m 70 how old is my sister?
A:
```

*Output 1:*

```jsx
当我6岁时，我妹妹是我年龄的一半，所以她是3岁。现在我70岁了，所以她是70-3=67。答案是67。
When I was 6 my sister was half my age, so she was 3. Now I am 70, so she is 70 - 3 = 67. The answer is 67.
```

*Output 2:*

```jsx
当叙述者6岁时，他的妹妹是他年龄的一半，也就是3岁。现在叙述者70岁了，他的妹妹应该是70-3=67岁。答案是67岁。
When the narrator was 6, his sister was half his age, which is 3. Now that the narrator is 70, his sister would be 70 - 3 = 67 years old. The answer is 67.
```

*Output 3:*

```jsx
当我6岁时，我妹妹是我年龄的一半，所以她是3岁。现在我70岁了，所以她是70/2=35。答案是35。
When I was 6 my sister was half my age, so she was 3. Now I am 70, so she is 70/2 = 35. The answer is 35.
```

计算最终答案涉及几个步骤（详情请查阅论文），但为了简单起见，我们可以清楚地看到，已经有一个主要的答案出现，所以这基本上会成为最终答案。

## Generated Knowledge Prompting 生成知识的Prompt

想要继续改进LLMs，其中一个流行的技术是通过纳入知识或信息来帮助模型做出更准确的预测。

利用类似的想法，模型是否也可以自在做出预测之前自己生成有帮助的知识？这就是[Liu et al. 2022](https://arxiv.org/pdf/2110.08387.pdf)的论文中所尝试的 -- 让模型生成知识来作为Prompt的一部分。我们想要知道，这对常识性推理等任务有多大帮助？

让我们试试一个简单的提示。

*Prompt:*

```jsx
高尔夫的一部分规则是试图获得比别人更高的总分，是或不是？
Part of golf is trying to get a higher point total than others. Yes or No?
```

*Output:*

```jsx
是
Yes.
```

这类错误揭示了LLM在执行一些需要更多显示世界知识的任务时具有局限性。我们如何用知识生成来改善这一点？

首先，让我们先尝试生成一些 "知识"：

*Prompt:*

```jsx
输入：希腊比墨西哥大。
知识：希腊大约有131,957平方公里，而墨西哥大约有1,964,375平方公里，所以墨西哥比希腊大1,389%。

输入：眼镜总是起雾。
知识点：当来自你的汗水、呼吸和环境湿度的水蒸气落在寒冷的表面上，冷却后变成微小的液滴，形成一层薄膜，你看到的就是雾，这时眼镜片上就会发生凝结。与你的呼吸相比，你的镜片会相对凉爽，特别是当外面的空气很冷时。

输入：鱼是能够思考的。
知识：鱼比它们看起来更聪明。在许多方面，如记忆，它们的认知能力与包括非人类灵长类在内的 "高等 "脊椎动物相匹配或超过。鱼的长期记忆有助于它们跟踪复杂的社会关系。

输入：一个人一生中抽了很多烟，一个常见的影响是患肺癌的机会比正常人高。
知识：那些在一生中平均每天持续少于一支烟的人，死于肺癌的风险是从未吸烟者的九倍。在每天抽1到10支烟的人中，死于肺癌的风险比从不吸烟的人高近12倍。

输入：一块石头与一块鹅卵石大小相同。
知识：卵石是一种岩石碎块，根据Udden-Wentworth的沉积学尺度，其颗粒大小为4至64毫米。通常认为卵石比颗粒（直径2至4毫米）大，比鹅卵石（直径64至256毫米）小。

输入：高尔夫的一部分是试图获得比别人更高的总分。
知识：
Input: Greece is larger than mexico.
Knowledge: Greece is approximately 131,957 sq km, while Mexico is approximately 1,964,375 sq km, making Mexico 1,389% larger than Greece.

Input: Glasses always fog up.
Knowledge: Condensation occurs on eyeglass lenses when water vapor from your sweat, breath, and ambient humidity lands on a cold surface, cools, and then changes into tiny drops of liquid, forming a film that you see as fog. Your lenses will be relatively cool compared to your breath, especially when the outside air is cold.

Input: A fish is capable of thinking.
Knowledge: Fish are more intelligent than they appear. In many areas, such as memory, their cognitive powers match or exceed those of ’higher’ vertebrates including non-human primates. Fish’s long-term memories help them keep track of complex social relationships.

Input: A common effect of smoking lots of cigarettes in one’s lifetime is a higher than normal chance of getting lung cancer.
Knowledge: Those who consistently averaged less than one cigarette per day over their lifetime had nine times the risk of dying from lung cancer than never smokers. Among people who smoked between one and 10 cigarettes per day, the risk of dying from lung cancer was nearly 12 times higher than that of never smokers.

Input: A rock is the same size as a pebble.
Knowledge: A pebble is a clast of rock with a particle size of 4 to 64 millimetres based on the Udden-Wentworth scale of sedimentology. Pebbles are generally considered larger than granules (2 to 4 millimetres diameter) and smaller than cobbles (64 to 256 millimetres diameter).

Input: Part of golf is trying to get a higher point total than others.
Knowledge:
```

*Knowledge 1:*

```jsx
高尔夫的目标是用最少的杆数打完一组球洞。一轮高尔夫比赛通常由18个洞组成。在标准的高尔夫球场上，每个洞都要打一次。每一杆计为一分，总杆数用来决定比赛的胜负。
The objective of golf is to play a set of holes in the least number of strokes. A round of golf typically consists of 18 holes. Each hole is played once in the round on a standard golf course. Each stroke is counted as one point, and the total number of strokes is used to determine the winner of the game.
```

*Knowledge 2:*

```jsx
高尔夫是一项精确的球杆和球类运动，参赛者（或高尔夫球手）使用多种类型的球杆，用最少的杆数将球打入球场上的一系列球洞。目标是以最低的分数完成比赛，分数的计算方法是将每个洞的总杆数相加。得分最低的选手赢得比赛。
Golf is a precision club-and-ball sport in which competing players (or golfers) use many types of clubs to hit balls into a series of holes on a course using the fewest number of strokes. The goal is to complete the course with the lowest score, which is calculated by adding up the total number of strokes taken on each hole. The player with the lowest score wins the game.
```

我们使用的是[Liu et al. 2022](https://arxiv.org/pdf/2110.08387.pdf)论文中提供的提示。

下一步是整合知识，获得我们需要的预测。我将问题转化为为QA格式，来引导答案的格式。

*Prompt:*

```jsx
问题：高尔夫的一部分是试图获得比别人更高的总分。是或不是？

知识：高尔夫的目标是用最少的杆数打完一组洞。一轮高尔夫比赛通常由18个洞组成。在一个标准的高尔夫球场上，每一个洞都要打一次。每一杆计为一分，用总杆数来决定比赛的胜负。

解释和回答：
Question: Part of golf is trying to get a higher point total than others. Yes or No?

Knowledge: The objective of golf is to play a set of holes in the least number of strokes. A round of golf typically consists of 18 holes. Each hole is played once in the round on a standard golf course. Each stroke is counted as one point, and the total number of strokes is used to determine the winner of the game.

Explain and Answer:
```

*Answer 1 (非常自信):*

```jsx
不，高尔夫的目标不是要比别人获得更高的总分。相反，目标是用最少的杆数打完一组洞。总杆数是用来决定比赛的赢家，而不是总分。
No, the objective of golf is not to get a higher point total than others. Rather, the objective is to play a set of holes in the least number of strokes. The total number of strokes is used to determine the winner of the game, not the total number of points.
```

```jsx
问题：高尔夫的一部分是试图获得比别人更高的总分。是或不是？
知识：高尔夫是一项精确的球杆和球类运动，参赛者（或高尔夫球手）使用多种球杆，用最少的杆数将球打进球场上的一系列球洞。目标是以最低的分数完成比赛，分数的计算方法是将每个洞的总杆数相加。得分最低的选手赢得比赛。
解释和回答：
Question: Part of golf is trying to get a higher point total than others. Yes or No?
Knowledge: Golf is a precision club-and-ball sport in which competing players (or golfers) use many types of clubs to hit balls into a series of holes on a course using the fewest number of strokes. The goal is to complete the course with the lowest score, which is calculated by adding up the total number of strokes taken on each hole. The player with the lowest score wins the game.
Explain and Answer:
```

*Answer 2 (没有那么自信):*

```jsx
是的，高尔夫的一部分是试图获得比别人更高的总分。每个选手都试图以最低的分数完成比赛，分数的计算方法是将每个洞的总杆数相加。得分最低的选手赢得比赛。
Yes, part of golf is trying to get a higher point total than others. Each player tries to complete the course with the lowest score, which is calculated by adding up the total number of strokes taken on each hole. The player with the lowest score wins the game.
```

这个例子中发生了一些非常有趣的事情。在第一个答案中，该模型非常自信，但在第二个答案中却不那么自信了。为了演示，我简化了这个过程，在得出最终答案的时候，还有一些细节需要考虑。详情请看论文中的内容。

## Automatic Prompt Engineer (APE) 自动Prompt工程

![https://github.com/dair-ai/Prompt-Engineering-Guide/raw/main/img/APE.png](https://github.com/dair-ai/Prompt-Engineering-Guide/raw/main/img/APE.png)

[Zhou et al., (2022)](https://arxiv.org/abs/2211.01910)提出了自动提示工程（APE）框架，用于自动生成和选择指令。该指令生成问题被视为自然语言合成，并使用大型语言模型（LLM）作为黑盒优化问题来生成和搜索候选解决方案。

第一步涉及一个大型语言模型（作为推理模型），该模型接收输出演示来为任务生成指令候选项。这些候选解决方案将指导搜索过程。然后，使用目标模型执行指令，然后根据计算的评估分数选择最合适的指令。

APE发现了一个比人工设计的“让我们一步一步地思考”提示更好的零样本链式思考(Zero-shot CoT) Prompt，该Prompt来自 (Kojima et al., 2022)。

这个提示: “让我们一步一步地解决这个问题，确保我们有正确的答案。Let's work this out it a step by step to be sure we have the right answer.”引发了链式思考推理，并提高了在MultiArith和GSM8K基准测试中的性能。

![https://github.com/dair-ai/Prompt-Engineering-Guide/raw/main/img/ape-zero-shot-cot.png](https://github.com/dair-ai/Prompt-Engineering-Guide/raw/main/img/ape-zero-shot-cot.png)

本文涉及到与提示工程相关的一个重要主题，即自动优化提示的思想。虽然我们在本指南中没有深入探讨这个主题，但如果您对这个主题感兴趣，以下是一些关键论文：

[AutoPrompt](https://arxiv.org/abs/2010.15980) - 提出了一种基于梯度引导搜索(gradient-guided search)的方法，自动创建各种任务的提示。

[Prefix Tuning](https://arxiv.org/abs/2101.00190) - 是一种轻量级的fine-tuning替代方案，为自然语言生成任务准备了一个可训练的连续前缀。

[Prompt Tuning](https://arxiv.org/abs/2104.08691) - 提出了一种通过反向传播(back propagation)学习软提示的机制。

# Prompt应用

在本指南中，我们将介绍一些先进而有趣的方法，可以利用Prompt工程来执行有用且更高级的任务。

**请注意，本节仍在更新中**。主题包括：

- [Generating Data](https://github.com/dair-ai/Prompt-Engineering-Guide/blob/main/guides/prompts-applications.md#generating-data)
- [Code as Reasoning](https://github.com/dair-ai/Prompt-Engineering-Guide/blob/main/guides/prompts-applications.md#pal-program-aided-language-models-code-as-reasoning)
- ...

---

## 生成数据

大型语言模型（LLMs）具有强大的文本生成能力。使用有效的Prompt策略可以引导模型产生更好、更一致、更真实的响应。 LLMs还在生成数据方面特别有用，这对于运行各种实验非常有用。例如，我们可以使用它来为情感分类器生成一些快速样本，如下所示：

*Prompt:*

```jsx
产生10个用于情感分析的范例。例子被归类为积极或消极。产生2个负面例子和8个正面例子。使用这种格式的例子。
问：<句子>
答：<情绪>
Produce 10 exemplars for sentiment analysis. Examples are categorized as either positive or negative. Produce 2 negative examples and 8 positive examples. Use this format for the examples:
Q: <sentence>
A: <sentiment>
```

*Output:*

```jsx
问：我刚刚得到了一个最好的消息!
答：积极的

问：我们刚刚在工作中得到了加薪!
答：积极的

问：我为我今天的成就感到非常自豪。
答：积极的

问：我度过了有史以来最美好的一天!
答：积极的

问：我真的很期待周末的到来。
答：积极的

问：我刚刚收到了有史以来最好的礼物!
答：积极的

问：我现在很高兴。
答：积极的

问：我很庆幸有这样一个了不起的家庭。
答：积极的

问：外面的天气太阴沉了。
答：消极的

问：我刚刚得到一些可怕的消息。
答：消极的
Q: I just got the best news ever!
A: Positive

Q: We just got a raise at work!
A: Positive

Q: I'm so proud of what I accomplished today.
A: Positive

Q: I'm having the best day ever!
A: Positive

Q: I'm really looking forward to the weekend.
A: Positive

Q: I just got the best present ever!
A: Positive

Q: I'm so happy right now.
A: Positive

Q: I'm so blessed to have such an amazing family.
A: Positive

Q: The weather outside is so gloomy.
A: Negative

Q: I just got some terrible news.
A: Negative
```

这是非常有用的。实际上，我们将这个例子生成的内容用在了之前的教程里。

---

## PAL（程序辅助语言模型）：代码即推理

[Gao et al., (2022)](https://arxiv.org/abs/2211.10435)提出了一种使用LLM来读取自然语言问题并生成程序作为中间推理步骤的方法。称为程序辅助语言模型（PAL），它与思维链CoT提示的不同之处在于，它不是使用自由形式的文本来获得解决方案，而是将解决步骤转换成一个可以执行的程序，并将其传递给一个运行环境（例如Python解释器），后者执行该程序以产生最终解决方案。

![https://github.com/dair-ai/Prompt-Engineering-Guide/raw/main/img/pal.png](https://github.com/dair-ai/Prompt-Engineering-Guide/raw/main/img/pal.png)

让我们看看一个使用LangChain和OpenAI GPT-3的例子。我们想开发一个简单的应用程序，它能够通过利用Python解释器来解释所问的问题并提供答案。

具体来说，我们想创建一个功能，可以让LLM回答那些需要理解日期的问题。我们将为LLM提供一个Prompt，其中包括一些从[这里](https://github.com/reasoning-machines/pal/blob/main/pal/prompt/date_understanding_prompt.py)采用的范例。

这些是我们需要的导入语句：

```jsx
import openai
from datetime import datetime
from dateutil.relativedelta import relativedelta
import os
from langchain.llms import OpenAI
from dotenv import load_dotenv
```

让我们首先配置一些东西：

```jsx
load_dotenv()

# API configuration
openai.api_key = os.getenv("OPENAI_API_KEY")

# for LangChain
os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY")
```

设置模型实例：

```jsx
llm = OpenAI(model_name='text-davinci-003', temperature=0)
```

设置Prompt+问题：

```jsx
exec(llm_out)
print(born)
```
