# Системы контроля версий - *Проектная работа 3* 

В качестве хостинга репозиториев с кодом выбран GitHub. Репозиторий с кодом располагается по адресу - https://github.com/slavnyj/sf_svc. Посмотреть существующий код можно в веб-интерфейсе, если имеется доступ к репозиторию или можно посмотреть на локальном сервере склонировав репозиторий командой 

```
git clone git@github.com:slavnyj/SkillFactory.git
```

Предлагаемая модель ветвления опирается на конфигурацию проекта, содержащую один центральный «истинный» репозиторий. Этот репозиторий считается центральным (так как Git является DVCS, у него нет такой вещи, как главный репозиторий, на техническом уровне). Мы будем называть этот репозиторий термином origin.
Каждый разработчик забирает и публикует изменения (pull & push) в origin. Но, помимо централизованных отношений push-pull, каждый разработчик также может забирать изменения от остальных коллег внутри своей микро-команды. Например, этот способ может быть удобен в ситуации, когда двое или более разработчиков работают вместе над большой новой фичей, но не могут издать незавершённую работу в origin раньше времени. Технически это реализуется несложно: Алиса создаёт удалённую ветку Git под названием bob, которая указывает на репозиторий Боба, а Боб делает то же самое с её репозиторием.

## Главные ветви  
Центральный репозиторий содержит две главные ветки, существующие всё время - **main** и **develop**.
Ветка **main** создаётся при инициализации репозитория. Параллельно ей также мы создаём ветку для разработки под названием **develop**.

Мы считаем ветку **origin/main** главной. То есть, исходный код в ней должен находиться в состоянии production-ready в любой произвольный момент времени.

Ветвь **origin/develop** мы считаем главной ветвью для разработки. Хранящийся в ней код в любой момент времени должен содержать самые последние изданные изменения, необходимые для следующего релиза. 
Когда исходный код в ветке разработки (develop) достигает стабильного состояния и готов к релизу, все изменения должны быть определённым способом влиты в главную ветку (main) и помечены тегом с номером релиза. Ниже мы рассмотрим этот процесс в деталях.

Следовательно, каждый раз, когда изменения вливаются в главную ветку (main), мы по определению получаем новый релиз.

## Вспомогательные ветви

Помимо главных веток **main** и **develop**, наша модель разработки содержит некоторое количество типов вспомогательных веток, которые используются для распараллеливания разработки между членами команды, для упрощения внедрения нового функционала (features), для подготовки релизов и для быстрого исправления проблем в производственной версии приложения. В отличие от главных веток, эти ветви всегда имеют ограниченный срок жизни. Каждая из них в конечном итоге рано или поздно удаляется.

Мы используем следующие типы веток: 
- Ветви функциональностей (Feature branches)
- Ветви релизов (Release branches)
- Ветви исправлений (Hotfix branches)

У каждого типа веток есть своё специфическое назначение и строгий набор правил, от каких веток они могут порождаться, и в какие должны вливаться. Ниже мы рассмотрим их по очереди. C технической точки зрения, у этих веток нет ничего «специфического». Разбиение веток на категории существует только с точки зрения того, как они используются. 

### Ветви функциональностей (feature branches)
***Могут порождаться от: develop***  
***Должны вливаться в: develop***  
***Соглашение о наименовании: всё, за исключением main, develop, release-\* или hotfix-\**** 

Ветви функциональностей (feature branches), также называемые иногда тематическими ветвями (topic branches), используются для разработки новых функций, которые должны появиться в текущем или будущем релизах. При начале работы над функциональностью (фичей) может быть ещё неизвестно, в какой именно релиз она будет добавлена. Смысл существования ветви функциональности (feature branch) состоит в том, что она живёт так долго, сколько продолжается разработка данной функциональности (фичи). Когда работа в ветке завершена, последняя вливается обратно в главную ветку разработки (что означает, что функциональность будет добавлена в грядущий релиз) или же удаляется (в случае неудачного эксперимента).

Ветви функциональностей (feature branches) обычно существуют в репозиториях разработчиков, но не в главном репозитории (origin).

