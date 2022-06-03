# Условие (Overview)
Дан смарт-контракт `KittenRegistry`, содержащий уникальные идентификаторы котов (`catID`) и отслеживающий их владельцев. Исходники приаттачены.  

Необходимо написать смарт-контракт питомника `KittyShelter`, умеющий принимать кота “на хранение” на определенный период времени. В результате, на период времени `<time>` (секунд) с момента вызова, владельцем кота становится смарт-контракт `KittyShelter`. Пользователь может забрать своего кота только по истечении времени `<time>`. Никто, кроме исходного владельца, не должен иметь возможности забрать чужого кота.

# Требования (Requirements)
1.	Смарт-контракт `KittyShelter` должен взаимодействовать с одним `KittenRegistry` по указанному адресу (адрес можно захардкодить или указать в конструкторе).
2.	У смарт-контракта должно быть две (обязательных) функции с такой сигнатурой:  
`function storeKitty(uint256 catId, uint256 time) public`  
`function retrieveKitty(uint256 catId) public`
3.	Один пользователь может оставить неограниченное количество котов в `KittyShelter`.
4.	Пользователь не может забрать кота раньше срока.
5.	Кот остается у смарт-контракта до тех пор, пока его не заберут. Пользователь должен сам “прийти” за котом, вызвав функцию `retrieveKitty`.
6.	Забрать кота может только тот пользователь, который его отдал.

# Формат сдачи
Результатом проделанной работы будет исходный код смарт-контракта “KittyShelter”. Как бонус, можно предоставить адрес рабочего контракта в тестовой сети `Ropsten`.

# Тестовые данные (Use case)
Боб уезжает в отпуск и хочет оставить своего кота Барсика (`catId=0x01`) в приюте (`KittyShelter address=0x010c`) на 2 недели. Для этого Боб:
1.	Разрешает приюту забрать у него кота вызовом `KittenRegistry.approve(0x010c, 0x01)`
2.	Обращается в приют, чтобы тот забрал у него кота `KittyShelter.storeKitty(0x01, 14 days)`
3.	Смарт-контракт приюта переводит себе кота Боба. Теперь кот безопасно хранится в приюте до истечения 14 дней с момента отъезда Боба.
4.	Через 14 дней, Боб обращается к приюту за своим котом `KittyShelter.retrieveKitty(0x01)`. Смарт-контракт отправляет Барсика обратно Бобу.
    - Боб вернулся раньше и хочет забрать кота - приют не отдает кота, пока не истек срок хранения.
    - Алиса решила попытаться украсть Барсика, обратившись в приют - приют не отдает кота чужим людям.

# Примечания (Tips/Notes)
**catID** - простой пример неделимой (`non-fungible`) монеты соответствующей стандарту **ERC-721**, почитать про который можно тут http://erc721.org/  

Для проверки работы смарт-контрактов в сети `Ropsten` существует смарт-контракт `PetShop`. У него всего одна функция: `PetShop.BuyAKitten()`. Функция не принимает параметров, но при её вызове нужно оплатить 0.5 Ether (за кота). В результате её выполнения вызывающий станет владельцем нового, случайного кота. Функция возвращает `CAT id` полученного кота. Разработанный в ходе решения задания смарт-контракт можно задеплоить в ту же тестовую сеть и протестировать его работу с полученным из `PetShop` котом.  

Адрес `KittenRegistry` в сети *Ropsten*: `0xf59459AE845116c5b0d5401E656073d0BfDCec73`  
Адрес `PetShop` в сети *Ropsten*: `0x8BE44B86b1817D6a4e26EF1C78c5d3bb517ee441`  

Для разработки можно использовать IDE **remix** https://remix.ethereum.org.   
Для доступа к сети Ropsten - расширение **metamask** https://metamask.io/.   
Запускать собственную сеть не нужно.   
Для получения монет в тестовой сети можно использовать https://faucet.ropsten.be/ и https://faucet.metamask.io/  

Для вызова функций смарт-контрактов из тестовой сети, в *Remix*, на вкладке *run* можно выбрать нужный контракт из выпадающего списка и вместо **deploy** - выбрать пункт **at address**, предварительно указав адрес в соседнем поле. (**Note**: Новый интерфейс отличается визуально, но поля такие же)  

