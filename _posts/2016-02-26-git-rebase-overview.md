---
layout: post
title:  "Обзор Git Rebase"
date:   2016-02-26 11:34:00 +0300
permalink: /git-rebase-overview/
tags: [git, rebase, gitflow, history]
keywords: [git, rebase, gitflow, history]
author: kupec
---
Удаление и перемещение коммитов в **git** опасны потерей данных или истории изменений.
**Git Rebase** выполняет эти и другие операции.
Владение этой командой повышает уверенность при работе с репозиторием.

<!--more-->

## Введение ##

Системы контроля версиями полезны, когда они могут рассказать историю всех изменений.
Эта история может быть как реальный рассказ о жизни и развитии программной системы.
Но может быть и кучкой коммитов, разобраться в которых нельзя и которые не несут пользы.

В больших системах путаница может быть невероятных размеров.
Такие модели как **Git Flow**[^git_flow] сильно облегчают жизнь, но только в том случае, если разработчики аккуратно относятся к своей истории коммитов, когда каждый коммит - это небольшой (или даже совсем крохотный) рассказ об одном исправлении в системе.
Это исправление настолько маленькое, что разобраться в нем не составляет труда, а название коммита, данное разработчиком, такое прозрачное, что заглядывать в тело коммита нет нужды.

[^git_flow]: http://nvie.com/posts/a-successful-git-branching-model

Вести аккуратную историю помогает команда **git rebase**. Основная ее цель "перетаскивать" коммиты из одного места в другое.
Также с помощью нее можно избавляться от ненужных коммитов, склеивать одни и менять порядок других.

Умение пользоваться командой **git rebase** придает уверенности при работе с репозиторием.

## Предостережение ##

Ребейзинг может "удалить" коммиты. Ребейзинг может испортить историю коммитов.

Однако в действительности удалить коммиты может *только сборщик мусора*.
Это знание не сильно поможет, если потерять все ссылки на коммит, но если добавить лишнюю ветку или просто запомнить **sha-1** хеш коммита, то всегда можно восстановить ребейзнутые коммиты.

## О структуре коммитов ##

Коммит - это изменения в файлах репозитория + дополнительная информация. Идентифицируются коммиты с помощью **sha-1** хеша.
*Коммиты с одинаковым хешем - одинаковые, с разными хешами - разные*.

Коммит хранит дату создания, поэтому если создать два одинаковых по содержанию коммита, то они будут иметь разные хеши и будут разными.
Так происходит при выполнение команды **git commit --amend** - команда перезаписывает текущий коммит и даже если ничего не изменилось, хеш все равно станет новым.

Ребейзинг **переприменяет** коммиты, то есть создает коммиты с таким же содержанием, но в другом месте. Новые коммиты получат новые хеши.

## Указатели на коммиты ##

Некоторые названия настолько неудачно подобраны, что, услышав термин, человек сразу понимает о чем идет речь, понимает приблизительно правильно и у него нет причин разбираться глубже.

Речь идет о ветках. Часто представляют ветку, как речку, ответвляющуюся от бурной реки и текущую дальше самостоятельно.
Поэтому кажется, что при создании ветки происходит что-то важное.
На деле ветка это просто указатель на коммит. Разница между хешом и веткой лишь та, что ветка перескакивает с одного коммита на другой по мере их добавления.
В остальном, ветка - это буквенный синоним хеша.

Проще говоря, ветка указывает на коммит. Поэтому если ответвить 5 веток от **master**, то все они будут указывать на один и тот же коммит.
Однако новые коммиты будут свои в каждой из веток.

## Создание коммита ##

Каждый коммит (кроме начального) имеет родительский коммит, который для него аналог "предыдущего".
Из одного коммита может выходить несколько дочерних коммитов.
Это делается с помощью ветвления. А так как ветки могут соединяться в одну, то у коммита может быть и *несколько* родительских коммитов.
Обычно их два, но есть возможность сделать **merge** трех и более коммитов одновременно.

Иногда коммит бывает создан не в том месте и не в то время.
Не в том месте означает, что он был создан не в той ветке, а не в то время означает, что коммит был создан, но потом ветка разработки обогатилась новым кодом и хочется пересоздать коммит, начиная с более свежего кода.

## Пересоздание коммитов ##

