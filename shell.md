## Introduction

本题要求你熟练使用Linux下Shell命令.

来自中世纪莱昂王国的阿方索国王由于处理宫廷事务心力憔悴，繁冗复杂的人脉关系让他失落沮丧，于是为了减少工作压力，更多的关注 家 庭 生 活 ，阿方索国王花费了重金（？）雇佣你来为他理清各个领主之间的关系。

但是光是伊比利亚地区的各个封臣都已经是恒河沙数了，要统计清整个欧洲的家族爵位关系显然是人力难为的，机智的你想到了利用计算机来帮阿方索国王解决他的问题。

由于当时的年代过于久远，没有c\c++\python之类的语言，你被推荐使用shell脚本来完成任务

而且因为阿方索国王是中世纪的人，对计算机那一套理论并不熟悉，所以你被期望开发出的脚本使用上亲民一点。

如果题目没有要求，你可以随意创建子文件夹, 但是请把所有Shell脚本放在同一个文件夹中以方便我们阅读。

## Hint

你可能用到的命令
touch mv ls rm uniq grep tar unzip sort split chmod ps ip base64 ifconfig find less more crontab which locate watch clear top vi echo nc
curl iptables xargs wget ssh git
你可能需要了解的知识
文件权限, 管道, 正则表达式，JSON

## Quest

* 为了方便你的编写，阿方索国王已经下令举国上下抄写了一份样表，存放在[github](https://github.com/ihopenot/uniqueshell)的仓库中。有需要可以取用。

* 每个人都有一个个人信息，这里使用 JSON 来描述。

  如例：

  ```json
  {
      "name":"Alfonso VI", //姓名
      "id": 1, //任意的不重复id
      "sex":"male", //性别
      "couples": [2, 3], //配偶 "Constance of Burgundy"与"Zaida of Seville"
      "family": "Jimena", //家族 家族都是独一无二的
      "children": [4, 5], //子嗣 "Urraca of León"与”Sancho Alfónsez“
      "maintitle": "Kingdom of León", //主头衔
      "titles": ["Kingdom of León", "Lord of Salamanca"] //所有头衔
  }
  ```

  这里只是一个示例，只要你保证脚本正常工作，任何结构的 JSON 都是可以接受的。（你可以随意增删元素，只要脚本正常工作）

* 首先，阿方索国王希望将每一个人的信息都存放在一个**数据文件夹**内。同时你的脚本应该能够支持

  * 迁移数据至另一个文件夹
  * 改变脚本的**数据文件夹**（这意味着如果之前脚本将个人信息存放在文件夹A下，之后将**数据文件夹**换成B，那么对于之后的所有操作里的个人信息均是B文件夹内的信息）
  * 备份数据并打包，可以指定路径，如果未指定，打包到当前目录下

  同时阿方索国王不希望自己手贱修改或删除掉某个文件铸成大错，你的脚本也应该支持

  * 每次运行脚本时校验文件相较上次结束脚本后是否变化，如果有变化，发出警告
  * 支持从备份的数据还原

  关于**数据文件夹**：为了个人信息安全，所有**数据文件夹**应该是仅所有者可读写的，如果改变**数据文件夹**操作的目标文件夹不满足要求，或者在某次脚本运行时发现当前**数据文件夹**不合规，脚本应该抛出警告。

* 其次，阿方索国王希望能够方便的导入和编辑个人信息，所以你的脚本应该能够支持

  * 从另一个文件夹导入数据
  * 从已经打包好的数据导入
  * 手动输入数据

  当然，导入任何数据都有可能导致和已有数据的冲突，为了简单起见，如果有冲突发生，脚本应该询问是否全部替换，导入数据不会和自己冲突，但是可能会重复。

* 首先，中世纪贵族非常看重血统，所以理清家族关系相当重要，由于整理过于麻烦，所以**并非所有人的个人信息都包含家族字段**，你的脚本应当支持

  * 查询某一个人属于哪个家族。（**所有孩子都跟随父亲的家族，这里不考虑入赘的情况**）如果无法得知这个人的所属家族，提示Unknown。

  * 将所有的人按家族分类导出到另一个文件夹，每个家族保存为一个文件，其中每一行包含了一个家族成员的 id 和 name，需要按id从小到大排序。

    例如：

    ```
    hh@hh:~# cat Jimena
    1 Alfonso VI
    7 blabla
    11 hahaha
    ```

    无法得知所属家族的人另外保存为一个文件 wildman

  * 阿方索国王意外的有点小八卦，所以他希望你也能统计出所有人中的私生子。为了简化问题，如果一个人的**个人信息中**的家族信息和他**应该**的家族信息（比如他父亲的家族）不同，那么我们认为这个人很有问题，~~（他爸被戴绿帽）~~，你需要将这些人同样按照每行id和name的格式保存，但是由于这些信息过于敏感，你应该先将它base64之后按行等分成10个文件后打包保存。

* 封建法中继承法是所有人关心的重点，因为这直接关系到他们死后领土的归属权，这里为了简单起见，我们假设所有地方实行的都是**男性优先均分继承法**，细化来说，只有被继承者没有儿子的情况下，女儿才有继承权，否则只有儿子有继承权，之后，在有继承权的人中最大的孩子继承被继承者的**主头衔**，其余的头衔依次由之后的继承者继承，如果头衔数多于继承者数，那么剩下的头衔在除开最大的继承者之后的继承者中继续依次分配。继承时父母头衔分别继承，互不影响。

  例如：

  ```
  被继承者拥有头衔（title）: [1, 2, 3, 4]
  主头衔为 1
  继承者 A B C
  其中A为长子
  那么1由A继承，
  之后B继承2，C继承3，B继承4
  不论有多少头衔，长子只会继承一个主头衔。
  每个人继承到的第一个头衔被认为是他的主头衔
  ```

  同样的，个人信息中关于头衔的继承也是不完善的，很多人没有titles这个字段，你的脚本需要支持

  * 查询某个人最终会继承到的头衔
  * 查询某个头衔最终会被谁继承到
  * 将所有头衔和最终继承到它们的人导出到另一个文件，格式为每行头衔名，继承者id和继承者name。需要按id从小到大排序。

* 最后，需要交互式的界面。