![pic_11_1](https://github.com/sotnikea/B_nTask11/raw/main/img/pic_11_1.png)   

# Обзор решения
### Тестирование с исходным KittenRegistry
Деплоим PatShop из файла KittenRegistry  

![pic1](https://github.com/sotnikea/B_nTask11/raw/main/pic1/1.png)     

Копируем адрес задеплоенного PatShop и используем его в качестве данных конструктора для деплоя KittenRegistry из файла KittenRegistry  
Таким образом база данных зарегистрированных животных привязывается к магазину  

![pic2](https://github.com/sotnikea/B_nTask11/raw/main/pic1/2.png)    

Деплоим написанный код контракта для животных  

![pic3](https://github.com/sotnikea/B_nTask11/raw/main/pic1/3.png)    

Используем функцию SetKittenRegistry с адресом KittenRegistry, чтобы после покупки, кот сразу регистрировался в базе данных

![pic4](https://github.com/sotnikea/B_nTask11/raw/main/pic1/4.png)    

Вводим нужную сумму для покупки и приобретаем по очереди 2 кота

![pic5](https://github.com/sotnikea/B_nTask11/raw/main/pic1/5.png)    

При попытке проверить баланс котов по адресу кошелька, с которого производилась оплата, видим, что котов в базе данных действительно два

![pic6](https://github.com/sotnikea/B_nTask11/raw/main/pic1/6.png)    

Регистрируем базу данных котов в созданном нами контракте

![pic7](https://github.com/sotnikea/B_nTask11/raw/main/pic1/7.png)    

В базе данных указываем адрес контракта и "нулевого" кота тем самым разрешая будущий перевод

![pic8](https://github.com/sotnikea/B_nTask11/raw/main/pic1/8.png)   

То же самое делаем со "первым" котом

![pic9](https://github.com/sotnikea/B_nTask11/raw/main/pic1/9.png)    

Отправляем на хранение "нулевого" кота с временем 0, чтобы его можно было сразу же забрать

![pic10](https://github.com/sotnikea/B_nTask11/raw/main/pic1/10.png)    

"Первого" кота отправляем с временем 1, чтобы его нельзя было забрать сразу

![pic11](https://github.com/sotnikea/B_nTask11/raw/main/pic1/11.png)    

Успешно забираем первого кота 

![pic12](https://github.com/sotnikea/B_nTask11/raw/main/pic1/12.png)   

И видим, что забрать второго кота пока нельзя, т.к. срок хранения не истек. Т.е. весь функционал работает верно

![pic13](https://github.com/sotnikea/B_nTask11/raw/main/pic1/13.png)    

### Тестирование контракта в сети Ropsten 

Для начала нужно загрузить написанный контракт в сеть Ropsten. Для этого меняем тип окружения на Inject Web3. В процессе разворачивания контракта автоматически происходит подключение к Metamask с запросом на одобрение операции.

![pic1](https://github.com/sotnikea/B_nTask11/raw/main/pic2/1.png)     

По окончанию получаем адрес контракта в сети  

![pic5](https://github.com/sotnikea/B_nTask11/raw/main/pic2/5.png)     

Деплоим по адресу в сети контракт магазина  

![pic2](https://github.com/sotnikea/B_nTask11/raw/main/pic2/2.png)     

Указываем нужную сумму для покупки и приобретаем по очереди 2 кота. В процессе приобретения автоматически происходит подключение к Metamask с запросом на одобрение операции.

![pic3](https://github.com/sotnikea/B_nTask11/raw/main/pic2/3.png)     

Деплоим по адресу в сети наш контракт

![pic4](https://github.com/sotnikea/B_nTask11/raw/main/pic2/4.png)     

Так же по адресу в сети деплоим KittenRegistry

![pic6](https://github.com/sotnikea/B_nTask11/raw/main/pic2/6.png)     

Проверяем баланc котов на счету, с которого они были приобретены и видим, что котов действительно два

![pic7](https://github.com/sotnikea/B_nTask11/raw/main/pic2/7.png)     

Открываем информацию по транзакциям с покупкой котов

![pic8](https://github.com/sotnikea/B_nTask11/raw/main/pic2/8.png)     

И запоминаем id купленного первого кота

![pic9](https://github.com/sotnikea/B_nTask11/raw/main/pic2/9.png)     

И второго кота

![pic10](https://github.com/sotnikea/B_nTask11/raw/main/pic2/10.png)     

В базе данных указываем адрес нашего контракта в сети и первого кота с индексом 24 тем самым разрешая будущий перевод..  И получаем бесконечную загрузку Метамаск. В самом метамаск в активности появляется запись "одобрить предел расходов metamask" при выборе которой так же бесконечная загрузка((

![pic11](https://github.com/sotnikea/B_nTask11/raw/main/pic2/11.png)     


![pic12](https://github.com/sotnikea/B_nTask11/raw/main/pic2/12.png) 

![pic13](https://github.com/sotnikea/B_nTask11/raw/main/pic2/13.png)     





# Ссылки
Цикл уроков по solidity - https://remix.ethereum.org/#optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.11+commit.d7f03943.js  
Размещение контракта в сети - https://vc.ru/crypto/28314-sozdaem-svoy-erc20-token-na-baze-ethereum-za-2-minuty