Пересоздать коммит просто. Предположим типичную ситуацию - необходимо добавить фичу А. Создается отдельная ветка **feature/A/super_code** от ветки **develop**.
На данном этапе новая ветка указывает по-прежнему на самый свежий коммит.
Пусть фича мысленно разбивается на два коммита.
Первый коммит с успехом создается.
В это время в ветку **develop** добавляются важные коммиты с важным классом Х, без которого завершить фичу А не получится.

Что же делать? Подтягивать изменения. Можно слить ветку **develop** в ветку с задачей. Безопасно, но история будет подпорчена. Делается ребейзинг

```
git rebase develop
```

Предполагается, что **develop** уже содержит новые коммиты, то есть, что уже были сделаны **fetch/pull**.

Как работает команда **git rebase**? В простом случае, это работает так:

- текущая ветка перестает указывать на текущий коммит
- коммиты, которые есть в текущей ветке, но которых нет в **develop** (то есть новые коммиты) пересоздаются от последнего коммита в **develop**
- текущая ветка начинает указывать на только что созданные коммиты
- старые коммиты остаются висячими - на них ветка с задачей больше не указывает

Ниже приведены три дерева коммита

Ветка с задачей исходит из актуального коммита

![](/assets/posts/git-rebase-overview/simple-rebase1.png)

Код разработки обновился, ветка с задачей исходит из старого коммита

![](/assets/posts/git-rebase-overview/simple-rebase2.png)

Выполнен ребейзинг, ветка с задачей исходит из актуального коммита

![](/assets/posts/git-rebase-overview/simple-rebase3.png)

Стоит обратить внимание на хеш коммита до ребейзинга (**8f26890**) и после (**3b16148**). Разумеется, по старому хешу можно найти старый коммит.

Создадим ветку, указывающую на старый коммит

```
git branch before_rebasing 8f26890
```

Дерево коммитов:

![](/assets/posts/git-rebase-overview/simple-rebase4.png)

Нетрудно видеть, что старый коммит остался в целости и сохранности, однако, если на него никто не указывает, то он не отображается в истории коммитов.
Такие висячие коммиты удаляются сборщиком мусора в последствии (например, через месяц).

### Потеря коммитов ###

Чтобы избавиться от риска потерять коммиты, перед ребейзингом следует создать дополнительную ветку

```
git branch backup
```

## Простой Rebase ##

Формат команды такой

```
git rebase [BRANCH or SHA1]
```

Команда изменяет **текущую** ветку.
Переданный параметр - это просто указатель на новое **начало коммитов**. Пересоздаются (копируются) коммиты из диапазона **[BRANCH or SHA1]..HEAD** .
То есть все предки текущего коммита (включая его самого) без предков коммита (включая его самого) **[BRANCH or SHA1]**.
Это то, что интуитивно можно назвать как "новые" коммиты.

### Но в Merge все наоборот! ###

- При ребейзинге текущая ветка - ветка с задачей, а ребейзятся относительно главной ветки.
- При слиянии текущей задачи - переключаются на главную ветку и сливают ветку с задачей.

Правило простое - **изменению подвержена всегда текущая ветка**. Почти все команды в **git** следуют этому правилу.

- При ребейзинге главная ветка не трогается, изменяется ветка с задачей - она пересоздается в другом месте.
- При слиянии главная ветка изменяется - в ней создается мерж-коммит. Ветка с задачей наоборот остается в прежнем виде.

Поэтому при ребейзинге придется дважды переключаться с ветки на ветку

```
git checkout develop
git pull
git checkout feature/A/super_code
git rebase develop
```

Однако если использоваться слияние для подтягивания изменений, то сценарий похожий

```
git checkout develop
git pull
git checkout feature/A/super_code
git merge develop
```

Разница есть только со сценарием завершения задачи

```
git checkout develop
git pull
git merge --no-ff feature/A/super_code
git push
git branch -d feature/A/super_code
```

Таким образом, для того, чтобы определить "что и куда", надо задать вопрос **"какую ветку надо изменить?"**.
Находиться следует в той ветке, которую *надо изменить*.

## Интерактивный Rebase ##

Ребейзинг позволяет также переписывать историю своих коммитов.

