# How to solve ODEs in real-time HLA distributed simulation

https://hal.archives-ouvertes.fr/hal-01399161/document

Jean-Baptiste Chaudron, David Saussié, Pierre Siron, Martin Adelantado

**ABSTRACT:** *In the context of the Research Platform for Embedded Systems Engineering (PRISE) Project, we are developing and maintaining a complete aircraft flight simulation using the High Level Architecture (HLA), an IEEE standard for distributed simulation. This complex distributed simulation is composed of different distributed HLA simulators (e.g., Flight Dynamics, Sensors), whose dynamic behaviors are implemented as Ordinary Differential Equations (ODEs). The resolution of these equations is done, locally for each simulator, by numerical integration with methods like Euler or Adams-Bashforth. The global behavior of this distributed simulation, where each component runs its own local resolution, is a key challenge. The main problem is to ensure the global simulation consistency and, in particular, the specific data flows between components with the correct temporal real-time behavior. This paper specifically addresses the problem of solving ODEs over an HLA distributed architecture and offers a complete study (specifications, implementation and validation) where several theoretical concepts and methods are discussed.*

**АНОТАЦІЯ:** *У контексті проекту «Дослідницька платформа для розробки вбудованих систем» (PRISE) ми розробляємо та підтримуємо повну симуляцію польоту літака з використанням архітектури високого рівня (HLA), стандарту IEEE для розподіленого моделювання. Це складне розподілене моделювання складається з різних розподілених симуляторів HLA (наприклад, динаміки польоту, датчиків), чия динамічна поведінка реалізована як звичайні диференціальні рівняння (ODE). Розв’язання цих рівнянь виконується локально для кожного симулятора шляхом чисельного інтегрування з такими методами, як Ейлер або Адамс-Бешфорт. Глобальна поведінка цього розподіленого моделювання, де кожен компонент виконує власну локальну роздільну здатність, є ключовим викликом. Основна проблема полягає в тому, щоб забезпечити узгодженість глобального моделювання і, зокрема, певні потоки даних між компонентами з правильною тимчасовою поведінкою в реальному часі. У цьому документі конкретно розглядається проблема вирішення ODE через розподілену архітектуру HLA та пропонується повне дослідження (специфікації, реалізація та перевірка), де обговорюються кілька теоретичних концепцій і методів.*

## 1.  Introduction

The main goal of the Research Platform for Embedded Systems Engineering (PRISE) project is to provide students and researchers with a platform for the study, evaluation and validation of new embedded system concepts, architectures and techniques through a dedicated hardware and software environment. Modern embedded systems become more and more complex with an increasing number of both components and interactions between them and the use of simulation is essential for relevant studies. In this context, we implemented from scratch and we are currently maintaining an aircraft component-based simulation composed of several distributed simulators [1], [2]. Each simulator represents a specific part of the aircraft (e.g., engines, actuators, control devices, flight control laws), its dynamic in the environment (flight dynamics) or the environment itself. An overview of this distributed simulation is depicted in Figure 1.

Основна мета проекту «Дослідницька платформа для розробки вбудованих систем» (PRISE) полягає в тому, щоб надати студентам і дослідникам платформу для вивчення, оцінки та перевірки нових концепцій, архітектур і методів вбудованих систем за допомогою спеціального апаратного та програмного середовища. Сучасні вбудовані системи стають все більш і більш складними зі збільшенням кількості компонентів і взаємодій між ними, і використання моделювання є важливим для відповідних досліджень. У цьому контексті ми реалізували з нуля і зараз підтримуємо моделювання на основі компонентів літака, що складається з кількох розподілених симуляторів [1], [2]. Кожен симулятор представляє певну частину літака (наприклад, двигуни, приводи, пристрої керування, закони керування польотом), його динаміку в навколишньому середовищі (динаміка польоту) або саме середовище. Огляд цього розподіленого моделювання зображено на малюнку 1.

![image-20220816162103415](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816162103415.png)

Figure 1. Overview of the SDSE aircraft simulation

