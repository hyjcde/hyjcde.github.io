---
layout: post
read_time: true
show_date: true
title:  Database Review 0
date:   2022-03-03 13:32:20 +0800
description: A brief review to database.
img: posts/20220304/db0.jpg
tags: [database, coding]
author: Hyjcde
github:  hyjcde/database-reviews/blob/main/DBRivision-0.md
mathjax: yes
---


# DBRiview-0 数据、数据库



[toc]



### 数据、数据库、数据库管理系统、数据库系统

> from wiki:In [computing](https://en.wikipedia.org/wiki/Computing), a **database** is an organized collection of [data](https://en.wikipedia.org/wiki/Data_(computing)) stored and accessed electronically. Small databases can be stored on a [file system](https://en.wikipedia.org/wiki/File_system), while large databases are hosted on [computer clusters](https://en.wikipedia.org/wiki/Computer_clusters) or [cloud storage](https://en.wikipedia.org/wiki/Cloud_storage). The [design of databases](https://en.wikipedia.org/wiki/Database_design) spans formal techniques and practical considerations including [data modeling](https://en.wikipedia.org/wiki/Data_modeling), efficient data representation and storage, [query languages](https://en.wikipedia.org/wiki/Query_language), [security](https://en.wikipedia.org/wiki/Database_security) and [privacy](https://en.wikipedia.org/wiki/Information_privacy) of sensitive data, and [distributed computing](https://en.wikipedia.org/wiki/Distributed_computing) issues including supporting [concurrent](https://en.wikipedia.org/wiki/Concurrent_computing) access and [fault tolerance](https://en.wikipedia.org/wiki/Fault_tolerance).
>
> A **[database management system](https://en.wikipedia.org/wiki/Database#Database_management_system)** (**DBMS**) is the [software](https://en.wikipedia.org/wiki/Software) that interacts with [end users](https://en.wikipedia.org/wiki/End_user), applications, and the database itself to capture and analyze the data. The DBMS software additionally encompasses the core facilities provided to administer the database. The sum total of the database, the DBMS and the associated applications can be referred to as a **database system.** Often the term "database" is also used loosely to refer to any of the DBMS, the database system or an application associated with the database.

##### 数据：

> 可以对数据做如下定义：**描述事物的符号记录**称为数据。描述事物的符号可以是数字，也可以是文字、图形、图像、音频、视频等，数据有多种表现形式，它们都可以经过数字化后存入计算机。

##### 数据库（Database）：

> 存放数据的仓库，这个仓库是在计算机存储设备上，而且数据是一定的格式存放的。 严格地讲，数据库是长期储存在计算机内、有组织的、可共享的大量数据的集合。数据库中的数据**按一定的数据模型组织、描述和储存**，（按照关系数据模型的为关系数据库）具有**较小的冗余度**(redundancy)、**较高的数据独立性**（data independency）和**易扩展性**（scalability），并可为各种用户共享。

##### 数据库管理系统（Database Management System）:

> ​	
>
> 一种**系统**软件，是数据库系统的核心。主要包含以下功能：
>
> 1. **数据定义功能**数据库管理系统提供数据定义语言（Data Definition Language，DDL)，用户通过它可以方便地对数据库中的数据对象的组成与结构进行定义。 
>
> 2.  **数据组织、存储和管理** 数据库管理系统要分类组织、存储和管理各种数据，包括数据字典、用户数据、数 的存取路径等。要确定以何种文件结构和存取方式在存储级上组织这些数据，如何实 覲数据之间的联系。数据组织和存储的基本目标是提高存储空间利用率和方便存取，提 哄多种存取方法（如索引查找、hash查找、顺序查找等）来提高存取效率。 
>
> 3. **数据操纵功能** 数据库管理系统还提供数据操纵语言（Data Manipulation Language，DML)，用户可以 使用它操纵数据，实现对数据库的基本操作，如查询、插入、删除和修改等。 
>
> 4. **数据库的事务管理和运行管理** 数据库在建立、运用和维护时由数据库管理系统统一管理和控制，以保证事务的正确 运行，保证数据的安全性、完整性、多用户对数据的并发使用及发生故障后的系统恢复。 
>
> 5. **数据库的建立和维护功能** 数据库的建立和维护功能包括数据库初始数据的输入、转换功能，数据库的转储、恢 夏功能，数据库的重组织功能和性能监视、分析功能等。这些功能通常是由一些实用程序 管理工具
>
>    

