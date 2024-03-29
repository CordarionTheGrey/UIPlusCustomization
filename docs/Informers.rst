============================
 Пользовательские информеры
============================

Информеры — средство оповещения о том, что герой попал в некую важную ситуацию. В eGUI+ бог может
тонко настроить, что именно считать «важными ситуациями». Окно настройки вызывается нажатием
на шестеренку в заголовке блока «Герой».

Информер состоит из трех частей: *заголовка*, *условия* и *типа*.

Условие — ключевая часть: оно определяет, что информер отслеживает. В условии проверяются значения
*переменных* (например, ``gv.health < 100``). Полный перечень переменных приведен в FAQ_ («Какие
операции и переменные доступны при написании правил пользовательских информеров?»).

Заголовок — это текст, который будет выдаваться при срабатывании информера. Может содержать
переменные и выражения в фигурных скобках (``{…}``).

Тип — набор букв, говорящий, что нужно сделать при срабатывании. Некоторые основные типы: ``L``
(повесить плашку в углу экрана), ``D`` (показать всплывающее уведомление), ``A`` (подать звуковой
сигнал). Если тип информера оставлен пустым, то он берется из `общих настроек`_ (по умолчанию —
``LD``). Все типы перечислены в FAQ_ («Какие бывают типы пользовательских информеров?»).

.. _FAQ: https://gv.erinome.net/godville/?show=FAQ
.. _общих настроек: https://godville.net/user/profile#ui_settings


------------------------------
 Некоторые полезные информеры
------------------------------

Выписанные ниже информеры постепенно формировались за время моего пребывания в Годвилле. Не
стесняйтесь брать их и дорабатывать под свои нужды.