This flight simulation, called SDSE (French acronym for Distributed Simulation of Embedded Systems) has been implemented by using the HLA IEEE standard [3], [4], [5] which provides a well-known simulation framework for composability, mainte- nance and reusability for this complex simulation. Moreover, HLA provides time management mechanisms [6], guaranteeing a consistent global logical time throughout the whole simulation, which is one of the main benefits of this simulation standard. Previous works showed that these time management mechanisms could be used to ensure the good behavior during run time,  especially in real-time simulations [7], [8], which is typically an aspect of the SDSE simulation. From the software point of view, we are using the open source CERTI [9] Run Time Infrastructure (HLA middleware implementation) because we have a complete control on its implementation. We have shown that it is suitable for interconnection of simulators (i.e., HLA federates) with short computation and communication cycles [10], [11]. Additionally, we have also investigated the usage of CERTI time management techniques and implementations for real-time simulations [12] and especially for SDSE.

Ця симуляція польоту, яка називається SDSE (французька абревіатура від Distributed Simulation of Embedded Systems), була реалізована за допомогою стандарту HLA IEEE [3], [4], [5], який забезпечує добре відому структуру моделювання для комбінування, обслуговування і можливість повторного використання для цього складного моделювання. Крім того, HLA забезпечує механізми керування часом [6], гарантуючи послідовний глобальний логічний час протягом усього моделювання, що є однією з головних переваг цього стандарту моделювання. Попередні роботи показали, що ці механізми керування часом можуть бути використані для забезпечення належної поведінки під час виконання, особливо в симуляції в реальному часі [7], [8], яка зазвичай є аспектом моделювання SDSE. З точки зору програмного забезпечення ми використовуємо інфраструктуру часу виконання CERTI [9] з відкритим кодом (реалізація проміжного програмного забезпечення HLA), оскільки ми повністю контролюємо її впровадження. Ми показали, що він підходить для взаємозв’язку симуляторів (тобто HLA-федератів) із короткими циклами обчислення та зв’язку [10], [11]. Крім того, ми також досліджували використання методів керування часом CERTI та реалізацій для моделювання в реальному часі [12] і особливо для SDSE.

Scheduling a real-time federation (i.e., an HLA simulation) consists of two complementary facets that must be distinguished, namely, the *temporal scheduling* and the *functional scheduling*.

1) The *temporal scheduling* belongs to scheduling theory. Here, the objective is to formally verify that the distributed simulation will run while ensuring compliance with the real-time constraints according to its specifications. This aspect has been addressed in [2] and is not the purpose of the present study.

2) The *functional scheduling* ensures that the sequence of the communications and computations between each simulator will provide global simulation results that will always be fair and relevant according to simulation model semantics. HLA time management mechanisms are providing a good baseline for this purpose. In the SDSE simulation, each federate implements a dedicated specific algorithm and some of these models are using Ordinary Differential Equations (ODEs) [13]. Therefore, the distributed resolution of these ODEs is quite complex and needs to be clearly analyzed; this is the purpose of the paper.

Планування об’єднання в реальному часі (тобто моделювання HLA) складається з двох взаємодоповнюючих аспектів, які слід розрізняти, а саме *часового планування* та *функціонального планування*.

1) *Часове планування* належить до теорії планування. Тут мета полягає в тому, щоб офіційно перевірити, що розподілене моделювання працюватиме, забезпечуючи відповідність обмеженням реального часу відповідно до його специфікацій. Цей аспект розглядався в [2] і не є метою цього дослідження.

2) *Функціональне планування* гарантує, що послідовність зв’язків і обчислень між кожним симулятором забезпечить глобальні результати моделювання, які завжди будуть чесними та відповідними відповідно до семантики моделі імітації. Механізми управління часом HLA є хорошою базою для цієї мети. У моделюванні SDSE кожна федерація реалізує спеціальний спеціальний алгоритм, а деякі з цих моделей використовують звичайні диференціальні рівняння (ODE) [13]. Таким чином, розподілена роздільна здатність цих ODE є досить складною і потребує чіткого аналізу; це мета статті.