Одна из таких ситуаций. 
В ветке с задачей 40 коммитов. 
Последние 2 отвечают за добавления фичи, а предыдущие 38 - это долгий и опасный рефакторинг. 
Для истории достаточно иметь один коммит для рефакторинга, поэтому эти коммиты следует склеить в один.

```
git rebase -i develop
```

Такая команда делает то же самое, что и раньше, но перед этим открывает текстовый редактор, в котором можно изменить поведение по умолчанию.

В редакторе помещены записи о коммитах.
Почти тоже самое, что в логе коммитов, но порядок обратный.
Также перед каждым коммитом записана команда - добавить коммит, объединить с предыдущим и так далее.
Сразу после коммитов **git** приводит инструкцию в комментариях.

Удаление записи о коммите - удаляет коммит, а точнее не создает его. Изменение порядка записей вызывает изменение порядка создания коммитов.

## Rebase и коммиты удаленного репозитория ##

Пересоздавать коммиты, которые были отправлены в удаленный репозиторий не стоит.
Причина простая - у других разработчиков могут остаться ссылки на старые коммиты.
Два одинаковых коммита в разных местах репозитория - печальная картина.

Правило здесь такое - если коммиты не были отправлены в удаленный репозиторий, то с ними можно делать что угодно.
Если же коммиты стали общим достоянием, то ребейзинг не стоит делать без согласования с командой.

## Конфликты ##

Конфликты случаются везде, где накладываются изменения из разных мест.
Это происходит при слиянии (**git merge**), при отмене коммитов (**git revert**) и даже при переключении (**git checkout**, **git stash**).
Во всех командах конфликты разрешаются по общим правилам.
Поэтому в ребейзинге разрешение конфликтов такое же как и при слиянии.

При слиянии создается специальный мерж-коммит, в котором хранятся все изменения, сделанные в отдельной ветке.
Поэтому все конфликты разрешаются в этом же коммите. В истории хранятся все коммиты из ветки с задачей и мерж-коммит.
Последний можно рассматривать как завершающий штрих.

При ребейзинге дополнительных коммитов не создается.
Поэтому разрешение конфликтов будет происходить прямо в коммитах задачи.
По большому счету это большой плюс, так как изменения в коммитах становятся более актуальными.
С другой стороны, разрешение конфликтов может стать болью - конфликты могут появиться в каждом коммите новой ветки.

В помощниках вроде **TortoiseGit** процесс разрешения конфликтов почти ничем не отличается от слияния.
В консоли используются следующие команды.

```
git rebase --continue
git rebase --abort
```

## Детальный Rebase ##

Существует еще одна форма команды ребейзинга

```
git rebase [FROM] [TO] --onto [START]
```

**FROM**, **TO**, **START** -- хеши коммитов или названия веток/тегов.
Команда копирует коммиты из диапазона **FROM..TO** в **START**.

Если **TO** - хеш, а не ветка, то результат работы увидеть сложно. Обычно это изменяемая ветка.
По сути новый параметр здесь только **FROM**, который обрезает ветку.

Стандартная команда, выполненная, находясь внутри ветки **feature/A/super_code**

```
git rebase develop
```

Может быть заменена на

```
git rebase develop feature/A/super_code --onto develop
```

### Примеры ###

Необходимо скопировать последний коммит из ветки с задачей в **develop**

```
git rebase feature/A/super_code^ feature/A/super_code --onto develop
```

(в **git** можно указать на родительский коммит с помощью значка шапочки)

------------------------------------

Коммит сделан в ветке **feature/A**, а должен был быть в ветке **bug/B**.

```bash
# save origin of bug/B
git branch copy_bug_B bug/B
# move bug/B to new commit
git branch -f bug/B feature/A
# move feature/A back
git branch -f feature/A feature/A^

# copy new commit to saved origin of bug/B
git rebase bug/B^ bug/B --onto copy_bug_B

# remove temporary branch
git branch -d copy_bug_B
```

------------------------------------------

## Чистая история коммитов ##

Рефакторинг привыкли относить только к коду, но хорошая и наглядная история коммитов в репозитории может дать больше информации разработчику.
Правило бойскаута из книги "Чистый код" Роберта Мартина «оставь место стоянки чище, чем оно было до твоего прихода» применимо и к истории коммитов.
Поэтому использование **Git Rebase** также полезно в работе с репозиторием, как инструменты рефакторинга в IDE для работы с кодом.