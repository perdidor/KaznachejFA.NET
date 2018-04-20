# KaznachejFA.NET
Библиотека для работы с кассовым аппаратом "Казначей ФА" через последовательный порт RS232. Используйте MS Visual Studio 2017.

# Использование (тестировалось на Windows IoT Core @ Raspberry Pi 3, возможно пересобрать под любую другую платформу)

	string aqs = SerialDevice.GetDeviceSelector();
	var dis = await DeviceInformation.FindAllAsync(aqs);
	foreach (var item in dis)
	{
		//ищем определенный последовательный порт
		if (item.Name.Contains("FT232"))
		{
			CashDesk.CashDeskDeviceSerialPort = await SerialDevice.FromIdAsync(item.Id);
			CashDeskDeviceID = item.Id;
			break;
		}
	}
	//конфигурируем порт
	CashDesk.CashDeskDeviceSerialPort.BaudRate = 9600;
    CashDesk.CashDeskDeviceSerialPort.Parity = SerialParity.None;
    CashDesk.CashDeskDeviceSerialPort.StopBits = SerialStopBitCount.One;
    CashDesk.CashDeskDeviceSerialPort.DataBits = 8;
	//подключаемся к событиям кассы
	CashDesk.DeviceTypeReceived += CashDesk_DeviceTypeReceived;
    CashDesk.KKTPrinterStateReceived += CashDesk_KKTPrinterStateReceived;
    ...
	...
    CashDesk.AllTasksCancelled += CashDesk_AllTasksCancelled;
	//запускаем взаимодействие с кассовым аппаратом
	CashDesk.StartCommunication();
	...
	...
	//устанавливаем режим ККТ "Регистрация"
	CashDesk.ChangeMode(DeviceStateMode.Registration_WaitingForCommand);
	//открываем смену
	CashDesk.OpenStage();
	...
	...
	//периодически опрашиваем кассу (необходимо подключение ко всем событиям кассы)
	while (true)
	{
		if (CashDesk.CashDeskDeviceSerialPort != null)
                {
                    while (CashDesk.PendingTasks.Count != 0)
                    {
                        await Task.Delay(100);
                    }
                    CashDesk.GetKKTInformation();
                }
		Task.Delay(60000);
	}
	...
	//печать чека с одной позицией
	List<string> LinesToPrint = new List<string>
                    {
                    "Сдача округляется до ближайшего",
                    "целого рубля 0-49коп=0руб 50-99коп=1руб",
                    "телефон для справок +79031234567"
                    };
	//аргументы: доп. строки, кол-во товара, цена за единицу в копейках, принято наличными, название товара, используемая система налогообложения (см. документацию ККТ)
	CashDesk.PrintReceipt(LinesToPrint, 0.5, 500, 1000, "Вода питьевая", 2);