##### 数据库系统：

> 数据库系统是由数据库、数据库管理系统（及其应用开发工具）、应用程序和数据库 管理员（Database Administrator，DBA)组成的存储、管理、处理和维护数据的系统。在一般不引起混淆的情况下，人们常常把数据库系统简称为数据库。

### 文件系统与数据库系统

> 用**文件系统**管理数据具有如下特点： 
>
> 1. **数据可以长期保存** 由于计算机大量用于数据处理，数据需要长期保留在外存上反复进行查询、修改、插 入和删除等操作。 
> 2. **由文件系统管理数据** 由专门的软件即文件系统进行数据管理，文件系统把数据组织成相互独立的数据文 件，利用“按**文件名访问**`（考虑inode)`，按记录进行存取”的管理技术，提供了对文件进行打开与关闭、 对记录读取和写入等存取方式。文件系统实现了**记录内**的结构性。
> 3. 但是，文件系统仍存在以下缺点： 
>    1. **数据共享性差，冗余度大** 在文件系统中，一个（或一组）文件基本上对应于一个应用程序，即文件仍然是面向应用的。当不同的应用程序具有部分相同的数据时，也必须建立各自的文件，而不能共享 相同的数据，因此数据的冗余度大，浪费存储空间。同时由于相同数据的重复存储、各自 管理，容易造成数据的不一致性，给数据的修改和维护带来了困难。 
>    2. **数据独立性差** 文件系统中的文件是为某一特定应用服务的，文件的逻辑结构是针对具体的应用来设计和优化的，因此要想对文件中的数据再增加一些新的应用会很困难。而且，当数据的逻辑结构改变时，应用程序中文件结构的定义必须修改，应用程序中对数据的使用也要改变， 因此数据依赖于应用程序，缺乏独立性。可见，文件系统仍然是一个不具有弹性的无整体结构的数据集合，即文件之间是孤立的，不能反映现实世界事物之间的内在联系。
>
> 相对于文件系统而言，**数据库系统**具有以下特点：
>
> 1. **数据结构化** 数据库系统实现整体数据的结构化，这是数据库的主要特征之一，也是数据库系统与 文件系统的本质区别。 在文件系统中，文件中的记录内部具有结构，但是**记录的结构和记录之间的联系被固化在程序中**，需要由程序员加以维护。这种工作模式既加重了程序员的负担，又不利于结构的变动。 所谓“整体”结构化是指数据库中的数据不再仅仅针对某一个应用，而是面向整个组织或企业：不仅数据内部是结构化的，而且整体是结构化的，数据之间是**具有联系**的`考虑schema`。
> 2. **数据的共享性高、冗余度低且易扩充** 数据库系统从整体角度看待和描述数据，数据不再面向某个应用而是面向整个系统， 因此数据可以被多个用户、多个应用共享使用。数据共享可以大大减少数据冗余，节约存 储空间。数据共享还能够避免数据之间的不相容性与不一致性。 **所谓数据的不一致性是指同一数据不同副本的值不一样**`ACID`。采用人工管理或文件系统管理时，由于数据被重复存储，不同的应用不一致。在数据库中数据共享减少了由于数据冗余造成的不一致现象。 由于数据面向整个系统，是有结构的数据，不仅可以被多个应用共享使用，而且容易增加新的应用，这就使得数据库系统弹性大，易于扩充，可以适应各种用户的要求。可以 选取整体数据的各种子集用于不同的应用系统，当应用需求改变或增加时，只要重新选取不同的子集或加上一部分数据便可以满足新的需求。
> 3. **数据独立性高** 数据独立性是借助数据库管理数据的一个显著优点，`通常由DBMS的二级印象保证`。它己成为数据库领域中一个常用术语和重要概念，包括数据的**物理独立性**和**逻辑独立性**。 物理独立性是指用户的**应用程序**与数据库中数据的**物理存储**是相互独立的。也就是说，数据在数据库中怎样存储是山数据库管理系统管理的，用户程序不需要了解，应用程序要处理的只是数据的逻辑结构，这样当数据的物理存储改变时应用程序不用改变。 逻辑独立性是指用户的**应用程序**与**数据库的逻辑结构**是相互独立的。也就是说，数据 的逻辑结构改变时用户程序也可以不变。
> 4. **数据由数据库管理系统统一管理和控制** 数据库的共享将会带来数据库的安全隐患，而数据库的共享是**并发的**（concurrency）共 享，`（数据的一致性）`即多个用户可以同时存取数据库中的数据，甚至可以同时存取数据库中同一个数 据，这又会带来不同用户间相互干扰的隐患。另外，数据库中数据的正确与一致也必须得到保障。为此，数据库管理系统还必须提供以下几方面的数据控制功能。 
>    1. 数据的安全性（security）保护数据的安全性是指保护数据以防止不合法使用造成的数据泄密和破坏。每个用户只能 按规定对某些数据以某些方式进行使用和处理。  `grant or revoke--dcl`
>    2. 数据的完整性（integrity）检查 数据的完整性指数据的正确性、有效性和相容性。完整胜检查将数据控制在有效的范围内，并保证数据之间满足一定的关系。 
>    3. 并发（concurrency)控制 当多个用户的并发进程同时存取、修改数据库时，可能会发生相互干扰而得到错误的 结果或使得数据库的完整性遭到破坏，因此必须对多用户的并发操作加以控制和协调。  `by transaction`
>    4. 数据库恢复（recovery） 计算机系统的硬件故障、软件故障、操作员的失误以及故意破坏也会影响数据库中数据的正确性，甚至造成数据库部分或全部数据的丢失。数据库管理系统必须具有将数据库 从错误状态恢复到某一己知的正确状态（亦称为完整状态或一致状态）的功能，这就是数据库的恢复功能。 `rollback/undo/redo`