Also, the notion of “*time*” within a real-time simulation is an important notion which needs to be clearly stated for consistent temporal and functional scheduling study. Three different concepts can be associated to the notion of time, each one describing a specific aspect [14]:

1) The *physical time*, denoted by *tPHY* , is the reference time of the physical system under study (the source system that we want to model and to simulate).

2) The *simulated time*, denoted by *tSIM* , is a representation of the physical time within the simulation. It is a set of ordered values representing different time instants for the modeled physical system.

3) *The absolute time* or *wall-clock time*, denoted by *tWCT* , is the time elapsed while executing the simulation. It can be measured by a hardware clock, like the CPU clock.

Крім того, поняття «*часу*» в симуляції в реальному часі є важливим поняттям, яке необхідно чітко сформулювати для послідовного дослідження часового та функціонального планування. З поняттям часу можна пов’язати три різні концепції, кожна з яких описує певний аспект [14]:

1) *Фізичний час*, позначений *tPHY*, є еталонним часом досліджуваної фізичної системи (вихідної системи, яку ми хочемо моделювати та імітувати).

2) *Змодельований час*, позначений *tSIM*, є представленням фізичного часу в рамках моделювання. Це набір упорядкованих значень, що представляють різні моменти часу для змодельованої фізичної системи.

3) *Абсолютний час* або *час настінного годинника*, позначений *tWCT*, це час, що минув під час виконання моделювання. Його можна виміряти апаратним годинником, як годинник ЦП.

In our real-time simulations, with the usage of conservative time management methods, these times might be equal (with respect to a certain unit and a certain precision). Therefore, during the run of a simulation, we have:

У наших симуляціях у реальному часі з використанням консервативних методів керування часом ці часи можуть бути рівними (щодо певної одиниці та певної точності). Отже, під час виконання моделювання ми маємо:

![image-20220816162710210](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816162710210.png) (1)

For example, if the *millisecond* is considered as the reference unit, 1 millisecond of the real physical system is equal to 1 unit of logical HLA simulation time, which is also equal to 1 millisecond of execution time.

Наприклад, якщо *мілісекунда* розглядається як еталонна одиниця, 1 мілісекунда реальної фізичної системи дорівнює 1 одиниці логічного часу симуляції HLA, який також дорівнює 1 мілісекунди часу виконання.

To resume, this paper focuses on the study of functional scheduling for HLA simulations (i.e., federations) using HLA time management semantics where some simulators (i.e., federates) are modeled by ODEs. Based on our experience with the SDSE aircraft simulation development, we present here a detailed analysis from initial ODEs characterization to distributed resolution description. The paper is organized as follows:

• Section 2 describes ODEs characterization for our real-time simulation;

• Section 3 outlines the resolution of ODEs for federation with a communication sequence;

• Section 4 presents the resolution of ODEs for federation with a communication loop;

• Section 5 summarizes these concepts and shows an illustration for the SDSE aircraft simulation.

Підсумовуючи, ця стаття зосереджена на вивченні функціонального планування для моделювання HLA (тобто федерацій) з використанням семантики керування часом HLA, де деякі симулятори (тобто федерати) моделюються ODE. Базуючись на нашому досвіді розробки моделювання літаків SDSE, ми представляємо тут детальний аналіз від початкових характеристик ODE до опису розподіленої роздільної здатності. Стаття організована таким чином:

• Розділ 2 описує характеристики ODE для нашого моделювання в реальному часі;

• Розділ 3 описує розділення ODE для об’єднання з послідовністю зв’язку;

• Розділ 4 представляє вирішення ODE для об'єднання з контуром зв'язку;

• Розділ 5 підсумовує ці концепції та показує ілюстрацію моделювання літака SDSE.

## 2.  ODEs Characterization

### 2.1  State-Space Description

The dynamic behavior of a system can usually be modeled by a set of Ordinary Differential Equations (ODEs) and/or Partial Derivative Equations (PDEs) derived from different laws of physics. Under the action of external excitation (*inputs*) and/or initial conditions, the system evolves accordingly, and one can then monitor signals of interest (*outputs*), representative of the system behavior. In this paper, we suppose that the federates simulate systems governed by ODEs and described by state-space models of the form:

Динамічну поведінку системи зазвичай можна змоделювати за допомогою набору звичайних диференціальних рівнянь (ODE) та/або рівнянь із частковими похідними (PDE), виведених із різних законів фізики. Під дією зовнішнього збудження (*входи*) та/або початкових умов система розвивається відповідно, і потім можна відстежувати цікаві сигнали (*виходи*), що представляють поведінку системи. У цьому документі ми припускаємо, що федерати моделюють системи, які керуються ODE і описуються моделями простору станів такого вигляду:

![image-20220816163027281](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816163027281.png)

 From an HLA point of view, a federate subscribes to object attributes that will constitute the input vector that drives the state equation, and publishes object attributes yielded by the output equation.

 З точки зору HLA, федерація підписується на атрибути об’єкта, які становитимуть вхідний вектор, що керує рівнянням стану, і публікує атрибути об’єкта, отримані з рівняння виходу.

### 2.2  Numerical methods for ODEs

Numerical solutions of ODEs can be performed by a large variety of methods depending on the nature of the problem and the desired accuracy [13]. Starting from an initial point, a numerical method takes a short step forward in time to find the next point, and so on. Among all the available numerical methods, one can find single-step methods (e.g., Euler method), Runge-Kutta methods, and multistep methods (e.g., Adams-Bashforth and Adams-Moulton methods). These methods imply the choice of an adequate time step ∆*t*, which can have a significant impact on the accuracy of the solution; they all lead to recurrence equations. For instance, the forward Euler method applied to original state equation (2) leads to the recurrence equation (5):

Чисельні розв’язки ОДЗ можуть виконуватися великою різноманітністю методів залежно від характеру задачі та бажаної точності [13]. Починаючи з початкової точки, чисельний метод робить короткий крок вперед у часі, щоб знайти наступну точку і так далі. Серед усіх доступних чисельних методів можна знайти однокрокові методи (наприклад, метод Ейлера), методи Рунге-Кутта та багатокрокові методи (наприклад, методи Адамса-Бешфорта та Адамса-Моултона). Ці методи передбачають вибір адекватного кроку за часом ∆*t*, який може суттєво вплинути на точність розв’язку; усі вони призводять до рекурентних рівнянь. Наприклад, прямий метод Ейлера, застосований до вихідного рівняння стану (2), призводить до рекурентного рівняння (5): 

![image-20220816163359630](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816163359630.png)

![image-20220816163421962](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816163421962.png)

On the opposite, there are implicit methods such as the backward Euler method described here in equation (9):

Навпаки, існують неявні методи, такі як зворотний метод Ейлера, описаний тут у рівнянні (9):

![image-20220816163454747](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816163454747.png)

where one needs to solve a non-linear algebraic equation at every time step in order to produce the next value **x***n*+1; this is usually achieved by using fixed point iteration or Newton-Raphson method. The computational cost is obviously higher, but implicit methods are usually more stable than their explicit counterparts. Nevertheless, the implicit methods are rejected in this HLA simulation context as they necessitate the value of the input **u***n*+1 at the next step *n* + 1, i.e., in the future.

де потрібно розв’язувати нелінійне алгебраїчне рівняння на кожному кроці за часом, щоб отримати наступне значення **x***n*+1; зазвичай це досягається за допомогою ітерації з фіксованою точкою або методу Ньютона-Рафсона. Витрати на обчислення, очевидно, вищі, але неявні методи зазвичай більш стабільні, ніж їхні явні аналоги. Тим не менш, неявні методи відхиляються в цьому контексті моделювання HLA, оскільки вони вимагають значення вхідних даних **u***n*+1 на наступному кроці *n* + 1, тобто в майбутньому.

### 2.3  Direct feedthrough

