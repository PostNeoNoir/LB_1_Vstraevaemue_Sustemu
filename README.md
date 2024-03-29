## P10 ##
Рассмотрим светодиодную матрицу размером 32x16 точек с 10 мм шагом между светодиодами, в данном проекте будут использоваться две такие панели. Яркость обеспечивается за счет козырька над светодиодами, не пропускающего паразитную засветку.
Панель представлена ниже:
![Panel](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/bfd30063-f84c-4eb9-bb22-55dd065d475d)


## Микросхемы логики ##
![Raspoloshenue_Logukuku](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/54f65423-1422-4015-8fe1-9aa60db76542)

1. 1 x SM74HC245D — неинвертирующий буфер
2. 1 x SM74HC04 — 6-канальный инвертор
3. 1 x SM74HC138D — 8-битный дешифратор
4. 4 x APM4953 — сборка из 2 P-канальных MOSFET
5. 16 x 74HC595D — сдвиговый регистр с защёлкой
Два 16-пиновых разъёма — интерфейсные, один из них входной (к нему подключается контроллер экрана), а второй — выходной (к нему подключается следующая матрица в цепочке). Стрелка на плате направлена от входного разъёма к выходному.
Питание подаётся на клеммы в центре платы. Напряжение питания — 5В, максимальный ток (когда включены все светодиодны матрицы) — 2А (для белой матрицы).
## Принципильная схема матрицы ##
![Prunchupualnya_Shema](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/4761b5df-d9ba-493e-ad46-27c5174922c0)

На плате для управления светодиодами использованы: буфер, дешифратор, инверторы, MOSFET транзисторы и 16 8-битных сдвиговых регистров для хранения состояния точек. Два 16-контактных разъема служат для последовательного подключения нескольких матриц. 

## Распиновка разъёма матрицы ##
![Raspunovka_Razuema](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/c26a30ad-511b-4d1a-9df2-a28439c7cb7a)

Сигналы управления следующие: nOE разрешает работу, А и В выбирают активную группу, CLK и R - тактовый сигнал и данные SPI, SCLK защелкивает данные на выходы регистров. Таким образом, обновление четверти экрана состоит из загрузки данных в SPI, включения нужной группы светодиодов и импульса на защелку для вывода точек из регистров.

Подключение в моём проекте:
![Podcluchenue_1](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/bf3516f4-08b1-4b8f-9bd9-3f5f52499671)
![Podcluchenue_2](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/88b58ca8-12ac-4262-83c5-fa1b808e5bed)


## Изучение SPI ##
**SPI (Serial Peripheral Interface)** - последовательный синхронный стандарт передачи данных в режиме полного дуплекса, 
предназначенный для обеспечения простого и недорогого высокоскоростного сопряжения микроконтроллеров и периферии

Для осуществленя подключения необходимо использовать четыре провода для передачи данных.
![SPI Port Names](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/5cb7e4d1-41f2-4526-95f7-fd737ec0153c)

Однако, в большинстве схем выделяют следующие наименование пинов:

- **MOSI** — выход ведущего, вход ведомого (англ. Master Out Slave In). Служит для передачи данных от ведущего устройства ведомому.

- **MISO** — вход ведущего, выход ведомого (англ. Master In Slave Out). Служит для передачи данных от ведомого устройства ведущему.

- **SCLK** или **SCK**— последовательный тактовый сигнал (англ. Serial Clock). Служит для передачи тактового сигнала для ведомых устройств.

- **CS** или **SS** — выбор микросхемы, выбор ведомого (англ. Chip Select, Slave Select).
Общий принцип передачи данных изображён на рисунке ниже.

![SPI communication system](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/3e25dbda-a117-42df-91ed-8cdcde7ce7ef)

Подлежащие передаче данные ведущее и ведомое устройства помещают в сдвиговые регистры. После этого ведущее 
устройство начинает генерировать импульсы синхронизации на линии *SCLK*, что приводит к взаимному обмену данными. 
Передача данных осуществляется бит за битом от ведущего по линии *MOSI* и от ведомого по линии *MISO*. Передача 
осуществляется, как правило, начиная со старших битов, но некоторые производители допускают изменение порядка 
передачи битов программными методами. После передачи каждого пакета данных ведущее устройство, в целях 
синхронизации ведомого устройства, может перевести линию *SS* в высокое состояние.

**Преимущества метода передачи данных**
- Средняя скорость передачи данных
- Возможность произвольного выбора длины пакета
- Простая аппаратная реализация
- Использованием лишь четырёх пинов
- Максимальная тактовая частота ограничена лишь производительностью устройств

**Недостатки метода передачи данных**
- Ведомое устройство не контролирует поток данных
- Нет официальных стандартов
- Отсутствие *"горячего"* подключения устройств
- Нет обратной связи от ведомого устройства

## Настройка отладочной платы Nukleo-144 ##
![Chastota](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/ccd4f9c1-2772-4bd5-93c5-b4579a55b9ad)
![GPIO](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/328c9fb9-3dcd-4099-ab59-215cbfbca0a9)
![SPI](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/ae17d10b-7e51-4208-9ba3-1fc0bd382d88)
![Pin](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/bec30dbd-6ca1-4e19-80cc-692dc0d6a2c3)
![Pin_2](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/891e5216-d231-446b-9011-1a5a719f7250)
![Pin_1](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/f46070de-08ae-4a49-b258-4b21b491098e)
![Pin_3](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/30ec670d-bb0f-4217-9f08-5b32dc52de47)

PD4 - A - отвечает за выбор строки для загрузки данных
PD5 - B - отвечает за выбор строки для загрузки данных
PD6 - OE - отвечает за включение / выключение панели
PD7 - SCLK - канал для применения полученных данных
PB10 - SPI2_SCK - канал для тактирования ведомого устройства от ведущего
PC3 - SPI2_MOSI -канал для передачи данных


## Программа для генерации изображений ##
На Python был разработан програмнный код, который может в зависимости от рисунка переводить его в матрицу состающую из 0 и 1 (горящий/негорящий светодиод). На вход данной программе необходимо предоставить картинку, размеры которой должны соответствовать: по длине 16 пикселей, по ширине кол-во матриц * 32. Данная программа считавает каждый пиксель изображения и преобразует в зависимости от его цвета в 0 или 1, далее это значение заносится в массив, с который далее будет выведен на экран. Этот массив необходимо скопировать в CubeIDE в файл display.c в любой из массивов. После проделанных опираций мы имеем полноценный кадр преобразованный из изображения.

![Prumer_Python_1](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/20a5f5e2-cdf8-4ccd-bcb6-2bb409cbbcda)
![Prumer_Python_2](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/cb284e23-00d2-431b-9346-ce5c089665fb)
![Prumer_Python_3](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/7990d3a3-0851-49ce-b5b8-14b8120cf659)


## Демонстрация работы ##
![Demonstratchue](https://github.com/PostNeoNoir/LB_1_Vstraevaemue_Sustemu/assets/128212528/c0e4ab35-24cb-44b4-935f-bea9c16ad305)