### 常见的适用于文件系统和数据库系统的场景

> 文件系统：
>
> 1. 包含大量半结构化数据与非结构化数据，如网页、图片等。
> 2. 需要完全隔离权限的操作系统应用（通过权限矩阵控制访问），如FAT、log等（这个举例可能不太妥当）
>
> 数据库系统：
>
> 1. 需要反复查询检索的web应用，如12306等
> 2. 需要DCL保护的应用，如ERP 的不同部门等
> 3. 包含大量并发性的访问与请求，如教务系统（

### 数据库管理系统的主要功能：

>  [Codd](https://en.wikipedia.org/wiki/Edgar_F._Codd) proposed the following functions and services a fully-fledged general purpose DBMS should provide:[[25\]](https://en.wikipedia.org/wiki/Database#cite_note-FOOTNOTEConnollyBegg201497–102-25) `FROM Wiki`
>
> - Data storage, retrieval and update  存储、恢复、更新
> - User accessible catalog or [data dictionary](https://en.wikipedia.org/wiki/Data_dictionary) describing the metadata 如何处理columns？
> - Support for transactions and concurrency 并发控制
> - Facilities for recovering the database should it become damaged 灾备
> - Support for authorization of access and update of data 权限
> - Access support from remote locations 远程
> - Enforcing constraints to ensure data in the database abides by certain rules 数据约束
>
> by the way, **Edgar Frank** "**Ted**" **Codd** (19 August 1923 – 18 April 2003) was an English [computer scientist](https://en.wikipedia.org/wiki/Computer_science) who, while working for [IBM](https://en.wikipedia.org/wiki/International_Business_Machines), invented the [relational model](https://en.wikipedia.org/wiki/Relational_model) for [database](https://en.wikipedia.org/wiki/Database) management, the theoretical basis for [relational databases](https://en.wikipedia.org/wiki/Relational_database) and [relational database management systems](https://en.wikipedia.org/wiki/Relational_database_management_system). He made other valuable contributions to [computer science](https://en.wikipedia.org/wiki/Computer_science), but the relational model, a very influential general theory of data management, remains his most mentioned, analyzed and celebrated achievement. 
>
> 

### 数据库的模式结构

> 上回书说到，The [relational model](https://en.wikipedia.org/wiki/Relational_model) was introduced by [E.F. Codd](https://en.wikipedia.org/wiki/E.F._Codd) in 1970[[2\]](https://en.wikipedia.org/wiki/Database_model#cite_note-2) as a way to make database management systems more independent of any particular application. It is a mathematical model defined in terms of [predicate logic](https://en.wikipedia.org/wiki/Predicate_logic) and [set theory](https://en.wikipedia.org/wiki/Set_theory), and implementations of it have been used by mainframe, midrange and microcomputer systems.
>
> 数据库系统的三级模式结构是指数据库系统是由外模式、模式和内模式三级构成。
>
> 1. **模式（schema)**  模式也称逻辑模式，是数据库中全体数据的逻辑结构和特征的描述，是所有用户的公共数据视图。它是数据库系统模式结构的中间层，既不涉及数据的物理存储细节和硬件环境，又与具体的应用程序、所使用的应用开发工具及高级程序设计语言无关。 `模式实际上是数据库数据在逻辑级上的视图。一个数据库只有一个模式。`数据库模式 以某一种数据模型为基础，统一综合地考虑了所有用户的需求，并将这些需求有机地结合 成一个逻辑整体。定义模式时不仅要定义数据的**逻辑结构**，例如数据记录由哪些数据项构 成，数据项的名字、类型、取值范围等；而且要定义数据之间的联系，定义与数据有关的安全性、完整性要求。 数据库管理系统提供模式数据定义语言（模式DDL)来严格地定义模式。
> 2. **外模式(external schema)** 外模式也称子模式（subschema）或用户模式，它是数据库用户（包括应用程序员和最 终用户）能够看见和使用的局部数据的逻辑结构和特征的描述，是数据库用户的数据**视图**， 是与某一应用有关的数据的逻辑表示。 外模式通常是模式的子集。**一个数据库可以有多个外模式。**
> 3. **内模式(internal schema)** 内模式也称存储模式（storage schema)，**一个数据库只有一个内模式。**它是数据物理 结构和存储方式的描述，是数据在数据库内部的组织方式。例如，记录的存储方式是堆存 储还是按照某个（些）属性值的升（降）序存储，或按照属性值聚簇（cluster)存储；索 引按照什么方式组织，是树索引还是hash索引：数据是否压缩存储，是否加密；数据 的存储记录结构有何规定，如定长结构或变长结构，一个记录不能跨物理页存储；等等。

