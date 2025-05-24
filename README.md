# Технический отчёт проблем с распознаванием и передачей кодов маркировки


Технический отчёт по диагностике и анализу проблем с распознаванием и передачей кодов маркировки (DataMatrix) камерой DeltaVis100 и системой управления линий (СУЛ)(Контур.Маркировка). 

В отчёте подробно рассмотрены случаи некорректного считывания кодов, выявлены причины ошибок в логах и сетевом трафике, проведён анализ с использованием инструментов (OpenCV, Wireshark, Hercules)






# Неверное считывание кода камерой + СУЛ
В этом скриншоте СУЛ в журнале ошибок есть 1.5л код маркировки который записался как [0625, в камере так же он распознался как [0625

![image](https://github.com/user-attachments/assets/c09d8798-7787-4e6e-b87c-1fbff97e60f1)

С помощью собственного скрипта получили скриншот экрана где камера записала код неверно
![image](https://github.com/user-attachments/assets/24d75a48-eac2-4095-bc36-4474e8258956)
Камера распознала как [0625

Как только увидели что код записан неверно и в СУЛ он в ошибке. Используя собственный скрипт нашли данный скриншот, так скажем в логах(скриншоты) и сразу выловили данную бутылку с кодам с линии. Дальше заново прогнали данный код через камеру и код Прочитался камерой верно и в СУЛе не было ошибки

![image](https://github.com/user-attachments/assets/6f619dec-8768-46ef-9cff-915b98aea1fb)
Тот же самый код который был распознан как [0625 и был заново прогнан через камеру и распознан без ошибок

В логах КонтурМаркировка ошибка записана так: 
![image](https://github.com/user-attachments/assets/d57714b8-839f-4f54-9bd0-a5d3b3a1a70c)


# Неполный id кода в камере
Когда в веб-интерфейсе в DMC: код записан в виде gtin + несколько символов(средняя длина dmc), то код будет распознан верно и в СУЛ не будет ошибок, и будет введён в оборот корректно
![image](https://github.com/user-attachments/assets/8ee110ff-76de-41ef-a9ae-17647303cdbe)
![image](https://github.com/user-attachments/assets/f64cffba-dab9-4e7a-b1d7-897f985e6fd1)


# 0.5л camera Cognex. Стирание нуля в gtin кода в СУЛ + wireshark
В wireshark в пакете распознанного кода, gtin написан ноль
![image](https://github.com/user-attachments/assets/56e00a65-aa15-4668-9fe6-85d5dbb28383)

Это происходит когда создав задание в СУЛ и через камеру проходит первый код 

![image](https://github.com/user-attachments/assets/dc2fad28-a0da-4527-ae35-b9715057c7c6)

# Сканирование кода с помощью openCV + СУЛ + hercules
![image](https://github.com/user-attachments/assets/7c3fbf2d-aee9-40b1-9dc3-9d4551357669)
![image](https://github.com/user-attachments/assets/c5bbd63f-5150-458f-8d19-a87826b13533)
![image](https://github.com/user-attachments/assets/ba4f0be6-4834-49cc-978c-f6bfea69e007)


Всё так же отловили с линии данный код и проверили через openCV¸код распознан корректно. Проверили через сканер телефона (встроенный, показывает id code) и тоже корректно.
Дальше создали новое задание и просканировали код через камеру и в СУЛ код в ошибку не попал, то есть прочитан верно + использовали hercules
![image](https://github.com/user-attachments/assets/c2d03779-9a67-4389-835d-f5febc99b79c)
![image](https://github.com/user-attachments/assets/088c8934-e21b-4b91-b5a2-7561fd529aeb)

И вот ещё один код просканировали с помощью openCV
![image](https://github.com/user-attachments/assets/d8df4929-3065-4df9-b3f8-c796001c5e7a)

Сканирование кода используя сканер штрихкодов
![image](https://github.com/user-attachments/assets/7fad25b9-faba-40ad-960d-83a5d85ab3d6)
![image](https://github.com/user-attachments/assets/05f66277-db90-4349-b4e3-cb8c7eeb9f5b)
![image](https://github.com/user-attachments/assets/93b7d081-beef-4a6c-9801-e1b86ec8c5db)


# Просмотр пакетов камеры в wireshark
Нашли пакет который относится к коду который просканирован как [0625 
(пакет 150665)
![image](https://github.com/user-attachments/assets/67b09382-9e65-47ef-bc35-f2c40d46d09b)

IP камеры 192.168.0.51
В данном пакете в конце написан как поняли json формат где в ключе dmc и в значении этого ключа должен находиться id code, но там [0625 
![image](https://github.com/user-attachments/assets/5ce9aa8c-09be-4d78-b972-0eb0df8da80c)
![image](https://github.com/user-attachments/assets/8289fb73-1a52-4a8c-a53a-1cea160ce414)
![image](https://github.com/user-attachments/assets/588867df-68cf-401e-b136-094d88729025)

В предыдущем пакете ```150664``` как выяснили длина пакета всего 7
150662	5835.194414	192.168.0.174	213.180.193.234	TCP	55	[TCP Keep-Alive] 53720 → 443 [ACK] Seq=3632 Ack=8429 Win=131584 Len=1

150663	5835.222529	213.180.193.234	192.168.0.174	TCP	66	[TCP Keep-Alive ACK] 443 → 53720 [ACK] Seq=8429 Ack=3633 Win=42496 Len=0 SLE=3632 SRE=3633

```150664```	5835.317574	192.168.0.51	192.168.0.174	TCP	61	3000 → 54892 [PSH, ACK] Seq=6603 Ack=1 Win=14656 Len=7

```150665```	5835.362953	192.168.0.51	192.168.0.174	WebSocket	79	WebSocket Text [FIN]

150666	5835.367249	192.168.0.174	192.168.0.51	TCP	54	55706 → 80 [FIN, ACK] Seq=4119 Ack=106449 Win=1050112 Len=0

Посмотрев другие пакеты и там длина пакета намного больше

```151090```	5866.660221	192.168.0.51	192.168.0.174	TCP	94	3000 → 54892 [PSH, ACK] Seq=7330 Ack=1 Win=14656 Len=40
151091	5866.678933	192.168.0.159	224.0.0.252	LLMNR	65	Standard query 0x3150 A gaz-s

```151092```	5866.696299	192.168.0.51	192.168.0.174	WebSocket	94	WebSocket Text [FIN]

151093	5866.701314	192.168.0.174	192.168.0.51	HTTP	465	GET /tmp/temp2.jpg?rand=0.4909025400273612 HTTP/1.1

![image](https://github.com/user-attachments/assets/fce987bb-872b-48bd-a170-08f71b9fe884)
![image](https://github.com/user-attachments/assets/9567f39b-872f-4469-bf25-30f0b89a988d)
![image](https://github.com/user-attachments/assets/8ad0c367-2daf-4d19-ab1c-1902e9c6c863)

# Сетевая топология (устаревшая)

![image](https://github.com/user-attachments/assets/760e8fc7-fb8b-4cd6-a882-d9af8ed18bd9)

# Появление STP протоколов
В нашей сети как мы поняли ещё появляются петли

![image](https://github.com/user-attachments/assets/bb228b7d-425c-4b4f-88c1-c589460735e8)

![image](https://github.com/user-attachments/assets/96b23c73-e1e5-4e28-aacb-64fd55a31fc3)

![image](https://github.com/user-attachments/assets/aa725321-e391-4b2b-95d0-f517e0aff265)

![image](https://github.com/user-attachments/assets/87ee1c63-3b94-48df-a4e9-6b8398413f9d)

# Вывод
У нас такие мысли что камера считывает код верно (зелёная рамка) и в этот момент отправляется данный пакет данных в сеть и длина этих данных короткая, и возможно во время передачи данные потерялись/повредились/не все пришли. И в следующем пакете в Web-Socket в json формате в ключе dmc: записывается ключ, то есть id кода. 
И туда записывается то что осталось от короткого пакета. В дальнейшем СУЛ обрабатывает этот json формат и видя что код не полный (Не похожий на КМ) отправляет его в ошибку. 
Так же в веб-интерфейсе камеры в DMC: показывается результаты ключа dmc из json формата и наверное поэтому в камере в DMC: и в СУЛ выводится одни и те же результаты. Либо всё проще, камера больше неисправна