The output equation (3) is an algebraic equation that produces the output vector **y**(*t*) containing the signals of interest. Generally, the output vector **y**(*t*) may depend on both the state vector **x**(*t*) and the input vector **u**(*t*), and eventually, explicitly on the time *t*, as in equation (3). Contrary to the state equation (2) which requires numeric solvers as shown above, the discretization of the output equation, described in (10), is straightforward:

Вихідне рівняння (3) є алгебраїчним рівнянням, яке створює вихідний вектор **y**(*t*), що містить цікаві сигнали. Загалом вихідний вектор **y**(*t*) може залежати як від вектора стану **x**(*t*), так і від вхідного вектора **u**(*t*), і, зрештою, явно на час *t*, як у рівнянні (3). На відміну від рівняння стану (2), яке вимагає числових розв’язувачів, як показано вище, дискретизація вихідного рівняння, описана в (10), є простою:

![image-20220816163623924](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816163623924.png)

Nevertheless, special attention must be paid to the presence, or not, of the input vector **u**(*t*) (or **u***n*) in the output equation. When the system output **y**(*t*) depends on the input vector **u**(*t*) (or on some of the inputs *ui*), it is called *direct feedthrough*. As it will be presented in Sections 3 and 4, this can reveal particularly intricate when simulating a distributed system. In the case where there is no direct feedthrough, the output equation (11) is:

Тим не менш, особливу увагу слід звернути на наявність чи відсутність вхідного вектора **u**(*t*) (або **u***n*) у вихідному рівнянні. Коли вихід системи **y**(*t*) залежить від вхідного вектора **u**(*t*) (або від деяких входів *ui*), це називається *прямим проходженням*. Як це буде представлено в розділах 3 і 4, це може виявитися особливо складним при моделюванні розподіленої системи. У випадку, коли немає прямого проходу, вихідне рівняння (11) має вигляд:

![image-20220816163712054](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816163712054.png)

### 2.4  Typical execution within a federate

The choice of the numerical method and the time step ∆*t* depends on the characteristics of the system but is not discussed here. The discretization of a state-space model with an explicit linear multi step method leads to the following discreet system (12):

Вибір чисельного методу та кроку за часом ∆*t* залежить від характеристик системи, але тут не обговорюється. Дискретизація моделі простору станів за допомогою явного лінійного багатокрокового методу призводить до наступної дискретної системи (12):

![image-20220816163812814](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816163812814.png)

At iteration *n*, knowing **x***n* and **u***n*, the output equation is first executed to provide **y***n*, then the state equation yields the state vector update **x***n*+1 for the next iteration. Note that depending on the numerical methods, previous values **x***n**−**k* and **u***n**−**k* must be stored for the recurrence equation.

На ітерації *n*, знаючи **x***n* і **u***n*, вихідне рівняння спочатку виконується, щоб отримати **y***n*, а потім рівняння стану дає вектор стану оновити **x***n*+1 для наступної ітерації. Зауважте, що залежно від чисельних методів попередні значення **x***n**−**k* і **u***n**−**k* повинні зберігатися для рекурентного рівняння.

For the sake of simplicity but without loss of generality, we will consider in the following linear time-invariant (LTI) systems, i.e, systems described by linear differential and algebraic equations with constant coefficients. The original equations (2)-(3) can be rewritten as linear equations (13)-(14):

Заради простоти, але без втрати загальності, ми розглянемо далі лінійні інваріантні в часі (LTI) системи, тобто системи, що описуються лінійними диференціальними та алгебраїчними рівняннями зі постійними коефіцієнтами. Вихідні рівняння (2)-(3) можна переписати як лінійні рівняння (13)-(14):

![image-20220816163951641](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816163951641.png)

In the following, the influence of direct feedthrough on simulation, is illustrated on a first example with a federation involving federates with a communication sequence. Thereafter, we pursue the study on a federation that presents communication loop between federates. These studies will lead to the establishment of simple rules for the automatic implementation of consistent functional scheduling for these types of federation.