### 数据独立性

> 上回书说到，数据独立性是借助数据库管理数据的一个显著优点，`通常由DBMS的二级印象保证`。它己成为数据库领域中一个常用术语和重要概念，包括数据的**物理独立性**和**逻辑独立性**。 物理独立性是指用户的**应用程序**与数据库中数据的**物理存储**是相互独立的。也就是说，数据在数据库中怎样存储是山数据库管理系统管理的，用户程序不需要了解，应用程序要处理的只是数据的逻辑结构，这样当数据的物理存储改变时应用程序不用改变。 逻辑独立性是指用户的**应用程序**与**数据库的逻辑结构**是相互独立的。也就是说，数据的逻辑结构改变时用户程序也可以不变。**数据与程序之间的独立性使得数据的定义和描述可以从应用程序中分离出去。**另外， 由于数据的存取由数据库管理系统管理，从而简化了应用程序的编制，大大减少了应用程 序的维护和修改。



### 数据库系统的组成

> 数据库系统是由**数据库、数据库管理系统（及其应用开发工具）、应用程序和数据库管理员**（Database Administrator，DBA)组成的存储、管理、处理和维护数据的系统。`实际上，数据库系统也包括支持系统的硬件及软件，以及数据库设计者及用户等人员组成s`



###  Acknowledgments

> 数据库系统概论，王珊，萨师煊等
>
> 数据库系统概念
>
> Wikipedia