**Создание ветви функциональности (feature branch)**  
При начале работы над новой функциональностью делается ответвление от ветви разработки (**develop**).
```
$ git checkout -b myfeature develop
Switched to a new branch "myfeature"
```
**Добавление завершённой функциональности в develop**  
Завершённая функциональность (фича) вливается обратно в ветвь разработки (**develop**) и попадает в следующий релиз.
```
$ git checkout develop
Switched to branch 'develop'
$ git merge --no-ff myfeature
Updating ea1b82a..05e9557
(Отчёт об изменениях)
$ git branch -d myfeature
Deleted branch myfeature (was 05e9557).
$ git push origin develop
```
Флаг `--no-ff` вынуждает git всегда создавать новый объект коммита при слиянии, даже если слияние может быть осуществлено алгоритмом fast-forward. Это позволяет не терять информацию о том, что ветка существовала, и группирует вместе все внесённые изменения.  
Конечно, такой подход создаёт некоторое дополнительное количество (пустых) объектов коммитов, но получаемая выгода более чем оправдывает подобную цену.

### Ветви релизов (release branches)

***Могут порождаться от: develop***  
***Должны вливаться в: develop и main***  
***Соглашение о наименовании: release-\****

Ветви релизов (release branches) используются для подготовки к выпуску новых версий продукта. Они позволяют расставить финальные точки над i перед выпуском новой версии. Кроме того, в них можно добавлять минорные исправления, а также подготавливать метаданные для очередного релиза (номер версии, дата сборки и т.д.). Когда вся эта работа выносится в ветвь релизов, главная ветвь разработки (develop) очищается для добавления последующих фич (которые войдут в следующий большой релиз).

Новую ветку релиза (release branch) надо порождать в тот момент, когда состояние ветви разработки полностью или почти полностью соответствует требованиям, соответствующим новому релизу. По крайней мере, вся необходимая функциональность, предназначенная к этому релизу, уже влита в ветвь разработки (develop). Функциональность, предназначенная к следующим релизам, может быть и не влита. Даже лучше, если ветки для этих функциональностей подождут, пока текущая ветвь релиза не отпочкуется от ветви разработки (develop).

Очередной релиз получает свой номер версии только в тот момент, когда для него создаётся новая ветвь, но ни в коем случае не раньше. Вплоть до этого момента ветвь разработки содержит изменения для «нового релиза», но пока ветка релиза не отделилась, точно неизвестно, будет ли этот релиз иметь версию 0.3, или 1.0, или какую-то другую. Решение принимается при создании новой ветви релиза и зависит от принятых на проекте правил нумерации версий проекта.  

**Создание ветви релиза (release branch)**  
Ветвь релиза создаётся из ветви разработки (**develop**). Например, текущий изданный релиз имеет версию 1.1.5, а на подходе новый большой релиз, полный изменений. Ветка разработки (**develop**) готова к «следующему релизу», и мы решаем, что этот релиз будет иметь версию 1.2 (а не 1.1.6 или 2.0). В таком случае мы создаём новую ветку и даём ей имя, соответствующее новой версии проекта:
```
$ git checkout -b release-1.2 develop
Switched to a new branch "release-1.2"
$ ./bump-version.sh 1.2
Files modified successfully, version bumped to 1.2.
$ git commit -a -m "Bumped version number to 1.2"
[release-1.2 74d9424] Bumped version number to 1.2
1 files changed, 1 insertions(+), 1 deletions(-)
```
Мы создали новую ветку, переключились в неё, а затем выставили номер версии (bump version number). В нашем примере bump-version.sh — это вымышленный скрипт, который изменяет некоторые файлы в рабочей копии, записывая в них новую версию. (Конечно, эти изменения можно внести и вручную.) Затем мы делаем коммит с указанием новой версии проекта.

Эта новая ветвь может существовать ещё некоторое время, до тех пор, пока новый релиз окончательно не будет готов к выпуску. В течение этого времени к этой ветви (а не к develop) могут быть добавлены исправления найденных багов. Но добавление крупных новых изменений в эту ветвь строго запрещено. Они всегда должны вливаться в ветвь разработки (develop) и ждать следующего большого релиза.

**Закрытие ветви релиза**  
Когда мы решаем, что ветвь релиза (release branch) окончательно готова для выпуска, нужно проделать несколько действий. В первую очередь ветвь релиза вливается в главную ветвь (напоминаю, каждый коммит в **main** — это по определению новый релиз). Далее, этот коммит в **main** должен быть помечен тегом, чтобы в дальнейшем можно было легко обратиться к любой существовавшей версии продукта. И наконец, изменения, сделанные в ветви релиза (release branch), должны быть добавлены обратно в разработку (ветвь **develop**), чтобы будущие релизы также содержали внесённые исправления багов.