Далі вплив прямого проходження на симуляцію проілюстровано на першому прикладі федерації, що включає федератів із послідовністю зв’язку. Після цього ми продовжуємо дослідження федерації, яка представляє петлю зв’язку між федератами. Ці дослідження призведуть до встановлення простих правил для автоматичного впровадження послідовного функціонального планування для цих типів федерації.

## 3.  Federation with a Communication Sequence

Let an HLA federation composed of 4 federates which are communicating in series as illustrated in Figure 2. We study the impact of direct feedthrough in some of the federates.

Нехай федерація HLA складається з 4 федератів, які взаємодіють послідовно, як показано на малюнку 2. Ми вивчаємо вплив прямого проходження в деяких федератах.

![image-20220816164111249](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816164111249.png)

Figure 2.   Federation with federates communicating in series

### **3.1**  **Standard case**

Federate 0 is a federate which provides a signal **u**1*,n* (every cycle *n*). For the sake of simplicity but without loss of generality, it is assumed that the numerical solution of the state-space models representing the dynamic behavior of the next 3 federates is described in equations (17)-(18):

Федерат 0 — це федерат, який забезпечує сигнал **u**1*,n* (кожен цикл *n*). Заради простоти, але без втрати загальності, передбачається, що числове рішення моделей простору станів, що представляють динамічну поведінку наступних 3 федератів, описано в рівняннях (17)-(18):

![image-20220816164136402](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816164136402.png)

where *i* *∈* 1*,* 2*,* 3 denotes the federate index. The federates are linked in series such that the output of Federate *i* is entirely the input of Federate *i* + 1, i.e., *i*     2*,* 3, **u***i,n* = **y***i**−*1*,n*. We can see that these federates do not exhibit direct feedthrough in their respective output equation, and therefore no special care is needed for the execution of the federates. This gives the following global system with equations (19)-(20):

де *i* *∈* 1*,* 2*,* 3 позначає федеративний індекс. Федерати з’єднані послідовно таким чином, що вихід Federate *i* є повністю входом Federate *i* + 1, тобто *i* 2*,* 3, **u***i,n* = * *y***i**−*1*,n*. Ми бачимо, що ці федерати не демонструють прямого проходження у відповідних вихідних рівняннях, і тому не потрібна особлива увага для виконання федератів. Це дає таку глобальну систему з рівняннями (19)-(20):

![image-20220816164227685](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816164227685.png)

З початкових умов [x1,0 x2,0 x3,0]> ми можемо обчислити стан і вихід для кожного кроку ітерації [y1,n y2,n y3,n]>. Ми припускаємо, що з початкового моменту t = 0 усі стани з Федератів 1, 2 і 3 є xi,0, ∆t — це часовий крок між циклом, а lk ∆t — це попередній перегляд. Federate 0 може бути реалізований відповідно до псевдокоду, описаного в таблиці 1, а інші Federate, як у таблиці 2.

Also, we consider the case where each federate is regulator and constrained for HLA time management mechanisms and we do not consider the *zero-lookahead* case (i.e., *lk* = 0).

Крім того, ми розглядаємо випадок, коли кожен федерат є регулятором і має обмеження для механізмів керування часом HLA, і ми не розглядаємо випадок *нульового перегляду* (тобто *lk* = 0).

Figure 3 illustrates the temporal behavior of this federation with ∆*t* = 10 and *lk* = 1. Letters *O* and *S* describe respectively output equation calculation and state equation calculation in every federate of interest.

Рисунок 3 ілюструє тимчасову поведінку цієї федерації з ∆*t* = 10 і *lk* = 1. Літери *O* і *S* описують відповідно обчислення вихідного рівняння та розрахунок рівняння стану в кожній федерації, що представляє інтерес.

![image-20220816164304912](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816164304912.png)

![image-20220816164318181](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816164318181.png)

![image-20220816164336859](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816164336859.png)

### **3.2**  **Direct feedthrough case**

We still consider the test case described above but we are assuming now that Federates 2 and 3 contain direct feedthrough relation. The equation system for Federates 1, 2 and 3 is depicted in (21)-(22)-(23)-(24):

