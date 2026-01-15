# PixSTD.DataUtilities
**Официальная библиотека от PixSTD**

📦 NuGet: `PixSTD.DataUtilities`  
👨💻 Автор: PixSTD  
🐙 Исходный код: https://github.com/PixSTD/DataUtilities.dll  
📄 Лицензия: MIT

Библиотека для работы с данными.

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
		hash: ваш_ключ_шифрования,		// byte[] или string
		basePath: appFolder,			// куда сохранять файлы
		sizeNameDirectoryHex: 8,      	// 8 символов для папки
		sizeNameFileHex: 8            	// 8 символов для файла (Пример: "player/profile" → "a1b2c3d4/e5f67890")
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
	new PlayerData(hash, startPath, sizeNameDirectoryHex, sizeNameFileHex)
	
	или
	
	void Initialize(string hash)
	void Initialize(byte[] hash)
	
	void SetStartPath(string path);
	
	void SetSizeNameDirectoryHex(int size);
	void SetSizeNameFileHex(int size);
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
	bool LoadA<T>(out T value, string encryptedPath, string unencryptedPath = "")
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
	string GetRightPath(string path)
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

	public static string HashHex(string data, int lengthByte = 32, int offsetByte = 0)
	public static string HashHex(byte[] data, int lengthByte = 32, int offsetByte = 0)
	
	public static byte[] HashRaw(string data)
	public static byte[] HashRaw(byte[] data)
	
	byte[] Encrypt(byte[] plaintext)
	byte[] Decrypt(byte[] data)
```