Первые два шага:
```
$ git checkout main
Switched to branch 'main'
$ git merge --no-ff release-1.2
Merge made by recursive.
$ git tag -a 1.2
```
Теперь релиз издан и помечен тегом.

Чтобы сохранить изменения и в последующих релизах, мы должны влить эти изменения обратно в разработку. Делаем это так:
```
$ git checkout develop
Switched to branch 'develop'
$ git merge --no-ff release-1.2
Merge made by recursive.
(Отчёт об изменениях)
```
Этот шаг, в принципе, может привести к конфликту слияния (нередко бывает, что причиной конфликта является изменение номера версии проекта). Если это произошло, исправьте их и издайте коммит.

Теперь мы окончательно разделились с веткой релиза. Можно её удалять, потому что она нам больше не понадобится:
```
$ git branch -d release-1.2
Deleted branch release-1.2 (was ff452fe).
```

### Ветви исправлений (hotfix branches)  
***Могут порождаться от: main***  
***Должны вливаться в: develop и main***  
***Соглашение о наименовании: hotfix-\****    

Ветви для исправлений (hotfix branches) похожи на ветви релизов (release branches), так как они тоже используются для подготовки новых выпусков продукта, разве лишь незапланированных. Они порождаются необходимостью немедленно исправить нежелательное поведение производственной версии продукта. Когда в производственной версии находится баг, требующий немедленного исправления, из соответствующего данной версии тега главной ветви (**main**) порождается новая ветвь для работы над исправлением.

Смысл её существования состоит в том, что работа команды над ветвью разработки (**develop**) может спокойно продолжаться, в то время как кто-то один готовит быстрое исправление производственной версии.

**Создание ветви исправлений (hotfix branch)**
Ветви исправлений (hotfix branches) создаются из главной (**main**) ветви. Пускай, например, текущий производственный релиз имеет версию 1.2, и в нём (внезапно!) обнаруживается серьёзный баг. А изменения в ветви разработки (**develop**) ещё недостаточно стабильны, чтобы их издавать в новый релиз. Но мы можем создать новую ветвь исправлений и начать работать над решением проблемы:
```
$ git checkout -b hotfix-1.2.1 main
Switched to a new branch "hotfix-1.2.1"
$ ./bump-version.sh 1.2.1
Files modified successfully, version bumped to 1.2.1.
$ git commit -a -m "Bumped version number to 1.2.1"
[hotfix-1.2.1 41e61bb] Bumped version number to 1.2.1
1 files changed, 1 insertions(+), 1 deletions(-)
```
Не забывайте обновлять номер версии после создания ветви!

Теперь можно исправлять баг, а изменения издавать хоть одним коммитом, хоть несколькими.
```
$ git commit -m "Fixed severe production problem"
[hotfix-1.2.1 abbe5d6] Fixed severe production problem
5 files changed, 32 insertions(+), 17 deletions(-)
```
**Закрытие ветви исправлений**
Когда баг исправлен, изменения надо влить обратно в главную ветвь (**main**), а также в ветвь разработки (**develop**), чтобы гарантировать, что это исправление окажется и в следующем релизе. Это очень похоже на то, как закрывается ветвь релиза (release branch).

Прежде всего надо обновить главную ветвь (**main**) и пометить новую версию тегом.
```
$ git checkout main
Switched to branch 'main'
$ git merge --no-ff hotfix-1.2.1
Merge made by recursive.
$ git tag -a 1.2.1
```  
Следующим шагом переносим исправление в ветвь разработки (**develop**).
```
$ git checkout develop
Switched to branch 'develop'
$ git merge --no-ff hotfix-1.2.1
Merge made by recursive.
```

У этого правила есть одно исключение: если в данный момент существует ветвь релиза (release branch), то ветвь исправления (hotfix branch) должна вливаться в неё, а не в ветвь разработки (**develop**). В этом случае исправления войдут в ветвь разработки вместе со всей ветвью релиза, когда та будет закрыта. (Хотя, если работа в develop требует немедленного исправления бага и не может ждать, пока будет завершено издание текущего релиза, Вы всё же можете влить исправления (bugfix) в ветвь разработки (**develop**), и это будет вполне безопасно).

И наконец, удаляем временную ветвь:
```
$ git branch -d hotfix-1.2.1
Deleted branch hotfix-1.2.1 (was abbe5d6).
```