Ми все ще розглядаємо тестовий випадок, описаний вище, але тепер припускаємо, що Federates 2 і 3 містять пряме наскрізне відношення. Система рівнянь для федератів 1, 2 і 3 зображена в (21)-(22)-(23)-(24):

![image-20220816164523414](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816164523414.png)

![image-20220816164538005](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816164538005.png)

​                 *∀* *∈*                

Reminding that *i* 1*,* 2*,* 3*,* **u***i,n* = **y***i**−*1*,n* we intuitively feel that direct feedthrough will cause problems. Let’s consider Federate 2 implemented as described in Table 2. At phase 2.(a), the federate should calculate its output value **y**2*,n* from **x**2*,n* and **u**2*,n* = **y**1*,n*. However for *n >* 0, according to this pseudo-code implementation, Federate 1 doesn’t know about **y**1*,n* but only about **y**1*,n**−*1. This means that it knows values from the previous step but not the one from the same step which has just been calculated per federate 1. Therefore for the same cycle, Federate 2 needs to know its input at instant *n* to calculate **y**2*,n*. As Federates are communicating in sequence, this problem is even worth for Federate 3 which must wait for the update of the output from Federate 2, which is itself depending on Federate 1. So there is a functional schedule principle to build in order to ensure consistency between publications and acquisitions.

Нагадуючи, що *i* 1*,* 2*,* 3*,* **u***i,n* = **y***i**−*1*,n*, ми інтуїтивно відчуваємо, що прямий прохід викличе проблеми. Давайте розглянемо Federate 2, реалізований, як описано в таблиці 2. На етапі 2.(a) федерація повинна обчислити своє вихідне значення **y**2*,n* з **x**2*,n* і ** u**2*,n* = **y**1*,n*. Однак для *n >* 0, відповідно до цієї реалізації псевдокоду, Federate 1 не знає про **y**1*,n*, а лише про **y**1*,n**−*1 . Це означає, що йому відомі значення з попереднього кроку, але не значення з того самого кроку, яке щойно було обчислено для об’єднання 1. Тому для того самого циклу об’єднання 2 має знати свої вхідні дані в момент *n*, щоб обчислити **y **2*,n*. Оскільки Federate спілкуються послідовно, ця проблема стоїть навіть для Federate 3, який має чекати оновлення виводу від Federate 2, який сам залежить від Federate 1. Отже, існує принцип функціонального розкладу, який потрібно побудувати, щоб забезпечити узгодженість. між публікаціями та придбаннями.

We solve this problem by combining the usage of NER with the usage of TAR and we also define a dedicated and correct value for the *lookahead*. NER service allows to request an advance in the HLA logical time but unlike the TAR service, this time advance will be granted either for the required time or for the time of the first available RAV if it is smaller than the requested time. Here the choice of a correct *lookahead* is a key issue to ensure a correct number of UAV/RAV for the same simulation cycle. Implementation strategies For federate 0 and Federate 1 don’t change but, as described in Tables 3 and 4, Federates 2 and 3 contain NER usage in their implementation strategies. Figure 4 illustrates the temporal behavior of this new strategy with ∆*t* = 10 et *lk* = 1.

Ми вирішуємо цю проблему, поєднуючи використання NER із використанням TAR, а також визначаємо спеціальне та правильне значення для *lookahead*. Послуга NER дозволяє запитувати випередження в логічному часі HLA, але на відміну від служби TAR, це випередження часу буде надано або на потрібний час, або на час першого доступного RAV, якщо він менший за запитуваний час. Тут вибір правильного *прогнозного* є ключовим питанням для забезпечення правильної кількості БПЛА/РАВ для того самого циклу моделювання. Стратегії реалізації для federate 0 і Federate 1 не змінюються, але, як описано в таблицях 3 і 4, Federates 2 і 3 містять використання NER у своїх стратегіях впровадження. Рисунок 4 ілюструє часову поведінку цієї нової стратегії з ∆*t* = 10 і *lk* = 1.

![image-20220816164611781](E:\san\Технології\моделиров\gitver_rtsimul\articles\media\image-20220816164611781.png)