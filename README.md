# PixSTD.DataUtilities
**Официальная библиотека от PixSTD**

📦 NuGet: `PixSTD.DataUtilities`  
👨💻 Автор: PixSTD  
🐙 Исходный код: https://github.com/PixSTD/DataUtilities.dll  
📄 Лицензия: MIT

**PixSTD.DataUtilities** — продвинутая библиотека для .NET, предоставляющая:

- **🔐 Безопасность** — шифрование и защита данных
- **🗂️ Хранение** — организация файлов с хешированием путей  
- **📡 Сетевая подготовка** — упаковка/распаковка данных для сетевой передачи
- **📦 Производительность** — быстрая сериализация через MessagePack
- **⚡ Совместимость** — .NET Standard 2.1, 8, 9, 10
- **🌐 Доступность** — Windows, Linux, Android
- **📝 Наблюдаемость** — полное логирование операций

## 🚀 Быстрый старт
```csharp
	using DataUtilities;
	using System;
	using System.IO;
	using System.Threading.Tasks;

	// 1. Инициализация
	
	string localAppData = Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData);
	string appFolder = Path.Combine(localAppData, "CompanyName", "YourAppName");
	
	var data = new PlayerData(
		hash: ваш_ключ_шифрования,				// byte[] или string
		startPath: appFolder,					// куда сохранять файлы
		lengthNameDirectory: HexLength.Short, 	// 8 символов для папки
		lengthNameFile: HexLength.Short,       // 8 символов для файла (Пример: "player/profile" → "a1b2c3d4/e5f67890")
		offsetHashHex: -1					// Смещение начала хеша. Значение -1 означает автоматический сдвиг, равный длине хешируемого сегмента
	);
	
	
	// 2. Сохранение данных (асинхронно)
	
	await data.SaveA(profile, "data/player/profile");
	// → сохранит в зашифрованном виде с хэшированными путями
	
	// 3. Загрузка данных (асинхронно)
	
	ProfileType profile;  // ваш тип класса профиля
	bool loaded = await data.LoadA<ProfileType>(out profile, "data/player/profile");
	
	if (loaded)
		Console.WriteLine("Профиль успешно загружен!");
	else
		Console.WriteLine("Профиль не найден или повреждён");
	
	
	// 4. Отправка/получение по сети
	
	byte[] networkData = data.EncryptNetworkData(request);  // → готово к отправке
	
	// На стороне получателя:
	MemoryStream receivedStream = new MemoryStream();  // сюда приходят пакеты
	
	// Когда получили кусок данных:
	byte[] chunk = ...;  // полученный кусок
	receivedStream.Write(chunk, 0, chunk.Length);
	
	// Обрабатываем всё, что накопилось
	var remains = data.DecryptNetworkData<Request>(
		receivedStream.ToArray(),
		req =>
		{
			// Здесь ваша логика обработки запроса
			Console.WriteLine($"Получен запрос: {req.Key}");
		});
	
	// remains — это неполный кусок, который ещё не образовал целое сообщение
	receivedStream.SetLength(0);  // очищаем поток
	
	if (remains.Length > 0)
	{
		receivedStream.Write(remains.ToArray(), 0, remains.Length);
		// теперь при следующем пакете остаток будет учтён
	}
	
	
	// 5. Логирование действий
	
	data.Log += (logEvent) =>
	{
		if (logEvent.Level <= DataLogLevel.Info)
			Console.WriteLine(logEvent.Message);
	};
```

## Инициализация
```csharp
	new PlayerData(hash, string startPath = "", HexLength lengthNameDirectory = HexLength.Long, HexLength lengthNameFile = HexLength.Full, int offsetHashHex = 0)
	
	или
	
	void SetHash(string hash)
	void SetHash(byte[] hash)
	
	void SetStartPath(string path);
	
	void SetLengthNameDirectory(int length);
	void SetLengthNameFile(int length);
	
	void SetOffsetHashHex(int offsetHex);
```

## Сохранение данных
```csharp
	void Save(object data, string encryptedPath, string unencryptedPath = "")
	Task SaveA(object data, string encryptedPath, string unencryptedPath = "")
```

## Загрузка данных
```csharp
	bool Load<T>(Action<T> apply, string encryptedPath, string unencryptedPath = "")
	Task<bool> LoadA<T>(Action<T> apply, string encryptedPath, string unencryptedPath = "")
	bool Load<T>(out T value, string encryptedPath, string unencryptedPath = "")
```

## копирование данных
```csharp
	bool CopyFile(string fromEncryptedPath, string toEncryptedPath, string fromUnencryptedPath = "", string toUnencryptedPath = "")
	(int copiedFiles, int totalFiles) CopyDirectory(string fromEncryptedPath, string toEncryptedPath, string fromUnencryptedPath = "", string toUnencryptedPath = "")
```

## Работа с файлами
```csharp
	int QuantityFiles(string encryptedPath, string unencryptedPath = "")
	string[] GetFiles(string encryptedPath, string unencryptedPath = "")
	bool SearchFile(string encryptedPath, string unencryptedPath = "")
	void DeleteFile(string encryptedPath, string unencryptedPath = "")
```

## Работа с директориями
```csharp
	int QuantityDirectories(string encryptedPath, string unencryptedPath = "")
	string[] GetDirectories(string encryptedPath, string unencryptedPath = "")
	bool SearchDirectory(string encryptedPath, string unencryptedPath = "")
	void DeleteDirectory(string encryptedPath, string unencryptedPath = "")
```

## Утилиты путей
```csharp
	string NormalizePath(string path)
	string GetNameFile(string path)
	string GetNameDirectory(string path)
	string GetPathToDirectory(string path)
	string GetCutPath(string path, int keepRight = 1)
```

## Конвертация
```csharp
	byte[] Convert(object data)
	T Convert<T>(byte[] data)
	object Convert(byte[] data, Type type)
```

## Шифрование
```csharp
	public byte[] EncryptNetworkData(object data)
	public ReadOnlyMemory<byte> DecryptNetworkData<T>(ReadOnlyMemory<byte> data, Action<T> action)

	public static string HashHex(string data, HexLength length = HexLength.Full, int offsetChars = 0)
	public static string HashHex(byte[] data, HexLength length = HexLength.Full, int offsetChars = 0)
	
	public static byte[] HashRaw(string data)
	public static byte[] HashRaw(byte[] data)
	
	byte[] Encrypt(byte[] plaintext)
	byte[] Decrypt(byte[] data)
```