.. [[[cog
    import informers
    informers.generate(R"""
    - desc: >-
        Гласовый. Срабатывает в поле в перерывах между сражениями с монстрами — тогда, когда можно
        что-нибудь покричать герою. Конечно же, большую часть времени этот информер стоит держать
        выключенным, включая по необходимости, иначе он вас заспамит.
      title: Бой окончен
      expr: >-
        !gv.currentMonster && !gv.inTown && !gv.inFight && !gv.voiceCooldown && gv.lastNews !~ "Переводит дух|Заканчивает перекур" && gv.mileStones && (!gv.isGoingForth || gv.questProgress <= 2) && !gv.isGoingBack && !gv.isFishing && !gv.isTrading && gv.health && gv.questName !~ " \\((?:выполнено|отменено)\\)"

    - desc: >-
        Возврат в город с рюкзаком, заполненным на 33% и более, — есть шанс, что герой идет
        на полный цикл, а не только на лечение.
      title: → {gv.currentTown || gv.nearestTown}
      expr: >-
        (gv.isGoingBack || gv.inTown) && gv.inventoryPrc >= 33 && gv.questName !~ " \\((?:выполнено|отменено)\\)" && !gv.isGoingToGodville && gv.currentTown != "Годвилль"
      mode: DQ

    - desc: >-
        Герой решил сходить в Годвилль, столицу Годвилля.
      title: ⇒ Годвилль
      expr: >-
        gv.isGoingToGodville && gv.questName !~ " \\((?:выполнено|отменено)\\)"
      mode: LDS

    - desc: >-
        Задание выполнено на 99%, и есть прана на то, чтобы что-нибудь сделать, пока не поздно.
      title: Задание почти выполнено
      expr: >-
        gv.questProgress >= 99 && gv.godpower >= 5 && gv.questName !~ " \\((?:мини|гильд|выполнено|отменено)\\)"

    - desc: >-
        Задание выполнено; герой идет в столицу.
      title: >-
        Задание выполнено: {gv.questName ~ ".*(?= \\(выполнено\\))"}
      expr: >-
        gv.questName ~ " \\(выполнено\\)" && gv.mileStones

    - desc: >-
        Полная прана; только вне дуэльных режимов.
      title: >-
        Прана: gv.godpower%
      expr: >-
        gv.godpowerPrc == 100 && !gv.inFight
      mode: D

    - desc: >-
        Здоровье в поле упало до 18 HP, но герой не убегает от монстра. Высок риск умереть
        в ближайшую минуту.
      title: При смерти! (gv.health HP)
      expr: >-
        gv.currentMonster && gv.health <= 18 && gv.auraName != "бессмертия" && (gv.health <= 12 || !gv.isEquipmentBold(2)) && !gv.isEquipmentBold(7)
      mode: LDRW

    - desc: >-
        Герой встретил бродячего торговца.
      title: Бродячий торговец
      expr: >-
        gv.isTrading && !gv.inTown

    - desc: >-
        Герой сел рыбачить.
      title: Рыбалка
      expr: >-
        gv.isFishing
      mode: LN

    - desc: >-
        Можно пойти в подземелье.
      title: Подземелье (gv.healthPrc% HP)
      expr: >-
        gv.healthPrc >= 67 && gv.dungeonAvailable && gv.logTimeout <= 10 && gv.godpower >= 85 && gv.charges >= 2
      mode: LDW

    - desc: >-
        Можно пойти в подземелье с пьянки в городе.
      title: Торговля заканчивается
      expr: >-
        gv.inventory <= gv.inventoryUnsellable + 1 && gv.inTown && !gv.inShop && gv.dungeonAvailable && gv.godpower >= 85 && gv.charges >= 2

    - desc: >-
        Можно пойти в море.
      title: Море
      expr: >-
        gv.sailAvailable && gv.godpower >= 70

    - desc: >-
        Можно пойти на полигон.
      title: Полигон
      expr: >-
        gv.miningAvailable && gv.byteTimeout <= 13 && gv.godpower >= 80

    - desc: >-
        Расчет времени на городской цикл. Спасибо богу `Ржавому Унитазу
        <https://godville.net/gods/Ржавый%20Унитаз>`_ за идею и прототип.
      title: >-
        {gv.healthPrc == 100 ? "" : "Л: " + Math.ceil((gv.healthMax - gv.health) / ((gv.currentTown ~ "Пустосвятово|Геролимп|Боянск" ? 21 : gv.currentTown ~ "ЛосАдминос|Малые Саппортуны|Воскрешанск|Баговещенск" ? 28 : gv.currentTown == "Скрежетальск" ? 5.6 : gv.currentTown == "Еретиченск" ? 10.5 : gv.currentTown == "Ёмаёво" ? 7 : gv.currentTown ~ "Новогодвилль|Заброшенск" ? 42 : 14) * (gv.isEquipmentBold(4) ? 1.6 : 1)) - 1e-6) + " мин."}{gv.inventory - gv.inventoryUnsellable <= 1 ? "" : (gv.healthPrc == 100 ? "" : ", ") + "Т: " + Math.ceil((gv.inventory - gv.inventoryUnsellable) * .733) + " мин."}
      expr: >-
        (gv.inTown || gv.isGoingBack || gv.isFishing) && (gv.healthPrc < 100 || gv.inventory > gv.inventoryUnsellable + 1)
      mode: LNW

    - desc: >-
        Босс может убить героя с одного удара.
      title: Пора лечиться (gv.health HP)
      expr: >-
        gv.lowHealth && (gv.fightType == "monster" || gv.fightType == "multi_monster") && (gv.godpower >= 5 || gv.charges >= 1)
      mode: LDAW

    - desc: >-
        У босса осталось мало здоровья. Срабатывает, только если вкладка браузера неактивна.
        К сожалению, нет возможности отличить подземного босса от рейдового, так что этот информер
        срабатывает на любых.
      title: Пора рулить
      expr: >-
        !gv.windowFocused && gv.enemyAbilitiesCount && gv.alliesCount && gv.enemyHealth <= (gv.healthMax + gv.alliesAliveHealthMax) * .08 && gv.health > 1 && (gv.godpower >= 5 || gv.charges >= 1)

    - desc: >-
        Счетчик гласов в подземелье со спецзаданием на соблюдение тишины.
      title: >-
        Гласы: {8 - gv.guidedStepsCount}
      expr: >-
        gv.dungeonChallenge == "silence" && gv.guidedStepsCount <= 8
      mode: LNW

    - desc: >-
        Встречен монстр, из которого выпадает трофей, дающий прану или заряд.
      title: Дарующий монстр
      expr: >-
        gv.currentMonster ~* "Пророк бога монстров|Лежебог|Дарующий|Юбилейный" && gv.godpower >= 5
      mode: LDAS

    - desc: >-
        Встречен монстр, которого, возможно, стоит помочь убить.
      title: gv.currentMonster
      expr: >-
        (gv.wantedMonster || gv.specialMonster || gv.currentMonster ~* "Андед[- ]Мороз|Сатан[- ]Клаус") && gv.godpower >= 25

    - desc: >-
        Взят мини-квест.
      title: >-
        Мини-квест: {gv.questName ~ ".*(?= \\(мини\\))"}
      expr: >-
        gv.questName ~ " \\(мини\\)"
      mode: LDNQ

    - desc: >-
        Мини-квест выполнен.
      title: Мини-квест выполнен
      expr: >-
        gv.questName !~ " \\(мини\\)"

    - desc: >-
        Можно вырезать из газеты купон.
      title: Купон на gv.couponPrize
      expr: >-
        gv.couponPrize && !gv.inFight
      mode: LNW

    - desc: >-
        Получена аура, за исключением «неинтересных».
      title: Аура gv.auraName
      expr: >-
        gv.auraName !~ "^$|бессмертия|вещизма|конфликта|охоты|полураспада|розыска|рыбалки"
      mode: DW

    - desc: >-
        Закончилась аура непереносимости. Как ни странно, это довольно полезное условие.
      title: Аура закончилась (непереносимость)
      expr: >-
        gv.auraName != "непереносимости"

    - desc: >-
        Герой-торговец вышел из своей лавки.
      title: Героиня вышла из лавки
      expr: >-
        !gv.inShop

    - desc: >-
        Герой вступил в бой с боссом (один на один) или бандой. Обычно такое происходит неожиданно.
      title: На героиню напали
      expr: >-
        (gv.fightType == "monster" || gv.fightType == "multi_monster") && !gv.alliesCount

    - desc: >-
        Много трофеев, подходящих в бинго; либо осталось 5 минут до крайнего срока сдачи
        (00:05 MSK).
      title: >-
        Бинго: gv.bingoItems/gv.bingoSlotsLeft (gv.bingoTriesLeft)
      expr: >-
        gv.bingoTriesLeft && !gv.inFight && ((gv.bingoItems && gv.bingoItems * gv.bingoTriesLeft >= gv.bingoSlotsLeft - 2) || (!gv.getHoursMSK && gv.getMinutes <= 4))
      mode: LDW

    - desc: >-
        Пришел рифмованный глас чужого бога. Они бывают забавны.
      title: >-
        🔉 {gv.lastDiaryVoice.slice(gv.lastDiaryVoice.indexOf("\n") + 5, gv.lastDiaryVoice.lastIndexOf("(") - 1)}
      expr: >-
        gv.lastDiaryVoice ~ "\n"
      mode: LDNSW

    - desc: >-
        Пришел глас, и не хватает 1% праны до «круглого» числа.
      title: Пришел глас
      expr: >-
        gv.lastDiaryVoice && gv.lastDiaryVoice !~ "\n" && (([4, 49, 99, 149, 199]).includes(gv.godpower) || (Math.max(gv.miningSendDelay / 60, gv.byteTimeout - 13) <= 40 && ([64, 79, 94]).includes(gv.godpower)))

    - desc: >-
        Встречен сильный монстр во время соответствующего подряда.
      title: Сильный gv.currentMonster
      expr: >-
        gv.strongMonster && gv.sideJobDuration && gv.sideJobName ~ "сильн" && gv.sideJobProgress < 100 && gv.godpower >= 25

    - desc: >-
        Найден трофей, дающий ауру, а соответствующий подряд выполнен хотя бы на 50%.
      title: Аурный трофей
      expr: >-
        gv.sideJobDuration && gv.sideJobName ~ "аур" && gv.sideJobProgress >= 50 && gv.sideJobProgress < 100 && gv.inventoryHasType("aura-box") && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 25 : 50) - (gv.godpowerCapAvailable && 12)

    - desc: >-
        Счетчик лечебных трофеев в инвентаре при соответствующем подряде.
      title: >-
        Зелья: gv.inventoryHealing
      expr: >-
        gv.sideJobDuration && gv.sideJobProgress < 100 && gv.sideJobName ~ "лечил"
      mode: LNW

    - desc: >-
        Закончились лечебные трофеи при соответствующем подряде (пора идти в город за новыми).
      title: Зелья кончились (рядом gv.nearestTown)
      expr: >-
        !gv.inTown && gv.inventoryHealing <= 1 && gv.godpower >= 5 && gv.sideJobDuration && gv.sideJobProgress < 100 && gv.sideJobName ~ "лечил" && gv.questName !~ " \\((?:мини|гильд)\\)" && ((!gv.inventoryHealing && !gv.currentMonster) || (gv.inventoryHasType("quest-box") && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 30 : 55)))
      mode: LDW

    - desc: >-
        Можно подать божественный знак на раскопки босса или прыжок по спирали.
      title: Можно подать знак (в поле)
      expr: >-
        !gv.inTown && !gv.currentMonster && gv.lastDiary ~ "\\(@\\)"
      mode: LDS

    - desc: >-
        Можно подать божественный знак на допрос монстра.
      title: Можно подать знак (монстр)
      expr: >-
        gv.currentMonster && gv.lastDiary ~ "\\(@\\)"
      mode: LDS

    - desc: >-
        Можно подать божественный знак на отказ от трат или особую молитву.
      title: Можно подать знак (в городе)
      expr: >-
        gv.inTown && gv.lastDiary ~ "\\(@\\)"
      mode: LDS

    - desc: >-
        В рюкзаке есть алхимический превращатель и хотя бы 4 других жирных трофея, которые не жалко
        превратить в кирпичи.
      title: Трансмутатор ({gv.inventoryCountLike("^(?!золотой кирпич|босскоин)", "b") - 1})
      expr: >-
        !gv.inFight && gv.inventoryHasType("transformer") && gv.inventoryCountLike("^(?!золотой кирпич|босскоин)", "b") >= 5 && !gv.inventoryCountLike("бесценный дар|старую шмотку|(?:сердце|глаз) босса |бонус за подряд|призовой сундук|пасхал(?:ку|ьное яйцо)|(?:золотую|светящуюся) тыкву|^заморск.. |морскую (?:джемчужину|златоустрицу|суперзвезду)|морской приз|(?:ларец|сундучок|ящик) из моря")
      mode: LDW

    - desc: >-
        Найден пранозапаковывающий трофей, и на его активацию хватает праны.
      title: Аккумуляторный трофей
      expr: >-
        gv.inventoryCountLike("слезинку бога в янтаре|средство от обезбоживания") || (gv.inventoryHasType("charge-box") && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 25 : 50) - (gv.godpowerCapAvailable && 12))

    - desc: >-
        Найден пранопополняющий трофей. Не срабатывает в столице на выполненном квесте — если этот
        трофей выпал из заморской активашки, его не обязательно использовать сразу же, ведь торговли
        не будет.
      title: Славительное
      expr: >-
        (!gv.inTown || gv.questName !~ " \\((?:выполнено|отменено)\\)") && gv.inventoryHasType("godpower-box")

    - desc: >-
        В рюкзаке есть 2 жирных трофея на «а».
      title: Крафтим алоэ веры
      expr: >-
        !gv.inShop && gv.inventoryCountLike("^[^а-яё]*а(?!лоэ веры)", "bc") >= 2 && !gv.inFight && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 30 : 55)

    - desc: >-
        В рюкзаке есть 2 жирных трофея на «б».
      title: Крафтим божью коробку
      expr: >-
        !gv.inShop && gv.inventoryCountLike("^[^а-яё]*б(?!ожью коробку|есценный дар|огомазь)", "bc") >= 2 && !gv.inFight && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 30 : 55)

    - desc: >-
        В рюкзаке есть 2 жирных трофея на «в».
      title: Крафтим веротренажёр
      expr: >-
        !gv.inShop && gv.inventoryCountLike("^[^а-яё]*в(?!еротренаж[её]р)", "bc") >= 2 && !gv.inFight && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 30 : 55)

    - desc: >-
        В рюкзаке есть 2 жирных трофея на «о».
      title: Крафтим освятительный прибор
      expr: >-
        !gv.inShop && gv.inventoryCountLike("^[^а-яё]*о(?!святительный прибор)", "bc") >= 2 && !gv.inFight && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 30 : 55)
    """)
    ]]]
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Гласовый. Срабатывает в поле в перерывах между сражениями с монстрами — тогда, когда можно что-нибудь покричать герою. Конечно же, большую часть времени этот информер стоит держать выключенным, включая по необходимости, иначе он вас заспамит.
    * - Заголовок
      - ``Бой окончен``
    * - Условие
      - ``!gv.currentMonster && !gv.inTown && !gv.inFight && !gv.voiceCooldown && gv.lastNews !~ "Переводит дух|Заканчивает перекур" && gv.mileStones && (!gv.isGoingForth || gv.questProgress <= 2) && !gv.isGoingBack && !gv.isFishing && !gv.isTrading && gv.health && gv.questName !~ " \\((?:выполнено|отменено)\\)"``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Возврат в город с рюкзаком, заполненным на 33% и более, — есть шанс, что герой идет на полный цикл, а не только на лечение.
    * - Заголовок
      - ``→ {gv.currentTown || gv.nearestTown}``
    * - Условие
      - ``(gv.isGoingBack || gv.inTown) && gv.inventoryPrc >= 33 && gv.questName !~ " \\((?:выполнено|отменено)\\)" && !gv.isGoingToGodville && gv.currentTown != "Годвилль"``
    * - Тип
      - ``DQ``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Герой решил сходить в Годвилль, столицу Годвилля.
    * - Заголовок
      - ``⇒ Годвилль``
    * - Условие
      - ``gv.isGoingToGodville && gv.questName !~ " \\((?:выполнено|отменено)\\)"``
    * - Тип
      - ``LDS``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Задание выполнено на 99%, и есть прана на то, чтобы что-нибудь сделать, пока не поздно.
    * - Заголовок
      - ``Задание почти выполнено``
    * - Условие
      - ``gv.questProgress >= 99 && gv.godpower >= 5 && gv.questName !~ " \\((?:мини|гильд|выполнено|отменено)\\)"``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Задание выполнено; герой идет в столицу.
    * - Заголовок
      - ``Задание выполнено: {gv.questName ~ ".*(?= \\(выполнено\\))"}``
    * - Условие
      - ``gv.questName ~ " \\(выполнено\\)" && gv.mileStones``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Полная прана; только вне дуэльных режимов.
    * - Заголовок
      - ``Прана: gv.godpower%``
    * - Условие
      - ``gv.godpowerPrc == 100 && !gv.inFight``
    * - Тип
      - ``D``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Здоровье в поле упало до 18 HP, но герой не убегает от монстра. Высок риск умереть в ближайшую минуту.
    * - Заголовок
      - ``При смерти! (gv.health HP)``
    * - Условие
      - ``gv.currentMonster && gv.health <= 18 && gv.auraName != "бессмертия" && (gv.health <= 12 || !gv.isEquipmentBold(2)) && !gv.isEquipmentBold(7)``
    * - Тип
      - ``LDRW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Герой встретил бродячего торговца.
    * - Заголовок
      - ``Бродячий торговец``
    * - Условие
      - ``gv.isTrading && !gv.inTown``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Герой сел рыбачить.
    * - Заголовок
      - ``Рыбалка``
    * - Условие
      - ``gv.isFishing``
    * - Тип
      - ``LN``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Можно пойти в подземелье.
    * - Заголовок
      - ``Подземелье (gv.healthPrc% HP)``
    * - Условие
      - ``gv.healthPrc >= 67 && gv.dungeonAvailable && gv.logTimeout <= 10 && gv.godpower >= 85 && gv.charges >= 2``
    * - Тип
      - ``LDW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Можно пойти в подземелье с пьянки в городе.
    * - Заголовок
      - ``Торговля заканчивается``
    * - Условие
      - ``gv.inventory <= gv.inventoryUnsellable + 1 && gv.inTown && !gv.inShop && gv.dungeonAvailable && gv.godpower >= 85 && gv.charges >= 2``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Можно пойти в море.
    * - Заголовок
      - ``Море``
    * - Условие
      - ``gv.sailAvailable && gv.godpower >= 70``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Можно пойти на полигон.
    * - Заголовок
      - ``Полигон``
    * - Условие
      - ``gv.miningAvailable && gv.byteTimeout <= 13 && gv.godpower >= 80``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Расчет времени на городской цикл. Спасибо богу `Ржавому Унитазу <https://godville.net/gods/Ржавый%20Унитаз>`_ за идею и прототип.
    * - Заголовок
      - ``{gv.healthPrc == 100 ? "" : "Л: " + Math.ceil((gv.healthMax - gv.health) / ((gv.currentTown ~ "Пустосвятово|Геролимп|Боянск" ? 21 : gv.currentTown ~ "ЛосАдминос|Малые Саппортуны|Воскрешанск|Баговещенск" ? 28 : gv.currentTown == "Скрежетальск" ? 5.6 : gv.currentTown == "Еретиченск" ? 10.5 : gv.currentTown == "Ёмаёво" ? 7 : gv.currentTown ~ "Новогодвилль|Заброшенск" ? 42 : 14) * (gv.isEquipmentBold(4) ? 1.6 : 1)) - 1e-6) + " мин."}{gv.inventory - gv.inventoryUnsellable <= 1 ? "" : (gv.healthPrc == 100 ? "" : ", ") + "Т: " + Math.ceil((gv.inventory - gv.inventoryUnsellable) * .733) + " мин."}``
    * - Условие
      - ``(gv.inTown || gv.isGoingBack || gv.isFishing) && (gv.healthPrc < 100 || gv.inventory > gv.inventoryUnsellable + 1)``
    * - Тип
      - ``LNW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Босс может убить героя с одного удара.
    * - Заголовок
      - ``Пора лечиться (gv.health HP)``
    * - Условие
      - ``gv.lowHealth && (gv.fightType == "monster" || gv.fightType == "multi_monster") && (gv.godpower >= 5 || gv.charges >= 1)``
    * - Тип
      - ``LDAW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - У босса осталось мало здоровья. Срабатывает, только если вкладка браузера неактивна. К сожалению, нет возможности отличить подземного босса от рейдового, так что этот информер срабатывает на любых.
    * - Заголовок
      - ``Пора рулить``
    * - Условие
      - ``!gv.windowFocused && gv.enemyAbilitiesCount && gv.alliesCount && gv.enemyHealth <= (gv.healthMax + gv.alliesAliveHealthMax) * .08 && gv.health > 1 && (gv.godpower >= 5 || gv.charges >= 1)``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Счетчик гласов в подземелье со спецзаданием на соблюдение тишины.
    * - Заголовок
      - ``Гласы: {8 - gv.guidedStepsCount}``
    * - Условие
      - ``gv.dungeonChallenge == "silence" && gv.guidedStepsCount <= 8``
    * - Тип
      - ``LNW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Встречен монстр, из которого выпадает трофей, дающий прану или заряд.
    * - Заголовок
      - ``Дарующий монстр``
    * - Условие
      - ``gv.currentMonster ~* "Пророк бога монстров|Лежебог|Дарующий|Юбилейный" && gv.godpower >= 5``
    * - Тип
      - ``LDAS``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Встречен монстр, которого, возможно, стоит помочь убить.
    * - Заголовок
      - ``gv.currentMonster``
    * - Условие
      - ``(gv.wantedMonster || gv.specialMonster || gv.currentMonster ~* "Андед[- ]Мороз|Сатан[- ]Клаус") && gv.godpower >= 25``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Взят мини-квест.
    * - Заголовок
      - ``Мини-квест: {gv.questName ~ ".*(?= \\(мини\\))"}``
    * - Условие
      - ``gv.questName ~ " \\(мини\\)"``
    * - Тип
      - ``LDNQ``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Мини-квест выполнен.
    * - Заголовок
      - ``Мини-квест выполнен``
    * - Условие
      - ``gv.questName !~ " \\(мини\\)"``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Можно вырезать из газеты купон.
    * - Заголовок
      - ``Купон на gv.couponPrize``
    * - Условие
      - ``gv.couponPrize && !gv.inFight``
    * - Тип
      - ``LNW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Получена аура, за исключением «неинтересных».
    * - Заголовок
      - ``Аура gv.auraName``
    * - Условие
      - ``gv.auraName !~ "^$|бессмертия|вещизма|конфликта|охоты|полураспада|розыска|рыбалки"``
    * - Тип
      - ``DW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Закончилась аура непереносимости. Как ни странно, это довольно полезное условие.
    * - Заголовок
      - ``Аура закончилась (непереносимость)``
    * - Условие
      - ``gv.auraName != "непереносимости"``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Герой-торговец вышел из своей лавки.
    * - Заголовок
      - ``Героиня вышла из лавки``
    * - Условие
      - ``!gv.inShop``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Герой вступил в бой с боссом (один на один) или бандой. Обычно такое происходит неожиданно.
    * - Заголовок
      - ``На героиню напали``
    * - Условие
      - ``(gv.fightType == "monster" || gv.fightType == "multi_monster") && !gv.alliesCount``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Много трофеев, подходящих в бинго; либо осталось 5 минут до крайнего срока сдачи (00:05 MSK).
    * - Заголовок
      - ``Бинго: gv.bingoItems/gv.bingoSlotsLeft (gv.bingoTriesLeft)``
    * - Условие
      - ``gv.bingoTriesLeft && !gv.inFight && ((gv.bingoItems && gv.bingoItems * gv.bingoTriesLeft >= gv.bingoSlotsLeft - 2) || (!gv.getHoursMSK && gv.getMinutes <= 4))``
    * - Тип
      - ``LDW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Пришел рифмованный глас чужого бога. Они бывают забавны.
    * - Заголовок
      - ``🔉 {gv.lastDiaryVoice.slice(gv.lastDiaryVoice.indexOf("\n") + 5, gv.lastDiaryVoice.lastIndexOf("(") - 1)}``
    * - Условие
      - ``gv.lastDiaryVoice ~ "\n"``
    * - Тип
      - ``LDNSW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Пришел глас, и не хватает 1% праны до «круглого» числа.
    * - Заголовок
      - ``Пришел глас``
    * - Условие
      - ``gv.lastDiaryVoice && gv.lastDiaryVoice !~ "\n" && (([4, 49, 99, 149, 199]).includes(gv.godpower) || (Math.max(gv.miningSendDelay / 60, gv.byteTimeout - 13) <= 40 && ([64, 79, 94]).includes(gv.godpower)))``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Встречен сильный монстр во время соответствующего подряда.
    * - Заголовок
      - ``Сильный gv.currentMonster``
    * - Условие
      - ``gv.strongMonster && gv.sideJobDuration && gv.sideJobName ~ "сильн" && gv.sideJobProgress < 100 && gv.godpower >= 25``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Найден трофей, дающий ауру, а соответствующий подряд выполнен хотя бы на 50%.
    * - Заголовок
      - ``Аурный трофей``
    * - Условие
      - ``gv.sideJobDuration && gv.sideJobName ~ "аур" && gv.sideJobProgress >= 50 && gv.sideJobProgress < 100 && gv.inventoryHasType("aura-box") && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 25 : 50) - (gv.godpowerCapAvailable && 12)``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Счетчик лечебных трофеев в инвентаре при соответствующем подряде.
    * - Заголовок
      - ``Зелья: gv.inventoryHealing``
    * - Условие
      - ``gv.sideJobDuration && gv.sideJobProgress < 100 && gv.sideJobName ~ "лечил"``
    * - Тип
      - ``LNW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Закончились лечебные трофеи при соответствующем подряде (пора идти в город за новыми).
    * - Заголовок
      - ``Зелья кончились (рядом gv.nearestTown)``
    * - Условие
      - ``!gv.inTown && gv.inventoryHealing <= 1 && gv.godpower >= 5 && gv.sideJobDuration && gv.sideJobProgress < 100 && gv.sideJobName ~ "лечил" && gv.questName !~ " \\((?:мини|гильд)\\)" && ((!gv.inventoryHealing && !gv.currentMonster) || (gv.inventoryHasType("quest-box") && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 30 : 55)))``
    * - Тип
      - ``LDW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Можно подать божественный знак на раскопки босса или прыжок по спирали.
    * - Заголовок
      - ``Можно подать знак (в поле)``
    * - Условие
      - ``!gv.inTown && !gv.currentMonster && gv.lastDiary ~ "\\(@\\)"``
    * - Тип
      - ``LDS``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Можно подать божественный знак на допрос монстра.
    * - Заголовок
      - ``Можно подать знак (монстр)``
    * - Условие
      - ``gv.currentMonster && gv.lastDiary ~ "\\(@\\)"``
    * - Тип
      - ``LDS``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Можно подать божественный знак на отказ от трат или особую молитву.
    * - Заголовок
      - ``Можно подать знак (в городе)``
    * - Условие
      - ``gv.inTown && gv.lastDiary ~ "\\(@\\)"``
    * - Тип
      - ``LDS``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - В рюкзаке есть алхимический превращатель и хотя бы 4 других жирных трофея, которые не жалко превратить в кирпичи.
    * - Заголовок
      - ``Трансмутатор ({gv.inventoryCountLike("^(?!золотой кирпич|босскоин)", "b") - 1})``
    * - Условие
      - ``!gv.inFight && gv.inventoryHasType("transformer") && gv.inventoryCountLike("^(?!золотой кирпич|босскоин)", "b") >= 5 && !gv.inventoryCountLike("бесценный дар|старую шмотку|(?:сердце|глаз) босса |бонус за подряд|призовой сундук|пасхал(?:ку|ьное яйцо)|(?:золотую|светящуюся) тыкву|^заморск.. |морскую (?:джемчужину|златоустрицу|суперзвезду)|морской приз|(?:ларец|сундучок|ящик) из моря")``
    * - Тип
      - ``LDW``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Найден пранозапаковывающий трофей, и на его активацию хватает праны.
    * - Заголовок
      - ``Аккумуляторный трофей``
    * - Условие
      - ``gv.inventoryCountLike("слезинку бога в янтаре|средство от обезбоживания") || (gv.inventoryHasType("charge-box") && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 25 : 50) - (gv.godpowerCapAvailable && 12))``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - Найден пранопополняющий трофей. Не срабатывает в столице на выполненном квесте — если этот трофей выпал из заморской активашки, его не обязательно использовать сразу же, ведь торговли не будет.
    * - Заголовок
      - ``Славительное``
    * - Условие
      - ``(!gv.inTown || gv.questName !~ " \\((?:выполнено|отменено)\\)") && gv.inventoryHasType("godpower-box")``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - В рюкзаке есть 2 жирных трофея на «а».
    * - Заголовок
      - ``Крафтим алоэ веры``
    * - Условие
      - ``!gv.inShop && gv.inventoryCountLike("^[^а-яё]*а(?!лоэ веры)", "bc") >= 2 && !gv.inFight && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 30 : 55)``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - В рюкзаке есть 2 жирных трофея на «б».
    * - Заголовок
      - ``Крафтим божью коробку``
    * - Условие
      - ``!gv.inShop && gv.inventoryCountLike("^[^а-яё]*б(?!ожью коробку|есценный дар|огомазь)", "bc") >= 2 && !gv.inFight && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 30 : 55)``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - В рюкзаке есть 2 жирных трофея на «в».
    * - Заголовок
      - ``Крафтим веротренажёр``
    * - Условие
      - ``!gv.inShop && gv.inventoryCountLike("^[^а-яё]*в(?!еротренаж[её]р)", "bc") >= 2 && !gv.inFight && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 30 : 55)``
.. list-table::
    :stub-columns: 1
    :widths: 1 50

    * - Описание
      - В рюкзаке есть 2 жирных трофея на «о».
    * - Заголовок
      - ``Крафтим освятительный прибор``
    * - Условие
      - ``!gv.inShop && gv.inventoryCountLike("^[^а-яё]*о(?!святительный прибор)", "bc") >= 2 && !gv.inFight && gv.godpower >= (gv.isForecast("lowpoweractivatables") ? 30 : 55)``
.. [[[end]]] (checksum: db788dae87d46ba0289b60df93ea4c22)
