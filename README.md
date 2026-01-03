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

	// 1. Создайте экземпляр
	// sizeNameDirectory/sizeNameFile - размер имени директории/файла при сохранении,
	// пример: "player/profile" => "(хеш размером sizeNameDirectory)/(хеш размером sizeNameFile)"

	var data = new PlayerData(hash, sizeNameDirectory, sizeNameFile);

	// 2. Сохранение
	await data.SaveA(profile, "data/player/profile");

	// 3. Загрузка
	await data.LoadA<T>(out T profile, "data/player/profile");

```

## Инициализация
```csharp
	new PlayerData(hash, sizeNameDirectory, sizeNameFile)
	
	или
	
	void Initialize(string hash) //hash - ключ с которым шифруются данные
	void Initialize(byte[] hash) //hash - ключ с которым шифруются данные
	
	void SetSizeNameDirectory(int size); //size - длина символов
	void SetSizeNameFile(int size); //size - длина символов
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
	byte[] Encrypt(byte[] plaintext)
	byte[] Decrypt(byte[] data)
```